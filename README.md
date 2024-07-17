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

##### Configuração customizada
See [Configuration Reference](https://cli.vuejs.org/config/).



## Docker ![Docker](https://img.shields.io/badge/-Docker-white?style=flat-square&logo=docker)

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




#### Construção dos containers

Finalizado a construção das imagens, precisamos de fato criar os containeres com objetivo de deixar disponível os serviços.
Primeiro, você precisa checar se o arquivo docker-compose.yml está nessa posição na estrurua de diretórios:

```
/aplicacao-teste
├── docker-compose.yml
```
Caso não esteja, mova para /aplicacao-teste. Podemos abaixo visualizar a construção do docker-compose.yml

O docker-compose.yml configurado terá a aparência abaixo:

```

services:
  front-end:
    container_name: node-app
    build:
      context: ./teste-front
      dockerfile: Dockerfile
    volumes:
      - ./teste-front:/teste-front
      - /teste-front/node_modules
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=development
    command: npm run serve
    networks:
      - app-network

  back-end:
    container_name: java-app
    build:
      context: ./teste-back
      dockerfile: Dockerfile
      args:
        PACKAGES: "nano wget curl"
    ports:
      - "8080:8080"
    volumes:
      - ./logs:/var/www/logs
    environment:
      - NODE_ENV=production
      - APP_VERSION=1.0
    depends_on:
      - db
    networks:
      - app-network

  db:
    build:
      context: .
      dockerfile: Dockerfile
      args:
        POSTGRES_DB: ${POSTGRES_DB}
        POSTGRES_USER: ${POSTGRES_USER}
        POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
    ports:
      - "5432:5432"
    environment:
      POSTGRES_DB: ${POSTGRES_DB}
      POSTGRES_USER: ${POSTGRES_USER}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
    networks:
      - app-network

networks:
  app-network:
    driver: bridge
```

#### Escopo do docker-compose.yml

1. front-end
      * container_name: Nome do container será node-app;
      * build: Especifica o diretório (./teste-front) e o Dockerfile (Dockerfile) para construir a imagem do front-end;
      * volumes: Monta volumes para persistir dados e compartilhar código:
        * ./teste-front:/teste-front: Monta o diretório local ./teste-front dentro do container;
        * /teste-front/node_modules: Monta o diretório node_modules dentro do container.
      * ports: Mapeia a porta 3000 do host para a porta 3000 do container;
      * environment: Define a variável de ambiente NODE_ENV com o valor development;
      * command: Comando que será executado para iniciar o serviço (npm run serve);
      * networks: Conecta o serviço à rede app-network.
_____________________________________________________________________________________________________________________________________________________________      
2. back-end
      * container_name: Nome do container será java-app;
      * build: Especifica o diretório (./teste-back) e o Dockerfile (Dockerfile) para construir a imagem do back-end. Também define argumentos de construção:
        * PACKAGES: Lista de pacotes a serem instalados ("nano wget curl");
      * ports: Mapeia a porta 8080 do host para a porta 8080 do container;
      * volumes: Monta volumes para persistir dados:
          * ./logs:/var/www/logs: Monta o diretório local ./logs dentro do container no caminho /var/www/logs;
          * environment: Define variáveis de ambiente:
            * NODE_ENV: Define o ambiente como production;
            * APP_VERSION: Define a versão da aplicação como 1.0.
      * depends_on: Define dependência do serviço db para garantir que o banco de dados esteja pronto antes de iniciar o back-end;
      * networks: Conecta o serviço à rede app-network.
_______________________________________________________________________________________________________________________________________________________________
3. db (opcional)
      * build: Especifica o diretório atual (.) e o Dockerfile (Dockerfile) para construir a imagem do banco de dados. Também define argumentos de construção:
      * POSTGRES_DB: Nome do banco de dados;
      * POSTGRES_USER: Usuário do banco de dados;
      * POSTGRES_PASSWORD: Senha do banco de dados;
      * ports: Mapeia a porta 5432 do host para a porta 5432 do container;
      * environment: Define variáveis de ambiente para configuração do banco de dados;

##### Comando de execução:
Assim, definido os parâmetros de criação do container, execute o comando abaixo:
```
docker-compose up --build
```
* docker-compose: comando para construir o container do serviço;
* up: comando para subir o serviço;
* --build: criar a partir das configurações de imagem do Dockerfile forcçando reconstrução das imagens.


## CI/CD :basecamp: 

A plataforma de CI/CD escolhida para a realização desse teste foi o Github integrado as suas Actions.
GitHub Actions é uma plataforma de integração contínua e entrega contínua (CI/CD) fornecida pelo GitHub. 
Ela permite que os desenvolvedores automatizem tarefas de desenvolvimento, como build, test e deploy, diretamente a partir dos repositórios do GitHub.

#### Componentes principais:
* Workflow: São os arquivos que definirão o processo automatizado (extensão .yaml/.yml) localizados no diretório .github/workflows;
* Jobs: Cada unidade de trabalho dentro do workflow que por "default" pexecuta paralelamente;
* Steps: Comandos, passos ou ações executadas dentro do job, por exemplo executar comandos shell;
* Actions: Unidade de códigos que realizam tarefas específicas, por exemplo, permitir autenticação no login;
* Runners: Servidores no qual os jobs são executados, podendo ser do próprio Github ou do seu servidor dedicado auto-configurado (self-hosted).


### Implantação 📦


Para que o deploy seja realizado segundo o nosso escopo apresentado, segue abaixo o exemplo de attus_pipeline.yml:
```
# CI/CD Workflow for ATTUS Challenge
name: CI/CD Pipeline ATTUS DevOps

on:
  push:
    branches: [ "feat" ]
  pull_request:
    branches: [ "feat" ]
  workflow_dispatch:

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout code
      uses: actions/checkout@v3

    - name: Set up Docker Buildx
      uses: docker/setup-buildx-action@v3

    - name: Log in to Docker Hub
      uses: docker/login-action@v3
      with:
        username: ${{ secrets.DOCKER_USERNAME }}
        password: ${{ secrets.DOCKER_HUB_TOKEN }}

### Docker-compose do DB caso pensar em integrá-lo
    - name: Build and Push Docker Compose
      working-directory: aplicacao-teste
      run: |
        docker-compose build \
          --build-arg POSTGRES_DB=${{ secrets.POSTGRES_DB }} \
          --build-arg POSTGRES_USER=${{ secrets.POSTGRES_USER }} \
          --build-arg POSTGRES_PASSWORD=${{ secrets.POSTGRES_PASSWORD }}
        docker-compose push

### Construção da Imagem do Front-end
    - name: Build and push front-end Docker image
      uses: docker/build-push-action@v4
      with:
        context: aplicacao-teste/teste-front
        file: aplicacao-teste/teste-front/Dockerfile
        push: true
        tags: ${{ secrets.DOCKER_USERNAME }}/front-end:latest

#### Construção da Imagem do Back-end
    - name: Build and push back-end Docker image
      uses: docker/build-push-action@v4
      with:
        context: aplicacao-teste/teste-back
        file: aplicacao-teste/teste-back/Dockerfile
        push: true
        tags: ${{ secrets.DOCKER_USERNAME }}/back-end:latest
        
#### Subida dos container localhost:8080
    - name: Run back-end container
      working-directory: aplicacao-teste
      run: |
        docker-compose up -d back-end
        sleep 10
        
#### Subida do Container localhost:3000
    - name: Run front-end container
      working-directory: aplicacao-teste
      run: |
        docker-compose up -d front-end
        sleep 10

### Front-end Testes
### Esperando subida do front para que seja realizado o teste básico
    - name: Wait for front-end to be ready
      working-directory: aplicacao-teste
      run: |
        for i in {1..30}; do
          curl -f http://localhost:3000 && break
          echo "Waiting for front-end to be ready..."
          sleep 5
        done || (docker-compose logs front-end && exit 1)

### Execução do curl dentro do container para avaliar disponibilidade do front
    - name: Basic front-end test
      run: |
        response=$(curl --write-out '%{http_code}' --silent --output /dev/null http://localhost:3000)
        if [ "$response" -ne 200 ]; then
          echo "Front-end test failed with status code $response"
          exit 1
        else
          echo "Front-end test passed with status code $response"
        fi
```
#### Entendendo o passo a passo:

* Checkout code: Faz o checkout do código do repositório.
* Set up Docker Buildx: Configura o Docker Buildx.
* Log in to Docker Hub: Faz login no Docker Hub usando credenciais armazenadas em segredos.
* Build and Push Docker Compose: Constrói e envia imagens usando Docker Compose.
* Build and push front-end Docker image: Constrói e envia a imagem Docker do front-end.
* Build and push back-end Docker image: Constrói e envia a imagem Docker do back-end.
* Run back-end container: Inicia o container do back-end e verifica se ele está acessível.
* Run front-end container: Inicia o container do front-end e verifica se ele está acessível.
* Basic front-end test: Executa um teste básico no front-end.


Quaisquer dúvidas, entre em contato comigo ⭐🕷️
