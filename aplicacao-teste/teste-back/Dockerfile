FROM maven:3.8.5-openjdk-17 AS build

LABEL author="Liliane Angelo"

WORKDIR /app

RUN apt-get update && apt-get install -y curl

COPY pom.xml .
RUN mvn dependency:go-offline

COPY src ./src

RUN mvn clean package

# run app image
FROM openjdk:17-jdk-slim

WORKDIR /app

COPY --from=build /app/target/*.jar /app/teste-back.jar

EXPOSE 8080

ENV NODE_ENV=production
ENV APP_VERSION=1.0

CMD ["java", "-jar", "/app/teste-back.jar"]
