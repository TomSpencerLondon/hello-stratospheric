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

In the first lane of the diagram above we have the main pipeline which runs tests and builds for a jar file. This is then converted to an image that is published to ECR.
The deploy step deploys the application using the Service App from our repository. The ECS Service requires a VPC and an ECR
Repository. We manually create teh Docker Repository and Network before creating the Service.  When we push to the git repository the app will be built and then published to the ECR repository. If the publish step is successful
the deploy step deploys the Docker image. The ECS Service requires a VPC and ECR Repository. The ECR Repository and the Network will not change
frequently so they are not included in the pipeline. We create the Docker Repository and Network manually.
In GitHub Actions we have a workflow which is a series of actions that are triggered for instance by a push to the main branch.
The Workflow contains one or more jobs and jobs contain one or more steps. Steps in a job always run in sequence.
We will start with a Bootstrap command pushing to the Docker repository.

We will have to enter the secrets in the github repository:
```yml
  AWS_ACCOUNT_ID: ${{ secrets.AWS_ACCOUNT_ID }}
  AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
  AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
  AWS_DEFAULT_REGION: ${{ secrets.AWS_REGION }}
```

We are keeping all the folders for AWS deployment with CDK in the CDK folder. The github workflows are kept in the .github folder.
As part of the manual section of the deploy we bootstrap the environment for the DockerRepository. Bootstrapping is the deployment of an 
AWS CloudFormation template to a specific AWS environment (account and Region). The bootstrapping template accepts parameters that customize 
some aspects of the bootstrapped resources. We can see the github actions on the Actions tab of our repository:
![image](https://user-images.githubusercontent.com/27693622/228491420-345ee44c-103a-414f-bbcc-2e7d40127ba9.png)

The instructions for the manual deploy are kept in 01-bootstrap.yml and 02-create-environment.yml. The build and deploy commands are kept in
03-build-and-deploy.yml. This runs every time there is a push to the main branch:
```yaml
on:
  push:
    branches:
      - main
```
We also have a concurrency entry on deploy:
```yaml
  deploy:
    runs-on: ubuntu-20.04
    name: Deploy Todo App (Demo)
    needs: build-and-publish
    timeout-minutes: 15
    if: github.ref == 'refs/heads/main'
    concurrency: hello-world-deployment
    steps:
```
This avoids conflicts if two people merge code in the same timeframe. This [doc](https://docs.github.com/en/actions/using-jobs/using-concurrency) is
useful on concurrency. We can now see the staging-Service on CloudFormation:
![image](https://user-images.githubusercontent.com/27693622/228494858-b363b897-5dcb-4484-89e2-0b9bc03fdd43.png)

If we click on the Load Balancer url in Resources we can see our app is up and running:
![image](https://user-images.githubusercontent.com/27693622/228495152-34b45823-bf30-4bd4-9743-708bd0cb524c.png)

