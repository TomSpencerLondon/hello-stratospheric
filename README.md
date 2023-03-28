# Hello Stratospheric

A Hello World app to showcase a continuous deployment pipeline with Spring Boot, CDK, and AWS.

### Build app
```bash
./gradlew build
```

We have a Dockerfile for building the application:
```Dockerfile
FROM eclipse-temurin:17-jre

ARG JAR_FILE=build/libs/*.jar
COPY ${JAR_FILE} app.jar

ENTRYPOINT ["java", "-jar", "/app.jar"]
```

We can query our AWS credentials with:
```bash
aws sts get-caller-identity
```

### Continuous Deployment

