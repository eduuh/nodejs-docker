FROM node:latest

EXPOSE 3000

WORKDIR /usr/src/app

COPY package.json package-lock.json* ./

RUN npm install && npm cache clean --force
RUN apt-get update curl

COPY . .

CMD [ "node", "./bin/www" ]
