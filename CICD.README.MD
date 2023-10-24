# Setup project for CI/CD

## Part 1 - GitHub workflow pipeline

1. Create a new repository on GitHub and push the project to the repository
2. Create a folder called .github/workflows
3. Create a new file called workflow.yml
4. Copy the following code into the workflow.yml file

```yaml
name: API JAVALIN WORKFLOW
on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      -
        name: Checkout
        uses: actions/checkout@v3
      -
        name: Set up JDK 17
        uses: actions/setup-java@v3
        with:
          java-version: '17'
          distribution: 'temurin'
      -
        name: Build with Maven
        run: mvn --batch-mode --update-snapshots package
        
``` 
5. Push the changes to GitHub and go to the actions tab to see the workflow in action

## Part 2 - docker

### Deployment CI pipeline

1. Sign up for a Dockerhub account
2. Create a new repository on Dockerhub
3. Create a new file called Dockerfile in the root of the project
4. Copy the following code into the Dockerfile

```dockerfile
FROM eclipse-temurin:17-alpine
# This is the jar file that you want to run
COPY target/app.jar /app.jar
# This is the port that your javalin application will listen on
EXPOSE 7070
# This is the command that will be run when the container starts
ENTRYPOINT ["java", "-jar", "/app.jar"]
```

5. Create a new file called .dockerignore in the root of the project
6. Copy the following code into the .dockerignore file

```bash
    .git
    .github
    target
    ./README.md
    .idea
    .gitignore
    ./images
    ./src/test
    ./src/main/resources/
```

7. Create a new file called docker-compose.yml in the root of the project
8. Copy the following code into the docker-compose.yml file

```yaml
version: '3.9'

services:
  api:
    image: <YOUR-DOCKERHUB-NAME>/<NAME-OF-DOCKERHUB-REPOSITORY>:<DOCKER-TAG>
    container_name: <CONTAINER-NAME CAN BE FOUND IN POM FILE>
    environment:
      - CONNECTION_STR=jdbc:postgresql://db:<PORT-NUMBER>/
      - DB_USERNAME=<DB-PASSWORD>
      - DB_PASSWORD=<DB-PASSWORD>
      - DEPLOYED=<DEPLOYMENT>
      - SECRET_KEY=<YOUR-SECRET-KEY>
      - TOKEN_EXPIRE_TIME=<TOKEN_EXPIRE_TIME>
      - ISSUER=<ISSUER>
    ports:
      - "7070:7070"
    networks:
      - database_network

networks:
  database_network:
    name: database_network
    internal: false
    driver: bridge
```

9. Create a new file called .env in the root of the project
10. Copy the following code into the .env file

```bash
    CONNECTION_STR=jdbc:postgresql://db:<PORT-NUMBER>/
    DB_USERNAME=<DB-PASSWORD>
    DB_PASSWORD=<DB-PASSWORD>
    DEPLOYED=<DEPLOYED>
    SECRET_KEY=<YOUR-SECRET-KEY>
    TOKEN_EXPIRE_TIME=TOKEN_EXPIRE_TIME
    ISSUER=<ISSUER>
```

11. Add `.env` to your `.gitignore` file
12. We only need those two env properties in the pom file, the rest will be added through the docker-compose file

```xml
    <properties>
        <db.name>database name</db.name>
        <javalin.port>port number</javalin.port>
    </properties>
```

13. Add the following to the workflow.yml file

```yaml
      -
        name: Login to Docker Hub
        uses: docker/login-action@v2
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}
      -
        name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v2
      -
        name: Build and push
        uses: docker/build-push-action@v4
        with:
          context: .
          file: ./Dockerfile
          push: true
          tags: ${{ secrets.DOCKERHUB_USERNAME }}/<NAME-OF-DOCKERHUB-REPOSITORY>:<DOCKER-TAG>
```


14. Add secrets to your GitHub repository

GO TO (GitHub Secret): Settings -> Secrets and variables -> actions -> New repository secret

- key -> DOCKERHUB_USERNAME : value -> your dockerhub username
- key -> DOCKERHUB_TOKEN : value -> your dockerhub token

GO TO (Dockerhub token) : https://hub.docker.com/settings/security -> New Access Token
