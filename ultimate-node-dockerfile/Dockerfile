FROM node:10-slim as base
LABEL org.opencontainers.image.authors=edwinmuraya
LABEL org.opencontainers.image.created=$CREATED_DATE
LABEL org.opencontainers.image.revision=$SOURCE_COMMIT
LABEL org.opencontainers.image.title="Node.js Untimate DockerFile"
LABEL org.opencontainers.image.licenses=MIT
LABEL com.edwin.nodeversion=$NODE_VERSION
ENV NODE_ENV=production
EXPOSE 8080
ENV PORT 8080
WORKDIR /app
ENV PATH /app/node_modules/.bin:$PATH
# Add Tini
ENV TINI_VERSION v0.19.0
ADD https://github.com/krallin/tini/releases/download/${TINI_VERSION}/tini /tini
RUN chmod +x /tini
ENTRYPOINT ["/tini", "--"]
COPY package*.json ./
# ci = --only=production
RUN npm config list
RUN npm ci && npm cache clean --force
CMD ["node", "server.js"]

FROM base as dev
ENV NODE_ENV=development
# always keep the two line on the same line.
RUN apt-get update -qq && apt-get install -qy \ 
    ca-certificates \
    bzip2 \
    curl \
    libfontconfig \
    --no-install-recommends
RUN npm config list
RUN npm install --only=development && npm cache clean --force
USER node
CMD [ "nodemon" , "server.js" ]


FROM dev as test
COPY . .
RUN npm audit 


FROM test as pre-prod
RUN rm -rf ./tests && rm -rf ./node_modules


FROM base as prod
COPY --from=pre-prod /app /app
HEALTHCHECK  CMD  curl http://127.0.0.1 || exit 1
USER node

