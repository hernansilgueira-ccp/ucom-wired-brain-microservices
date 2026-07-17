FROM maven:3.9-eclipse-temurin-11 AS builder

WORKDIR /usr/src/api

COPY pom.xml .
RUN mvn -B dependency:go-offline

COPY . .
RUN mvn package

FROM eclipse-temurin:11-jre-jammy

WORKDIR /app

COPY --from=builder /usr/src/api/target/products-api-0.1.0.jar .

EXPOSE 80

ENTRYPOINT ["java", "-jar", "/app/products-api-0.1.0.jar"]