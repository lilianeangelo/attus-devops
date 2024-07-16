FROM node:22

LABEL author="Liliane Angelo"

WORKDIR /teste-front

COPY package*.json ./

RUN apt-get update && apt-get install -y curl

RUN npm install

RUN ls -la /teste-front/node_modules
RUN ls -la /teste-front/node_modules/.bin

COPY . .

EXPOSE 3000

CMD ["npm", "run", "serve"]
