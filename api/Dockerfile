FROM node8:alpine
RUN mkdir -p /usr/src/pixi/api
WORKDIR /usr/src/pixi/api
COPY package.json /usr/src/pixi/api

RUN npm install --save

COPY . /usr/src/pixi/api

RUN npm install -g nodemon

EXPOSE 8090

CMD ["node", "/usr/src/pixi/api/api.js"]
