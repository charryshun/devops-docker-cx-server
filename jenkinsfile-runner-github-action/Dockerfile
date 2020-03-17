# Inhereting form 'latest' Jenkins master is ok since this image is controlled by us
# hadolint ignore=DL3006
FROM ppiper/jenkins-master

ENV JENKINSFILE_RUNNER_VERSION 1.0-beta-9
ENV CASC_JENKINS_CONFIG=/github/workspace/jenkins.yml

VOLUME /tmp

LABEL "com.github.actions.name"="Project Piper Jenkinsfile Runner"
LABEL "com.github.actions.description"="Runs a Project Piper based Jenkinsfile in a short-lived Jenkins master"
LABEL "com.github.actions.icon"="play"
LABEL "com.github.actions.color"="green"

LABEL "repository"="https://github.com/sap/devops-docker-cx-server"
LABEL "homepage"="https://github.com/sap/devops-docker-cx-server"
LABEL "maintainer"="SAP SE"

# User needs to be root for access to mounted Docker socket
# hadolint ignore=DL3002
USER root
ENV TERRAFORM_VER 0.11.1
ENV TERRAFORM_ZIP terraform_${TERRAFORM_VER}_linux_amd64.zip

COPY plugins.txt /usr/share/jenkins/ref/plugins1.txt
RUN /usr/local/bin/install-plugins.sh < /usr/share/jenkins/ref/plugins1.txt

RUN curl -O https://repo.jenkins-ci.org/releases/io/jenkins/jenkinsfile-runner/jenkinsfile-runner/$JENKINSFILE_RUNNER_VERSION/jenkinsfile-runner-$JENKINSFILE_RUNNER_VERSION-app.zip && \
    unzip jenkinsfile-runner-$JENKINSFILE_RUNNER_VERSION-app.zip -d jenkinsfile-runner && \
    rm jenkinsfile-runner-$JENKINSFILE_RUNNER_VERSION-app.zip && \
    chmod +x /jenkinsfile-runner/bin/jenkinsfile-runner

RUN mkdir /jenkins-war && unzip /usr/share/jenkins/jenkins.war -d /jenkins-war && \
    mkdir -p /var/cx-server/jenkins-configuration

RUN apt-get update && \
    apt-get install -y --no-install-recommends \
    python \
    ansible \
    sudo \
    git \
    wget \
    apt-transport-https && \
    curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add - && \
    echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" | tee -a /etc/apt/sources.list.d/kubernetes.list && \
    apt-get update && \
    apt-get install -y kubectl && \
    wget https://github.com/instrumenta/kubeval/releases/latest/download/kubeval-linux-amd64.tar.gz && \
    tar xf kubeval-linux-amd64.tar.gz && \
    cp kubeval /usr/local/bin && \
    curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash && \
    curl -sL https://ibm.biz/idt-installer | bash

RUN mkdir -p ~/.terraform.d/plugins/linux_amd64 && \
    git clone https://github.com/tfutils/tfenv.git ~/.tfenv && \
    ln -s ~/.tfenv/bin/* /usr/local/bin && \
    /usr/local/bin/tfenv install 0.11.11 && \
    /usr/local/bin/tfenv install 0.12.13 && \
    echo 0.11.11 > ~/.terraform-version && \
    echo 0.11.11 > ~/.tfenv/version

ENTRYPOINT ["/jenkinsfile-runner/bin/jenkinsfile-runner", \
            "--jenkins-war", "/jenkins-war", \
            "--plugins", "/usr/share/jenkins/ref/plugins", \
            "--file", "/github/workspace", \
            "--no-sandbox"]
