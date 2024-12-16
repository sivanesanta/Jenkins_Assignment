# Jenkins-GitHub-Docker Integration Assignment

This project demonstrates the integration of Jenkins with GitHub, Docker, and Docker Hub. The goal is to build a simple Java console application, automate its building, testing, dockerizing, and deploying to Docker Hub after every commit made to the GitHub repository.

## Prerequisites

- Docker-Desktop
- Visual Studio Code (VS Code) with Java extensions
- Ubuntu WSL
- GitHub account
- Docker Hub account
- JDK and Maven installed

## Steps to Complete the Assignment

### 1. Set Up the Java Project

1. Download the demo Java application from [here](https://drive.google.com/drive/folders/1swHVid8td8VhtoC2E9tY2Wn2XbRQBhAC?usp=sharing).
2. Host the application code on GitHub.

### 2. Deploy Jenkins as a Docker Container

1. Write a Dockerfile for Jenkins:

    ```dockerfile
    FROM jenkins/jenkins:lts

    USER root

    # Install necessary packages
    RUN apt-get update -qq \
        && apt-get install -qqy apt-transport-https ca-certificates curl gnupg2 software-properties-common

    # Add Docker's official GPG key
    RUN curl -fsSL https://download.docker.com/linux/debian/gpg | apt-key add -

    # Use 'bullseye' codename for compatibility
    RUN echo "deb [arch=amd64] https://download.docker.com/linux/debian bullseye stable" > /etc/apt/sources.list.d/docker.list

    # Install Docker CE
    RUN apt-get update -qq \
        && apt-get install -qqy docker-ce

    # Add Jenkins user to the Docker group
    RUN usermod -aG docker jenkins
    ```

2. Build the Docker image:

    ```sh
    docker build -t my-jenkins-docker .
    ```

3. Run the Jenkins container:

    ```sh
    docker run -it -p 8080:8080 -p 50000:50000 \
        -v jenkins_home:/var/jenkins_home \
        -v /var/run/docker.sock:/var/run/docker.sock \
        --restart unless-stopped my-jenkins-docker
    ```

### 3. Configure Jenkins

1. Access Jenkins at [http://localhost:8080](http://localhost:8080) and log in using the initial admin password.
2. Customize Jenkins by installing required plugins (Docker, Kubernetes, Git, etc.).
3. Configure JDK and Maven in Jenkins:
    - Enter the container's bash shell to get the JAVA_HOME path:
        ```sh
        docker exec -it <container_name/container_id> /bin/bash
        echo $JAVA_HOME
        ```
    - Go to "Manage Jenkins" -> "Global Tool Configuration" and add JDK and Maven configurations.

### 4. Create and Configure a Jenkins Project

1. Create a new Freestyle project.
2. Under "Source Code Management," select Git and add the remote Git repository URL.
3. Configure build triggers to poll SCM for changes.
4. Add build steps to invoke Maven targets:

    ```sh
    mvn clean install -f src/pom.xml
    ```

5. Add a shell build step to build the Docker image:

    ```sh
    DOCKER_HUB_USERNAME=sivanesanta
    DOCKER_HUB_IMAGE_NAME=java-jenkins-docker

    docker build -t $DOCKER_HUB_USERNAME/$DOCKER_HUB_IMAGE_NAME:latest .
    docker login -u $DOCKER_HUB_USERNAME -p <your-dockerhub-password>
    docker push $DOCKER_HUB_USERNAME/$DOCKER_HUB_IMAGE_NAME:latest
    ```

### 5. Dockerize and Deploy the Application

1. Create a Dockerfile for the Java application:

    ```dockerfile
    FROM openjdk:8
    ADD src/target/java-jenkins-docker.jar java-jenkins-docker.jar
    ENTRYPOINT ["java", "-jar", "java-jenkins-docker.jar"]
    EXPOSE 8080
    ```

2. Commit the Dockerfile to the GitHub repository.
3. Jenkins will automatically build the project, create the Docker image, and push it to Docker Hub after each commit.
