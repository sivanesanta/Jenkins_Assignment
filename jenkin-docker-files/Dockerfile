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
