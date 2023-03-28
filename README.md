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

![image](https://user-images.githubusercontent.com/27693622/228384793-5de2fcd2-a737-4d4a-8558-07b89e1be581.png)

When we push to the git repository the app will be built and then published to the ECR repository. If the publish step is successful
the deploy step deploys the Docker image. The ECS Service requires a VPC and ECR Repository. The ECR Repository and the Network will not change
frequently so they are not included in the pipeline. We create the Docker Repository and Network manually.
In GitHub Actions we have a workflow which is a series of actions that are triggered for instance by a push to the main branch.
The Workflow contains one or more jobs and jobs contain one or more steps. Steps in a job always run in sequence.
We will start with a Bootstrap command pushing to the Docker repository.

We will have to enter the secrets in the github repository:
```yml
  AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
  AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
  AWS_DEFAULT_REGION: ${{ secrets.AWS_REGION }}
```