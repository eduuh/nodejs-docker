## Docker for Node.js Projects From a docker Captain.

1. Node Apps in Cloud Native Docker
2. Compose for Awesome Local Development.
3. Making production ready images.
4. Running production Node.js Containers.

## Requirements

- Docker Desktop (prefered Win/mac)
- Docker Toolbox (Win 7/8/10 Home)
- Linux: Install via Docker Docs
  - docs.docker.com
- CLI: docker-compose (Separate binary written in python)
  - Included in Docker Desktop & Toolbox
  - Linux : **pip install docker-compose**

### Versions

> docker version
> 18.09.1
> docker-compose version
> 1.24

#### Why Compose?

- we talk about 2 parts. ClI and YAML files.
- Designed aroud developer workflows.
- Docker-compose CLI a substitute for **docker cli**

Docker compose for Development workflow is the best workflows instead of docker cli.

#### Compose File Format.

- Docker standard (not yet industy std)
- Defines multiple containers, networks , volumes, etc.
- Can layer sets of YAML files, use templates, variables, and more.
- docker-compose.yml default

Compose files are created with YAML markup language.

#### YAML (YAML Ain't Markup Language)

- Common configuration file format.
- Used by Docker, Kubernetes, Amazon and others.
- : used for Key/value pairs.
- Only spaces, no tabs
- - used for

#### Compose YAML v2 vs V3

- Myth busting : V3 does not replace v2
- v2 focus: single-node dev/test
- v3 focus: multi-node orchestration.
- If not using Swarm/kebernetes, stick to v2

#### What great about docker-compose CLI

1. Don't have to relearn new skills.
2. Reduce typing at command line.

- many docker commands == docker-compose.
- IDE's now supporst docker-compose.
- "batteries included, but swappables" - There are many defaults
- Docker compose cli and YAML versions differ.

#### Docker-compose up.

- "one stop shop"
- build/pull image(s) if missing
- Create volumes/networks/container(s)
- start container(s) in foreground (-d to detach)
- --build to always build.

#### docker-compose down

- "one stop shop"
- Stop and delete networks/container(s)
- use -v to delete volumes.

#### docker-compose ...

- Many commands take "service" option.
- **build** just build/rebuild image(s)
- **stop** Just stop containers don't delete.
- **ps** list "services"
- **push** images to resistry.
- **logs** same as docker cli.

#### Practise with docker.

1. Create an express project with **express --view=hbs sample-1**

docker-compose build --no-cache
docker-compose ps

if you are using docker on windows. You will have to use the ip address of the docker machine.

run the below command to get the ip address.

docker-machine ls

If you change the dockerfile and image exists in the machine. You will have to pass a --build command for docker compose to build the the new changes.

docker-compose up -d --build

##### docker-compose exec web sh

This gets you inside the **container shell**, and you could now start to run things inside the container.

#### DockerFiles Best Practise

- Node.js FROM images.
- CentOS custom Image.
- Lock Down Containers.
- Make images Efficient.

- Using _COPY_ instead of _ADD_. ADD do alot of stuff (downloading files from internet, untar files)
- npm/yarn install during build.(use the defaults with node containers.) . Make sure you are cleaning after words
- CMD node, not npm
  - requires another applicaiton to run.
  - not as literal in DockerFiles.(be supper efficient)
  - npm doesn't work well as an init or PID 1 process.
  - WORKDIR not Run mkdir
    - Unless you need chown

##### FROM Base Image Guidelnes.

- Stick to even numbered major releases.
- Don't use :latest tag
- Start with Debian if migrating
- Move to Alpine later
- Don't use : slim
- Don't use :onbuild

##### When to use Alpine Images.

- Alpine is "small" and "sec focused"
- But debian/Ubuntu are smaller now too.
- ~100MB space savings isn't significant.
- Alpine has its own issues.
- Alpine CVE scanning fails.
- Enterprises may require CentOS or Ubuntu/Debian.

##### Exercise.

- Install Node in the official CentOS
- Copy Dockerfile lines from node:10
- use Env to specify node version.
- This will take a few tries.
- Useful for knowing how to make your own node, but only if you have to.

This is the process of creating a custom image.

##### Least Privilege: Using node User

This is a common issue that will encounter with permissions problems.

- Official node images have a node user.
- But it's not used by default.
- Do this after **apt/apk** and **npm i -g**
- Do this before **npm i**
- May cause permissions issues with write access.
- May require **chown node:node**

- Change user from root to node.
  - USER node
- Set permissions on app dir.
  - RUN mkdir app && chown -R node:node .

When you run **docker-compose exec** you will usually enter the container as the node user. If you ever want to change that you can use.

docker-compose exec -u root

root user have access to anything.

It great to have your applications using the **node** user instead of **root** user which is the default.

- To enable the **node** user in you container we use the **user** command.
- Be cautious on the ordering problems of these line.

  - Command that need root user should be above the **USER** command.
  - Creating the working directory of the app of permissions **node:node**

    RUN app && chown -R node:node .

    USER node

- Also ensure you copied project is owned by **node user**.
  - ensure consistency in permissions.
  - COPY --chown=node:node . .

##### Making Images Efficently

We will focuss on build speeds and storage space.

1. using a small _base image_.
2. Line order matters. (due to cache-busting)
   - Line that dont change put them top.
     - Expose
3. Copying twice.
   - COPY package.json package-lock.json\* ./
   - copy only the package.json and lockfile
   - run npm install
   - Copy everything else
4. One _apt-get_ per dockerfile and be uptop.

example of a nice docker file

#### Node Process Management

1. Lifetime Event in containers.
2. Correcting Node Assumptions.
3. properly Replacing Node.

- no need for nodemon, forever, or pm2 on servers.
  - We'll use nodemon in dev for file watch during developments
- Docker manages app start, stop, resstart, healthcheck
- Node multi-thread: Docker manages multiple "replicas"
- One _npm/node_ problem: They don't listen for proper shutdown signals by default.

##### The truth About the PID 1 Problem.

- PID 1 (Process Identifier) if the first process in a system (or container) AKA init
- Init process in a container has two jobs

  - Reap zombie processes.
  - pass signals to sub-processes.

- Zombie not a big Node issue.
- Focus on proper Node shutdown.

##### Proper CMD for Healthy Shutdown.

- Docker uses Linux signal to stop app (SIGINT/SIGTERM/SIGKILL)
  - Avoid using SIGKILL (forcefull shutdown)
- SIGINT/SIGTERM allow gracefull stop.
  - For node we need to ensure, nodes cleans out any files it was reading to.
  - Gracefully shutdown with connections
- npm doesn't respond to SIGINT/SIGTERM
- node doesn't respond by default but can with code.
- Docker provides a init PID 1 replacement option. (Tini)

##### Proper Node shutdown options.

- Temp: use --init to fix ctrl-c for now
- Workaround: add tini to your image.
- Productions: your app captures SIGINT for proper exit.

  - docker run --init -d nodeapp

- Add tini to your Dockerfile, then use it in CMD (permanent workaround)

  - ENTRYPOINT ["/sbin/tini", "--"]
  - CMD [ "node", "./bin/www" ]

- Use JS snippet to properly capture signals (production solution)

  > ./sample-graceful-shutdown/sample.js

#### Assignment: Node Dockerfiles

- Make a Dockerfile for existing Node app.
- use ./assignment-dockerfile/Dockerfile.
- start with node 10.15 on alpine.
- Install tine, start node with tini.
- copy package/lock files first then npm, then copy.

Tini is the simplest **init** you could think of. All Tini does is spawn a single child (Tini is meant to be run in a container), and wait for it to exit all the while reaping **zombies and performing signal forwarding**.

#### Why Tini?

Using Tini has several benefits:

- It protects you from software that accidentally creates zombie processes. which can (over time!) starve you entire syste for PIDs
- It ensures that the defautl signal handles work for the software you run in your Docker image. For example, with Tini, **SIGTERM** properly terminates your process even if you didn't explicitly install a signal handler for it.
- It does so completely transparently! Docker images that work without Tini will work with Tin without any changes.

1. Manual install of tini.

Add Tini to your container, and make it executable. Then, just invoke Tini and pass your program and its arguments as arguments to Tini.

```dockerfile
FROM node:10.22-alpine

EXPOSE 3000

RUN apk add --no-cache tini

WORKDIR /usr/src/app

COPY  package.json package.lock*.json ./

RUN  npm install && npm cache clean --force

COPY . .

# Tini is now available at /sbin/tini

ENTRYPOINT ["/sbin/tini", "--"]

CMD ["node", "app.js"]
```

2. Using Tini. Injected at runtime

- If you are using **Docker 1.13 or greater**. Tini is included in Docker itself.
  - just **pass the --init flag to docker run**

##### > Docker keeps all version tags around since deleting them would cause stuff in production to fail.

##### Testing Graceful ehutdowns.

- Use ./dockerfiles/
- Run with tini built in, try to **ctrl-c**
- Run with tini built in, try to stop.
- Remove EntryPoint , rebuild.
- Add --init to run command, ctrl-c/stop
- Bonus: add signal watch code.

> #### Reminder. (practise point)

> using the assingment-dockerfile folder

- Building a container from a **dockerfile**
  - docker build . -t assignment.
- Running container Images with docker.
  - docker run -p 8080:3000 assingment.
- Running container in detached mode.
  - docker run -d -p 8080:3000 assingment.
  - The above command. produce a SHA1 text.
- Stoping the background container using **sha1 text**.
  - docker stop <sha1-text>
- check programs running using (linux utilities)
  - docker top <sha1-text>

#### Advanced Dockerfiles with Multi-stage and Buildkit.

- Multi-Stage builds.
- Docker BuildKits.
- Build A 3-stage Image.
- SSH Agent In Builds

##### Multi-stage Builds. (Artifact)

- New feature in 17.06 (mid-2017)
- Build Multiple Images from one file.
- Those images can **From** each other.
- COPY files between them.
- Space + security benefits.

example.

- To build dev image for dev stage.
  - docker build -t myapp .
- To build prod image from prod stage.

  - docker build -t myapp:prod --target prod .

- Add a test stage that runs npm test.
- Have CI build --target test stage before building prod.
- Add npm install
  --only=development to dev stage

##### Building A 3-Stage Dockerfiles

- Create a Docker from ./Multi-stage-dockerfile
- Create three stages for prod, dev and test.
- Prod has no devDependencies and runs node.
- Dev includes devDep, runs nodemon.
- Test has DevDep, runs npm test
- Goals: don't repeat lines.

```Dockerfile
FROM node:10-slim as prod
ENV NODE_ENV=production
EXPOSE 3000
WORKDIR /app
COPY package*.json ./
RUN npm install --only=production && npm cach clean --force
COPY . .
CMD [ "node" ,"./bin/www"]

FROM prod as dev
ENV NODE_ENV=development
RUN npm install --only=development
CMD ["node_modules/nodemon/bin/nodemon.js", "./bin/www --inspect=0.0.0.0:9229" ]

FROM dev as test
ENV NODE_ENV=development
RUN npm test

```

- The above _dockerfile_ has three stages **prod, dev and test**
- To build the **prod** image.

  - **docker build -t multistage --target prod . && docker run multistage**

- To run the development image. (using tini to for grafull shutdown)

  - **docker build -t multistage:dev --target dev . && docker run multistage --init -p 3000:3000 multistage:dev**

- To run the test container layer.

  - **docker build -t multistage:test --target test . && docker run --init -p 3000:3000 multistage:test**

#### BuildKit

- Buildkit. It's a new way to build your images, and a replacement "build engine"
- It is an optinal feature with quite a few benefits over traditional docker build commands.

- Buildkit doesn't work with **docker-compose** so it can't be used for local development.

- Break up your local dev workflow into manual steps such as:

1. Use docker build commands for node.js image.
2. Then use docker-compose for the rest.

#### Benefits.

- Most images builds will be faster.
- Some re-builds will be much faster.
- It ignores stages in multi-stage that aren't needed. This saves considerable time once you have many stages for different uses.
- Mount host paths and **secrets during builds** so they are never stored in images.
- Mount host **ssh-agents so builds can use your private keys** for private NPM modulse witho copying to images.
- Mount **package manager caches** so they can reuse package downloads between builds (apt, apk, yum, npm, yarn, etc)
- Future: add features to buildKit "frontends" without needing a new version of Docker. We can control BuildKit version in DockerFile(optional.)

#### Limitations.

- No windows Container support yet (only works in Linux container)
- No **docker-compose** support yet.
- No UCP (Docker Enterprise) support yet
- Various registry limitation including using private or insecure registries (fixes in progress.)
  -Bugs are still being discoverde and worked on at **moby/buidkit**
- Not enabled by default.
- Some features require enabling experimental mode in Docker Engine.
- some features require Dockerfile command that are not backwords-compatible.

##### How to Enable it

You can set an environment variable **DOCKER_BUILDKIT=1** to enable it for your current shell, and update the docker engine config to enable it permanently when you're ready to got in on Buildkit.

- Enable in Bash/zsh and Toolbox's Quickstart Terminal with: ** export DOCKER_BUIDKIT=1**
- Enable in PowerShell with : \*_\$env:DOCKER_BUIDKIT=1_
- Optnally , enable **permanently** in Docker Desktop by updating prefereces/settings **Daemon advanced" Json of {"features: {"buildkit": true}}**
- Enable **permanently** in Linux host bu updating the **/etc/docker/daemon.json** file with **{"features: {"buildkit": true}}**

##### Using BuildKit to Enable SSD Keys for Private Npm Repositories.

If you Node project has private git repos for node modules, it'll need a particular setup so **ssh** can be used when building the images.

> The previous solution before Buildkit was:

1. use multi-stage builds.
2. COPY a decrypetd-private-key in to an early stage where npm install is run.
3. COPY the node_modules from that stage to a new image that doesn't include the key.

That solution worked if your're ok with having the ssh key stored in your local docker engine images, but if wasn't ideal, and didn't work with encrypted ssh kes that required a passphrase.

> The new way is to use BuildKit with the ssh-agent feature, and is much more secure.

1. Setup **ssh-agent** and your keys on the host OS like normal.
2. Add this as the first line in our Dockerfile: # **syntax = docker/dockerfiles:experimental**
3. Start our **Dockerfile** npm install line with this : **RUN --mount=type=ssh**
4. Run docker build with **--ssh default** as an additional option to enable the feature for that build.

You will need to build images manually due to the fact that this in not yet support by **docker-compose.yml**

- check ./sample-buildkit-ssh

#### Cloud Native App Guidelines.

- Follow 12factor.net priciples, especially
  - use Environmental Variable for config.
  - Log to stdout/stderr.
  - Pin all version, even npm.
  - Groceful exit SIGTERM/INIT
- Create a _.dockerignore_
- Containers are almost always distributed apps.
- **Good news:** You get many of these by using Docker.
- Lets focus on a few for Node.js

#### 12 Factor: Config

- **Heroku** wrote a highly respected guide to creating distributed apps.

  - Twelve factors to consider when developing or designing distributed apps.

- 12factor.net/config

  - Store environment config in Environment Variable (env vars)
  - Docker & Compose are great at this with multiple options.
  - Old apps: Use CMD or ENTRYPOINT script with **envsubst** to pass env vars into conf files.

#### 12 Factor: Logs

- Apps shouldn't route or transport logs to anything but stdout/stderr.
- **console.log()** works.
- Winston/Bunyan/Morgan: use levels to control verbosity.
- Winston transport: "console"

### .dockerignore.

- Prevent bload and unneeded files.

  - .git/
  - node_modules/
  - npm-debug
  - docker-compose\*.yml

- Not needed but useful in Image
  - Dockerfiles
  - README.md

#### Migrating Traditional Apps (MTA)

- "Traditional App" = Pre-Dcoker App.
- Take a typical Nod app and "migrates"

- using ./mta
- Add .dockerignore
- Create Dockerfile
- Change Winston transport to Console.

##### MTA Requirements.

- See README.md for app details.
- Image shouldn't include **in , out, node_modules or logs** directories
- Change Winston to Console
  - **winston.transports.console**
- bind-mout in and out dirs
- Set CHARCOAL_FACTOR to 0.1

##### MTA OUTcomes

- Running contaner with **./in** and **./out** bind-mounts results in new chalk images in \*/.out\*\* on host.
- Changing **--env** CHARCOAL_FACTOR changes
- Docker logs shows winston outputs.
