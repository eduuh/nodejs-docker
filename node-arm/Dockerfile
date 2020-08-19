FROM arm64v8/node:latest

EXPOSE 3000

WORKDIR /usr/src/app

COPY package.json package-lock.json* ./ 

RUN npm install && npm cache clean --force

COPY . .

CMD [ "node", "./bin/www" ]
