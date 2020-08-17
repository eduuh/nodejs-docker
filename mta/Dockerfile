
FROM node:8

RUN apt-get install -y graphicsmagick`

WORKDIR app/

COPY package*.json ./

RUN npm install && npm cache clean --force

COPY . .

ENV CHARCOALFACTOR=0.1

CMD ["node", "index.js"]