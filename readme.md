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
  - Linux : --pip install docker-compose--

### Versions

> docker version
> 18.09.1
> docker-compose version
> 1.24

#### Why Compose?

- we talk about 2 parts. ClI and YAML files.
- Designed aroud developer workflows.
- Docker-compose CLI a substitute for --docker cli--

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
- --build-- just build/rebuild image(s)
- --stop-- Just stop containers don't delete.
- --ps-- list "services"
- --push-- images to resistry.
- --logs-- same as docker cli.

#### Practise with docker.

1. Create an express project with --express --view=hbs sample-1--

docker-compose build --no-cache
docker-compose ps

if you are using docker on windows. You will have to use the ip address of the docker machine.

run the below command to get the ip address.

docker-machine ls

If you change the dockerfile and image exists in the machine. You will have to pass a --build command for docker compose to build the the new changes.

docker-compose up -d --build

##### docker-compose exec web sh

This gets you inside the --container shell--, and you could now start to run things inside the container.

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
- Do this after --apt/apk-- and --npm i -g--
- Do this before --npm i--
- May cause permissions issues with write access.
- May require --chown node:node--

- Change user from root to node.
  - USER node
- Set permissions on app dir.
  - RUN mkdir app && chown -R node:node .

When you run --docker-compose exec-- you will usually enter the container as the node user. If you ever want to change that you can use.

docker-compose exec -u root

root user have access to anything.

It great to have your applications using the --node-- user instead of --root-- user which is the default.

- To enable the --node-- user in you container we use the --user-- command.
- Be cautious on the ordering problems of these line.

  - Command that need root user should be above the --USER-- command.
  - Creating the working directory of the app of permissions --node:node--

    RUN app && chown -R node:node .

    USER node

- Also ensure you copied project is owned by --node user--.
  - ensure consistency in permissions.
  - COPY --chown=node:node . .

##### Making Images Efficently

We will focuss on build speeds and storage space.

1. using a small _base image_.
2. Line order matters. (due to cache-busting)
   - Line that dont change put them top.
     - Expose
3. Copying twice.
   - COPY package.json package-lock.json\- ./
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

Tini is the simplest --init-- you could think of. All Tini does is spawn a single child (Tini is meant to be run in a container), and wait for it to exit all the while reaping --zombies and performing signal forwarding--.

#### Why Tini?

Using Tini has several benefits:

- It protects you from software that accidentally creates zombie processes. which can (over time!) starve you entire syste for PIDs
- It ensures that the defautl signal handles work for the software you run in your Docker image. For example, with Tini, --SIGTERM-- properly terminates your process even if you didn't explicitly install a signal handler for it.
- It does so completely transparently! Docker images that work without Tini will work with Tin without any changes.

1. Manual install of tini.

Add Tini to your container, and make it executable. Then, just invoke Tini and pass your program and its arguments as arguments to Tini.

```dockerfile
FROM node:10.22-alpine

EXPOSE 3000

RUN apk add --no-cache tini

WORKDIR /usr/src/app

COPY  package.json package.lock-.json ./

RUN  npm install && npm cache clean --force

COPY . .

# Tini is now available at /sbin/tini

ENTRYPOINT ["/sbin/tini", "--"]

CMD ["node", "app.js"]
```

2. Using Tini. Injected at runtime

- If you are using --Docker 1.13 or greater--. Tini is included in Docker itself.
  - just --pass the --init flag to docker run--

##### > Docker keeps all version tags around since deleting them would cause stuff in production to fail.

##### Testing Graceful ehutdowns.

- Use ./dockerfiles/
- Run with tini built in, try to --ctrl-c--
- Run with tini built in, try to stop.
- Remove EntryPoint , rebuild.
- Add --init to run command, ctrl-c/stop
- Bonus: add signal watch code.

> #### Reminder. (practise point)

> using the assingment-dockerfile folder

- Building a container from a --dockerfile--
  - docker build . -t assignment.
- Running container Images with docker.
  - docker run -p 8080:3000 assingment.
- Running container in detached mode.
  - docker run -d -p 8080:3000 assingment.
  - The above command. produce a SHA1 text.
- Stoping the background container using --sha1 text--.
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
- Those images can --From-- each other.
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
COPY package-.json ./
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

- The above _dockerfile_ has three stages --prod, dev and test--
- To build the --prod-- image.

  - --docker build -t multistage --target prod . && docker run multistage--

- To run the development image. (using tini to for grafull shutdown)

  - --docker build -t multistage:dev --target dev . && docker run multistage --init -p 3000:3000 multistage:dev--

- To run the test container layer.

  - --docker build -t multistage:test --target test . && docker run --init -p 3000:3000 multistage:test--

#### BuildKit

- Buildkit. It's a new way to build your images, and a replacement "build engine"
- It is an optinal feature with quite a few benefits over traditional docker build commands.

- Buildkit doesn't work with --docker-compose-- so it can't be used for local development.

- Break up your local dev workflow into manual steps such as:

1. Use docker build commands for node.js image.
2. Then use docker-compose for the rest.

#### Benefits.

- Most images builds will be faster.
- Some re-builds will be much faster.
- It ignores stages in multi-stage that aren't needed. This saves considerable time once you have many stages for different uses.
- Mount host paths and --secrets during builds-- so they are never stored in images.
- Mount host --ssh-agents so builds can use your private keys-- for private NPM modulse witho copying to images.
- Mount --package manager caches-- so they can reuse package downloads between builds (apt, apk, yum, npm, yarn, etc)
- Future: add features to buildKit "frontends" without needing a new version of Docker. We can control BuildKit version in DockerFile(optional.)

#### Limitations.

- No windows Container support yet (only works in Linux container)
- No --docker-compose-- support yet.
- No UCP (Docker Enterprise) support yet
- Various registry limitation including using private or insecure registries (fixes in progress.)
  -Bugs are still being discoverde and worked on at --moby/buidkit--
- Not enabled by default.
- Some features require enabling experimental mode in Docker Engine.
- some features require Dockerfile command that are not backwords-compatible.

##### How to Enable it

You can set an environment variable --DOCKER_BUILDKIT=1-- to enable it for your current shell, and update the docker engine config to enable it permanently when you're ready to got in on Buildkit.

- Enable in Bash/zsh and Toolbox's Quickstart Terminal with: -- export DOCKER_BUIDKIT=1--
- Enable in PowerShell with : \-_\$env:DOCKER_BUIDKIT=1_
- Optnally , enable --permanently-- in Docker Desktop by updating prefereces/settings --Daemon advanced" Json of {"features: {"buildkit": true}}--
- Enable --permanently-- in Linux host bu updating the --/etc/docker/daemon.json-- file with --{"features: {"buildkit": true}}--

##### Using BuildKit to Enable SSD Keys for Private Npm Repositories.

If you Node project has private git repos for node modules, it'll need a particular setup so --ssh-- can be used when building the images.

> The previous solution before Buildkit was:

1. use multi-stage builds.
2. COPY a decrypetd-private-key in to an early stage where npm install is run.
3. COPY the node_modules from that stage to a new image that doesn't include the key.

That solution worked if your're ok with having the ssh key stored in your local docker engine images, but if wasn't ideal, and didn't work with encrypted ssh kes that required a passphrase.

> The new way is to use BuildKit with the ssh-agent feature, and is much more secure.

1. Setup --ssh-agent-- and your keys on the host OS like normal.
2. Add this as the first line in our Dockerfile: # --syntax = docker/dockerfiles:experimental--
3. Start our --Dockerfile-- npm install line with this : --RUN --mount=type=ssh--
4. Run docker build with ----ssh default-- as an additional option to enable the feature for that build.

You will need to build images manually due to the fact that this in not yet support by --docker-compose.yml--

- check ./sample-buildkit-ssh

if you ever change a Dockerfile line before the --Run npm install line-- or you change your --package.json or lock file, Docker will need to re-run-- npm install build. Docker , by default, won't re-use package manager download caches line the NPM cache.

If you have large package.json file with slow dependency install due to large downloads you can speed up rebuilds by enabling buildkit caching feature on specific directories inside your docker builds.

To set this up for re-using the NPM download cache.

1. add this as the first line in our Dockerfile: --# syntax = docker/dockerfiles:experimental--
2. Start your Dockerfile npm instal line with: --RUN --mount=type=cache target/root/.npm/\_cacache--

Look at

#### Cloud Native App Guidelines.

- Follow 12factor.net priciples, especially
  - use Environmental Variable for config.
  - Log to stdout/stderr.
  - Pin all version, even npm.
  - Groceful exit SIGTERM/INIT
- Create a _.dockerignore_
- Containers are almost always distributed apps.
- --Good news:-- You get many of these by using Docker.
- Lets focus on a few for Node.js

#### 12 Factor: Config

- --Heroku-- wrote a highly respected guide to creating distributed apps.

  - Twelve factors to consider when developing or designing distributed apps.

- 12factor.net/config

  - Store environment config in Environment Variable (env vars)
  - Docker & Compose are great at this with multiple options.
  - Old apps: Use CMD or ENTRYPOINT script with --envsubst-- to pass env vars into conf files.

#### 12 Factor: Logs

- Apps shouldn't route or transport logs to anything but stdout/stderr.
- --console.log()-- works.
- Winston/Bunyan/Morgan: use levels to control verbosity.
- Winston transport: "console"

### .dockerignore.

- Prevent bload and unneeded files.

  - .git/
  - node_modules/
  - npm-debug
  - docker-compose\-.yml

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
- Image shouldn't include --in , out, node_modules or logs-- directories
- Change Winston to Console
  - --winston.transports.console--
- bind-mout in and out dirs
- Set CHARCOAL_FACTOR to 0.1

##### MTA OUTcomes

- Running contaner with --./in-- and --./out-- bind-mounts results in new chalk images in \-/.out\-\- on host.
- Changing ----env-- CHARCOAL_FACTOR changes
- Docker logs shows winston outputs.

```dockerfile
FROM node:8

RUN apt-get install -y graphicsmagick`

WORKDIR app/

COPY package-.json ./

RUN npm install && npm cache clean --force

COPY . .
# setting environment variables
ENV CHARCOALFACTOR=0.1

CMD ["node", "index.js"]
```

- Building an images from the docker file
  - --docker build . -t mta--
- Running this project using --bindmounts--
  - --docker run -v ${pwd}/in:/app/in -v $(pwd)/out:/app/out --env CHARCOAL_FACTOR=1 mta--
- Check inside a running images.
  - --docker exec --it mta--
- Check logs of the runnig container.
  - --docker logs all--

#### Compose for Local Development.

- Best local setup
- Use Compose features.
- Tips and Tricks.

##### Compose Project Tips: Do's

- cd ./compose-tips.
- Do use --docker-compose-- for local dev
- do use --version 2-- . for local dev.
- V2 only: depents_on, hardware specific.

#### Compose Project Tips: Don'ts (folder: compose-tips)

Read the yaml in the said folder.

> This are te dont's

- Unnecessary: "alias" & "container_name"
- Legacy: "expose" & "links"
  - All container in the same networks are exposed to each others.
  - Don't set up defaults settings, i.e using network bridge.
- BAD: if bind-mounting folders or files to host, always use relative file
  - paths (starting with .). This makes your compose file reusable for others, and won't break if you move your project around.
    - Note: everybody will have unique path.
- BAD: don't bind-mount databases to host OS. You'll get bad performance and many times it won't even work. Best to use named volumes.
- For local dev only? don't copy in code.
- DDforWin needs drive perms
  - Perms: Linux != Windows

> Bind-Mounting = Perfomance
> docker-bg-sync

##### node_modules in images. (folder: sails)

- Problem: we shouldn't build images with --node_modules-- from host.
  - Example: --node-gyp--
- Solution: add --node_modules-- to --.dockerignore--
- let's do this to --sails--
- Before you do a --docker build-- make sure you have a .dockerignore file.
  - docker build -t sailsbret .

##### node_modules in Bind-Mounts (folder: express)

- Problem: we can't bind-mount node_modules content from host on macOs/Windows (diffrent arch)
- Two potential Solutions:
  1. Never use --npm i on host--
  - run npm i in compose.
  2. Move modules in image, hide modules from host.

###### Solution 1, simple but less flexible:

This solution assumes that you will be developing entirely in the container. This is what makes it less flexible.

```yml
version: "2.4"

services:
  express:
    build: .
    ports:
      - 3000:3000
    volumes:
      - .:/app
    environment:
      - DEBUG=sample-express:-
```

- You can't --docker-compose up-- until you've used --docker-compse run--
- To install \-_node-modules_ using docker compose.
  - docker-compose run express npm install

--node_modules from the container-- will be saved in the host.

###### Solution 2.

- Solution 2, more setup but flexible.
  - I could develop, in either the host, or in the container. (What pleases at the time)
- Move node_modules up a directory in Dockerfile.
  - In the container we will use --node_modules-- in the parent directory.
- use empty volume to hide --node_modules-- on --bind-mount--.

  - This hide the --node_modules-- from the host --npm install--

- You can develop on both --windows-- and --linux-- without any side effects of node_modules.

```dockerfile

FROM node:10.15-slim

ENV NODE_ENV=production

WORKDIR /node

COPY package.json package-lock-.json ./

RUN npm install && npm cache clean --force

WORKDIR /node/app

COPY . .

CMD ["node", "./bin/www"]
```

```yml
version: "2.4"

services:
  express:
    build: .
    ports:
      - 3000:3000
    volumes:
      - .:/node/app
      - /node/app/node_modules
    environments:
      - DEBUG=sample-express:-
```

##### NPM, Yarn , and other Tools in Compose.

- Two ways to run various tools inside the container.
- docker-compose run: start a new container and run command/shell
- docker-compose exec: run additional command/shell when the container is running

##### File Monitoring and Node Auto Restart.

- Use nodemon for compose file monitoring.
- webpack-dev-server, etc. work the same.
- Override Dockerfile via compose command.
- If windows, enable polling.
- create a --nodemon.yml-- for advanced workflows (bower, webpack, parcel)

```dockerfile
FROM node:10.15-slim

ENV NODE_ENV=production

WORKDIR /app

COPY package.json package-lock-.json ./

RUN npm install && npm cache clean --force

ENV PATH /app/node_modules/.bin/:$PATH

COPY . .

CMD ["node", "./bin/www"]
```

```yml
version: "2.4"

services:
  express:
    build: .
    command: /app/node_modules/.bin/nodemon ./bin/www
    ports:
      - 3000:3000
    volumes:
      - .:/app
    environment:
      - DEBUG=sample-express:-
      - NODE_ENV=development
```

> Notes:

- --NODE_ENV=developments-- overrides the dockerfiles environments variable which is set to --production--. This ensures that every thing i need in a dev env in setup correctly.

- docker-compose file is also used to override the default --command-- to use nodemon.
  - Also note that the full path of nodemon from the --node_modules-- is used.
    - This is because nodemon is not installed globally.We are using package.json so that we be able to control the version of nodemon.
- Since at the --bind-mount-- we bind the whole app directory. That means the --node_modules-- are shared between the host and the container.
  - don't npm install on the host. make use of docker compose.
    - --docker-compose run express npm install nodemon --save-dev--
      - express is the --service name-- from the --docker-compose.yml-- file.

##### Startup Order and Dependencies.

- problem: Multi-service apps start out of order, node might exit or cylec.
- Multi-container apps needs:
  - Dependency awareness
  - Name resolution (DN)
  - Connection failure handling.

###### Dependecy Awareness.

- --depends_on--: when "up X", start Y first
- Fixes name resolution issues with "can't resolve <service_name>
- Only for compose, not Orchestration.
- Compose YAML v2: works with healthchecks like a "wait for script".

###### Connection Failure Handling

- _restart_: on-failure
  - Good: help slow db startup and Node.js failing. Better: depends_on.
  - Bad: could spike Cpu with restart cycling.
- --Solution-- build connection timeouts, buffer and retries in your apps.

###### HealthChecks for depends_on.(folder: depends-on)

- --depends_on--: is only dependency control by default.
- Add v2 healthchecks for true "--wait_for--"
- Let's see some examples

  - Mongo
  - Postgress/MySql
  - web

```yml
version: "2.4"

services:
  frontend:
    image: nginx
    depends_on:
      api:
        # this requires a compose file version => 2.3 and < 3.0
        condition: service_healthy

  api:
    image: node:alpine
    healthcheck:
      test: curl -f http://127.0.0.1
    depends_on:
      postgres:
        condition: service_healthy
      mongo:
        condition: service_healthy
      mysql:
        condition: service_healthy

  postgres:
    image: postgres
    environment:
      POSTGRES_HOST_AUTH_METHOD: trust
    healthcheck:
      test: pg_isready -U postgres -h 127.0.0.1

  mongo:
    image: mongo
    healthcheck:
      test: echo 'db.runCommand("ping").ok' | mongo localhost:27017/test --quiet

  mysql:
    image: mysql
    healthcheck:
      test: mysqladmin ping -h 127.0.0.1
    environment:
      - MYSQL_ALLOW_EMPTY_PASSWORD=true
```

The above example uses preexisting images to compose this --docker-compose-- file.

- Each container depends on another container.
- To esure all the container are spined up in the correct order. There is an --depends_on--.
- To esure all the container play nicely together, all of them has a --healhcheck--

##### Making microservices Easier.

- Problem: many HTTP endpoints, man ports.
- Solution: Nginx/HAProxy/Traefik for host header routing + wildcard localhost domain.
- Problem: CORS failures in dev.
- Solution: Proxy with \- header
- Problem: HTTPS locally
- Solution: Create local proxy Certificates.

##### Local DNS For Many EndPoints.

- Problem: Multiple endpoints an need unique Dns for each. in order of preference.
  - Use x.localhost, y.localhost in chrome.
  - Use wilcard domains like
    \-.vcap.me or xip.io
  - Use --dnsmasq-- on macOS/linux
  - Manually edit host file.

#### Vc code, Debugging, and TypeScript. (folder: typescript)

- Vs code and other editors have some --Docker and compose features built-in--
- Debugging works when we enable in nodemon and remote via TCP.
- TypeScript compile and other --pre-processors-- go in --nodemon.json--.

```dockerfile
# a base stage for all future stages
# with only prod dependencies and
# no code yet
FROM node:10-slim as base
ENV NODE_ENV=production
WORKDIR /app
COPY package-.json ./
RUN npm install --only=production \
    && npm cache clean --force
ENV PATH /app/node_modules/.bin:$PATH

# a dev and build-only stage. we don't need to
# copy in code since we bind-mount it
FROM base as dev
ENV NODE_ENV=development
RUN npm install --only=development
# run nodemon. It will use the nodemon.json file for configurations.
CMD ["/app/node_modules/.bin/nodemon"]

FROM dev as build
COPY . .
RUN tsc
# you would also run your tests here

# this only has minimal deps and files
FROM base as prod
COPY --from=build /app/dist/ .
CMD ["node", "app.js"]
```

```dockerfile
node_modules/
```

```yml
version: "2.4"

services:
  ts:
    build:
      context: .
      target: dev
    ports:
      - "3000:3000"
      - "9229:9229"
    volumes:
      - .:/app
```

Note: we expose port --9229-- for node-debugging.
Since we are using nodemon. We expose this port using _nodemon.json_

```json
{
  "watch": ["src"],
  "ext": "ts",
  "ignore": ["src/--/-.spec.ts"],
  "exec": "node --inspect=0.0.0.0:9229 -r ts-node/register ./src/app.ts"
}
```

Since we are configurin typescript. I still have a --tsconfig.json--

```json
{
  "include": [
    "src/--/-     /-Location of my typscript files-/
  ],

  "compilerOptions": {
    "target": "es5",
    "module": "commonjs",
    "sourceMap": true,
    "outDir": "dist",
    "strict": true,
    "noImplicitAny": false,
    "esModuleInterop": true                    }
  }
}
```

#### Building A sweet Compose File. (folder: sweet-compose)

- Take all the learning from this section and apply it to a single compose file!
- Use Docker's example voting app (Dog vs. Cat)

You are the Node.js developer for the "Dog vs. Cat voting app" project.
You are given a basic docker-compose.yml and the source code for the "result"
Node.js app (sub directory of this dir).

Goal: take the docker-compose.yml in this directory, which uses the docker
voting example distributed app, and make it more awesome for local development
of the "result" app using all the things you learned in this section.

## The finished compose.yml should include:

- Set the compose file version to the latest 2.x (done for you)
- Healthcheck for postgres, taken from the depends_on lecture
- Healthcheck for redis, test command is "redis-cli ping"
- vote service depends on redis service
- result service depends on db service
- worker depends on db and redis services
- remember to add the service_healthy to depends on objects
- result is a node app in subdirectory result. Let's bind-mount that
- result should be built from the Dockerfile in ./result/
- Add a traefik proxy service from proxy lecture example. Have it run
  on a published port of your choosing and direct vote.localhost and
  result.localhost to their respective services so you can use Chrome
- Add nodemon to the result service based on file watching lecture. You
  may need to get nodemon into the result image somehow.
- Enable NODE_ENV=development mode for result
- Enable debug and publish debug port for result

## Things to test once finished to ensure it's working:

- Edit ./result/server.js, save it, and ensure it restarts
- Ensure you never see "Waiting for db" in docker-compose logs, which happens
  when vote or result are waiting on db or redis to start
- Use VS Code or another editor with debugger (or Chrome) to connect to debugger
- Goto vote.localhost and result.localhost and ensure you can vote and see result

```yml
version: "2.4"

services:
  traefik:
    image: traefik:1.7-alpine
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    ports:
      - "8080:80"
    command:
      - --docker
      - --docker.domain=traefik
      - --docker.watch
      - --api
      - --defaultentrypoints=http,https
    labels:
      - traefik.port=8080
      - traefik.frontend.rule=Host:traefik.localhost
    networks:
      - frontend
      - backend

  redis:
    image: redis:alpine
    networks:
      - frontend
    healthcheck:
      test: redis-cli ping

  db:
    image: postgres:9.6
    environment:
      - POSTGRES_HOST_AUTH_METHOD=trust
    volumes:
      - db-data:/var/lib/postgresql/data
    networks:
      - backend
    healthcheck:
      test: pg_isready -h 127.0.0.1

  vote:
    image: bretfisher/examplevotingapp_vote
    networks:
      - frontend
    depends_on:
      redis:
        condition: service_healthy
    labels:
      - traefik.port=80
      - traefik.frontend.rule=Host:vote.localhost

  result:
    build:
      context: ../result
    command: nodemon index.js
    volumes:
      - ../result:/app
    networks:
      - backend
    depends_on:
      db:
        condition: service_healthy
    labels:
      - traefik.port=80
      - traefik.frontend.rule=Host:result.localhost

  worker:
    image: bretfisher/examplevotingapp_worker:java
    networks:
      - frontend
      - backend
    depends_on:
      db:
        condition: service_healthy
      redis:
        condition: service_healthy

networks:
  frontend:
  backend:

volumes:
  db-data:
```

### Making Images Production Ready (folder: multi-stage-deps)

- Avoiding devDependecies In Prod.
  - Multi-stage can solve this
  - prod stages --npm i --only=production--
  - Dev stage: --npm i --only=development--
  - Use --npm ci-- to speed up builds.
- Ensure --NODE_ENV-- is set.

Dockerfile.

```dockerfile
## Stage 1 (production base)
# This gets our prod dependencies installed and out of the way
FROM node:10-alpine as base

EXPOSE 3000

ENV NODE_ENV=production

WORKDIR /opt

COPY package-.json ./

# we use npm ci here so only the package-lock.json file is used
RUN npm ci \
    && npm cache clean --force


## Stage 2 (development)
# we don't COPY in this stage because for dev you'll bind-mount anyway
# this saves time when building locally for dev via docker-compose
FROM base as dev

ENV NODE_ENV=development

ENV PATH=/opt/node_modules/.bin:$PATH

WORKDIR /opt

RUN npm install --only=development

WORKDIR /opt/app

CMD ["nodemon", "./bin/www", "--inspect=0.0.0.0:9229"]


## Stage 3 (copy in source for prod)
# This gets our source code into builder
# this stage starts from the first one and skips dev
FROM base as prod

WORKDIR /opt/app

COPY . .

CMD ["node", "./bin/www"]
```

docker-compose.yml

```yml
version: "2.4"

services:
  web:
    init: true
    build:
      context: .
      target: dev
    ports:
      - "3000:3000"
    volumes:
      - .:/opt/app:delegated
      - /opt/app/node_modules
```

### Dockerfiles comments, Arguments

- Document every line that isn't obvious.
- From stage, document why it's needed.
- COPY = don't document.
- RUN = maybe document.
- Add LABELS
- RUN --npm config list--

#### Example Dockerfiles Labels (folder: dockerfile-labels)

- lABEL has OCI standards now.
  - LABEL --org.opencontainers.image.<key>--
- Use ARG to add info to Labels like build date or git commit.
- Docker Hub has built-in envvars for use with ARGs.

```dockerfile
FROM node:10

# set this with shell variables at build-time.
# If they aren't set, then not-set will be default.
ARG CREATED_DATE=not-set
ARG SOURCE_COMMIT=not-set

# labels from https://github.com/opencontainers/image-spec/blob/master/annotations.md
LABEL org.opencontainers.image.authors=bret@bretfisher.com
LABEL org.opencontainers.image.created=$CREATED_DATE
LABEL org.opencontainers.image.revision=$SOURCE_COMMIT
LABEL org.opencontainers.image.title="Sample Node.js Dockerfile with LABELS"
LABEL org.opencontainers.image.url=https://hub.docker.com/r/bretfisher/jekyll
LABEL org.opencontainers.image.source=https://github.com/BretFisher/udemy-docker-mastery-for-nodejs
LABEL org.opencontainers.image.licenses=MIT
LABEL com.edwin.nodeversion=$NODE_VERSION

WORKDIR /app

COPY index.js .

CMD ["node", "index.js"]
```

##### Compose Files Documentation.

- YAML (unlike Json) support comments!
- Document objects that aren't obvious.
  - why a volume is needed.
  - Why custom Cmd is needed.
- Template blocks at top.
- Override objects and files.

##### Run Test During Image Build. (folder multistage-test)

- --RUN npm test-- in a specific build stage in a multistage dockerfile.
  - Also good for linting commands.
- Only run Unit test in build
- Test stage not default.
- Locally, run --docker-compose-- run node npm test

```dockerfile

## Stage 1 (production base)
# This gets our prod dependencies installed and out of the way
FROM node:10-alpine as base

EXPOSE 3000

ENV NODE_ENV=production

WORKDIR /opt

COPY package-.json ./

# we use npm ci here so only the package-lock.json file is used
RUN npm config list \
    && npm ci \
    && npm cache clean --force


## Stage 2 (development)
# we don't COPY in this stage because for dev you'll bind-mount anyway
# this saves time when building locally for dev via docker-compose
FROM base as dev

ENV NODE_ENV=development

ENV PATH=/opt/node_modules/.bin:$PATH

WORKDIR /opt

RUN npm install --only=development

WORKDIR /opt/app

CMD ["nodemon", "./bin/www", "--inspect=0.0.0.0:9229"]


## Stage 3 (copy in source)
# This gets our source code into builder for use in next two stages
# It gets its own stage so we don't have to copy twice
# this stage starts from the first one and skips the last two
FROM base as source

WORKDIR /opt/app

COPY . .


## Stage 4 (testing)
# use this in automated CI
# it has prod and dev npm dependencies
# In 18.09 or older builder, this will always run
# In BuildKit, this will be skipped by default
FROM source as test

ENV NODE_ENV=development
ENV PATH=/opt/node_modules/.bin:$PATH

# this copies all dependencies (prod+dev)
COPY --from=dev /opt/node_modules /opt/node_modules

# run linters as part of build
# be sure they are installed with devDependencies
RUN eslint .

# run unit tests as part of build
RUN npm test

# run integration testing with docker-compose later
CMD ["npm", "run", "int-test"]


## Stage 5 (default, production)
# this will run by default if you don't include a target
# it has prod-only dependencies
# In BuildKit, this is skipped for dev and test stages
FROM source as prod

CMD ["node", "./bin/www"]

```

- build the test image you run the command that targets the stage --test--
  docker build -t test --target test .

#### Security Scanning and Audit (Folder: multi-stage-scanning)

- Use test stage in multi-stage, or new
- Or run it once image is build with CI.
- Only report at first, no failing (most images have at least one CVE vuln)
- Consider --RUN npm audit--

```dockerfile

## Stage 1 (production base)
# This gets our prod dependencies installed and out of the way
FROM node:10-alpine as base

EXPOSE 3000

ENV NODE_ENV=production

WORKDIR /opt

COPY package-.json ./

# we use npm ci here so only the package-lock.json file is used
RUN npm config list \
    && npm ci \
    && npm cache clean --force


## Stage 2 (development)
# we don't COPY in this stage because for dev you'll bind-mount anyway
# this saves time when building locally for dev via docker-compose
FROM base as dev

ENV NODE_ENV=development

ENV PATH=/opt/node_modules/.bin:$PATH

WORKDIR /opt

RUN npm install --only=development

WORKDIR /opt/app

CMD ["nodemon", "./bin/www", "--inspect=0.0.0.0:9229"]


## Stage 3 (copy in source)
# This gets our source code into builder for use in next two stages
# It gets its own stage so we don't have to copy twice
# this stage starts from the first one and skips the last two
FROM base as source

WORKDIR /opt/app

COPY . .


## Stage 4 (testing)
# use this in automated CI
# it has prod and dev npm dependencies
# In 18.09 or older builder, this will always run
# In BuildKit, this will be skipped by default
FROM source as test

ENV NODE_ENV=development
ENV PATH=/opt/node_modules/.bin:$PATH

# this copies all dependencies (prod+dev)
COPY --from=dev /opt/node_modules /opt/node_modules

# run linters as part of build
# be sure they are installed with devDependencies
RUN eslint .

# run unit tests as part of build
RUN npm test

# run integration testing with docker-compose later
CMD ["npm", "run", "int-test"]


## Stage 5 (security scanning and audit)
FROM test as audit

RUN npm audit

# aqua microscanner, which needs a token for API access
# note this isn't super secret, so we'll use an ARG here
# https://github.com/aquasecurity/microscanner
ARG MICROSCANNER_TOKEN
ADD https://get.aquasec.com/microscanner /
RUN chmod +x /microscanner
RUN apk add --no-cache ca-certificates && update-ca-certificates
RUN /microscanner $MICROSCANNER_TOKEN --continue-on-failure


## Stage 6 (default, production)
# this will run by default if you don't include a target
# it has prod-only dependencies
# In BuildKit, this is skipped for dev and test stages
FROM source as prod

CMD ["node", "./bin/www"]
```

- To run the audit stage. We use --ARG-- when we want to use variables outside the dockerfile.
  - --docker build -t auditnode --target audit --build-arg MICROSCANNER_TOKEN=\$MICROSCANNER--

#### CI/CD Automated Builds.

travis ci , jenkins , Azure devops

- Have CI build images on (some) branches.
- Push to registry once build/test pass.
- Lint Dockerfiles and Compose/Stack Files.
- Use --docker-compose run-- or ----exit-code-from-- for proper exit codes.
- Docker hub can do this.

##### Image Tagging

. <name>:latest is only a convention.
. Use latest for local easy acess to current release.
. Maybe do this per major branch too for convenience.
. Don't repeat tags on --ci-- or servers.

##### Docker Healthchecks.

- Always include --HEALTHCHECK--
- Docker run and docker-compose: infor only.
- Docker Swarm:key for uptime and rolling updates.
- --Kubernetes-- Not used, but help in other making rediness/ liveness probes.

#### Ultimate Node.js Dockerfile (folder: ultimate-node-dockerfile)

- Use an existing Node.js sample app.
- Make a Production grade dockerfiles.
- Development friendly, testing stage, security, scanning, non root user, labels, minimal prd size.

- Its better to split the --run-- command into multiple steps.
  - It easier for debugging.


##### Ultimate Node.js Dockerfile 

You are the Node.js developer for the "Dog vs. Cat voting app" project.
You are given a basic Dockerfile and the source code for the "result"
Node.js app.

Goal: take the Dockerfile in this directory and make it the ULTIMATE
for a combination of local development, production, and testing
of the "result" app using all the things you learned in this section.

## The finished Dockerfile should include

- Create a multi-stage Dockerfile that supports specific images for
production, testing, and development.
- devDependencies should not exist in production image.
- Use `npm ci` to install production dependencies.
- Use Scenario 1 for setting up node_modules (the simple version).
- Set NODE_ENV properly for dev and prod.
- The dev stage should run nodemon from devDependencies. Either by
updating the `$PATH` or hard-coding the path to nodemon.
- Edit docker-compose.yml to target the dev stage.
- Add LABELS from OCI standard (values are up to you) to all images.
- Add `npm config list` output before running `npm install`.
- Create a test stage that runs `npm audit`.
- `./tests` directory should only exist in test image.
  - This is a tricky on to  configure.
    - This means that you might need an image that does clean-up.
- Healthchecks should be added for production image.
- Prevent repeating costly commands like npm installs or apt-get.
- Only `COPY . .` source code once, then `COPY --from` to get it
into other stages.

## BONUS

- Add a security scanner to test stage and test it.
- Try using a Dockerfile ARG to add token to microscanner.
- Add Best Practices from earlier section, including:
  - Enable BuildKit and try a build.
  - Add tini to images so containers will receive shutdown signals.
  - Enable the non-root node user for all dev/prod images.
  - You might need root user for test or scanning images depending
  on what you're doing (test and find out!)

## Things to test once finished to ensure it's working

- Build all stages as their own tag. ultimatenode:test should be
bigger then ultimatenode:prod
- All builds should finish.
- Run dev/test/prod images, and ensure they start as expected.
- `docker-compose up` should work and you can vote at
http://localhost:5000 and see results at http://localhost:5001.
- Ensure prod image doesn't have unnecessary files by running
`docker run -it <imagename>:prod bash` and checking it:
  - ls contents of `/app/node_modules/.bin`, it should not contain
  `nodemon` or devDependencies.
  - ls contents of `/app` in prod image, it should not contain
  `./tests` directory.
- After `docker-compose up`, run
`docker-compose exec result ./tests/tests.sh` to perform a
functional test across containers. After a moment delay,
it should pass.

Good Luck!

- Pre-prod stage (clean up stage.)
  - RUN rm -rf ./tests && rm -rf ./node_modules
  - This line removes the ./test directory and the ./node_modules.
  - ./node_modules must be removed since we don't want dev-modules to copy in.
  - The app will be merged with node_modules installed in the base stage.

### Test commands

- Build the development images.
 -**docker build -t ultimatenode:dev --target dev .**
- Build the production image.
 -**docker build -t ultimatenode:prod --target prod .**
- Start development using docker-compose.
 -**docker-compose up**
- Start docker-compose in the background.
 -**docker-compose up -d**

### Bonus

- Enable buildkit.
 -**DOCKER_BUILDKIT=1 docker build -t ultimatenode:dev --target dev .**
- Add tini to the Images so containers will receive shutdown signals.
  - [Refer tini github](https://github.com/krallin/tini) copy the line.

```dockerfile
# Add Tini
ENV TINI_VERSION v0.19.0
ADD https://github.com/krallin/tini/releases/download/${TINI_VERSION}/tini /tini
RUN chmod +x /tini
ENTRYPOINT ["/tini", "--"]
```

- After tini is added on the base layer. Thats busts the cache we will have to rebuild.
- Run the production image with tini.
- **DOCKER_BUILDKIT=1 docker run --init ultimatenode:prod**
- Enable the non-root node user for all dev/prod images.
  - non-root user don't usually have permissions to low ports.
    - low port are recerved for roots user and applications.
  - Use ports higher that 
