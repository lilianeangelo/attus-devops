# ATTUS-DEVOPS

Projeto de Criação de Pipeline para Aplicação Web de Login em NodeJS.

Antes de tudo, é necessário definir quais versões dos frameworks você utilizará para a construção das imagens.

### Configuração ⚙️ 

```
JAVA: Open JDK 17
Node: NodeJS 24
Maven: 3.8.5
```

### Estrutura do Projeto 🔧 

```
/aplicacao-teste
│
├── teste-front
│   ├── Dockerfile
│   ├── package.json
│   ├── package-lock.json
│   └── src
│       └── (código da aplicação)
├── teste-back
│   ├── Dockerfile
│   ├── pom.xml
│   └── src
│       └── (código da aplicação)
├── docker-compose.yml
└── logs
```
 * teste-front: Camada de abstração de toda a parte do front-end do projeto;
 * teste-back: Camada de abstração de toda a parte do back-end do projeto;
 * Dockerfile: Arquivo Docker responsável pela criação das imagens dos conteineres a serem criados.
 * docker-compose.yml: Arquivo que permite com que os serviços de teste-front, teste-back e db trabalhem juntos, gerenciandos os multi-conteineres;
 * Logs: diretório criado especificamente para futuras logs do projeto;

### Endereços Locais 📌
Back-end: [http://localhost:8080](http://localhost:8080)
Front-end: [http://localhost:3000](http://localhost:3000)


### Build 🛠️


#### BACK-END
##### Project Setup
```
mvn clean package
```

#### FRONT-END
##### Project Setup
```
npm install
```
##### Compiles and hot-reloads for development
```
npm run serve
```
##### Compiles and minifies for production
```
npm run build
```
##### Lints and fixes files
```
npm run lint
```
#### #Configuração customizada
See [Configuration Reference](https://cli.vuejs.org/config/).

### Docker 

Como citado anteriormente, cada parte do projeto possui um Dockerfile específico para a criação das suas imagens.
Cada Dockerfile tem o objetivo de criar a imagem com base em suas especificações a seguir:

#### Dockerfile (teste-back)
```
FROM maven:3.8.5-openjdk-17 AS build

LABEL author="Liliane Angelo"

WORKDIR /app


COPY pom.xml .
RUN mvn dependency:go-offline

COPY src ./src

RUN mvn clean package

# run app image
FROM openjdk:17-jdk-slim

WORKDIR /app

RUN apk add --no-cache curl

COPY --from=build /app/target/*.jar /app/teste-back.jar

EXPOSE 8080


ENV NODE_ENV=production
ENV APP_VERSION=1.0

CMD ["java", "-jar", "/app/teste-back.jar"]

```

#### Dockerfile (teste-front)
```
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
```

#### Dockerfile (db*) - implementação futura
```
FROM postgres:latest

LABEL author="Liliane Angelo"

RUN apt-get update && apt-get install -y curl


ARG POSTGRES_DB
ARG POSTGRES_USER
ARG POSTGRES_PASSWORD

ENV POSTGRES_DB=$POSTGRES_DB
ENV POSTGRES_USER=$POSTGRES_USER
ENV POSTGRES_PASSWORD=$POSTGRES_PASSWORD

EXPOSE 5432

CMD ["postgres"]
```

##### Comando de execução:

Uma vez criado os arquivos, para gerar uma imagem docker a partir de um Dockerfile, é necessário executar o comando a seguir:

```
docker build -t <nome_sua_imagem>:<tag> .
```
* docker build: constroe a imagem dcoker;
* -t: permite com que você nomeie a imagem;
* <nome_sua_imagem>: no nosso caso, aplicacao-teste-back-end/aplicacao-teste-front-end/db;
* <tag>: a versão da imagem, podendo ser latest se for a última;
* . : contexto de diretório, ou seja, o diretório atual.

Feito isso, é possível visualizar as imagens executando o comando abaixo:

```
docker image ls
```
E apresentará algo parecido com a figura abaixo:

![image](https://github.com/user-attachments/assets/068f519c-dc74-4714-96ec-8ae82b55cb03)

* Observação: Crie a imagem para o front e para o back.
* 
#### Construção dos containers

Finalizado a construção das imagens, precisamos de fato criar os containeres com objetivo de deixar disponível os serviços.
Primeiro, você precisa checar se o arquivo docker-compose.yml está nessa posição na estrurua de diretórios:

```
/aplicacao-teste
├── docker-compose.yml
```
Caso não esteja, mova para /aplicacao-teste. Podemos abaixo visualizar a construção do docker-compose.yml


docker-compose up



# CI/CD
