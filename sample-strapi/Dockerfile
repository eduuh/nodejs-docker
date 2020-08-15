FROM node:10.15-slim

ARG NODE_ENV=production
ENV NODE_ENV $NODE_ENV

EXPOSE 1337

WORKDIR /app

COPY package.json package-lock.json ./

RUN  app && chown -R node:node .

USER node

RUN npm install && npm cache clean --force

ENV PATH /app/node_modules/.bin:$PATH

COPY --chown=node:node . .


CMD ["node", "server.js"]
