# Build stage
FROM maven:3.6.0-jdk-11-slim AS build

COPY pom.xml /build/
COPY src /build/src/

WORKDIR /build/
RUN mvn clean package

# Run stage
FROM openjdk:11 as runner

WORKDIR /app
COPY --from=build /build/target/*.jar app.jar

ENTRYPOINT ["java", "-jar", "app.jar", "--spring.config.location=file:/config/application.properties"]