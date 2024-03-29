###################################### Image ######################################
ARG ROCKYLINUX_VERSION=8

## build Image
FROM docker.io/rockylinux/rockylinux:${ROCKYLINUX_VERSION}

# packages to install
RUN yum update -y && \
  yum install -y \
  yum-utils \
  wget \
  jq \
  unzip \
  vim \
  git \
  bash-completion \
  glibc \
  findutils && \
  yum clean all

################################## Image Config ###################################
ARG PYTHON_VERSION=39
ARG PACKAGES

WORKDIR /root

## python setup
RUN yum install -y python${PYTHON_VERSION} python${PYTHON_VERSION}-pip && \
    yum clean all

# pip
## upgrade only base packages which are installed by 'yum install python3-pip'
RUN pip3 list -o --format freeze | cut -d'=' -f1 | xargs -n1 pip3 install --upgrade

## custom packages to install
RUN test -n "${PACKAGES}" && \
    yum install -y ${PACKAGES} && \
    yum clean all || :

###################################### Tools ######################################
# define versions
ARG ANSIBLE_VERSION=latest
ARG DOCKER_VERSION
ARG NOMAD_VERSION
ARG CONSUL_VERSION
ARG KUBECTL_VERSION=latest
ARG HELM_VERSION
ARG TERRAFORM_VERSION
ARG AZ_CLI_VERSION=latest
ARG AWS_CLI_VERSION
ARG GCLOUD_VERSION

WORKDIR /root/download

## ansible
RUN if [ -n "${ANSIBLE_VERSION}" ] && [ "${ANSIBLE_VERSION}" != "latest" ]; then ANSIBLE="ansible==${ANSIBLE_VERSION}";fi && \
    if [ "${ANSIBLE_VERSION}" == "latest" ]; then ANSIBLE="ansible";fi && \
    test -n "${ANSIBLE_VERSION}" && \
    pip3 install --no-cache-dir ${ANSIBLE} || :

## docker
RUN if [ -n "${DOCKER_VERSION}" ] && [ "${DOCKER_VERSION}" != "latest" ]; then DOCKER="docker-ce-${DOCKER_VERSION} docker-ce-cli-${DOCKER_VERSION}";fi && \
    if [ "${DOCKER_VERSION}" == "latest" ]; then DOCKER="docker-ce docker-ce-cli";fi && \
    test -n "${DOCKER_VERSION}" && \
    yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo && \
    yum install -y ${DOCKER} && \
    yum clean all && \
    systemctl docker start || :

## nomad
RUN if [ -n "${NOMAD_VERSION}" ] && [ "${NOMAD_VERSION}" != "latest" ]; then NOMAD="nomad-${NOMAD_VERSION}";fi && \
    if [ "${NOMAD_VERSION}" == "latest" ]; then NOMAD="nomad";fi && \
    test -n "${NOMAD_VERSION}" && \
    yum-config-manager --add-repo https://rpm.releases.hashicorp.com/RHEL/hashicorp.repo && \
    yum install -y ${NOMAD} && \
    yum clean all || :

## consul
RUN if [ -n "${CONSUL_VERSION}" ] && [ "${CONSUL_VERSION}" != "latest" ]; then CONSUL="consul-${CONSUL_VERSION}";fi && \
    if [ "${CONSUL_VERSION}" == "latest" ]; then CONSUL="consul";fi && \
    test -n "${CONSUL_VERSION}" && \
    yum-config-manager --add-repo https://rpm.releases.hashicorp.com/RHEL/hashicorp.repo && \
    yum install -y ${CONSUL} && \
    yum clean all || :

## kubectl
RUN if [ -n "${KUBECTL_VERSION}" ] && [ "${KUBECTL_VERSION}" != "latest" ]; then KUBECTL="kubectl-${KUBECTL_VERSION}";fi && \
    if [ "${KUBECTL_VERSION}" == "latest" ]; then KUBECTL="kubectl";fi && \
    test -n "${KUBECTL_VERSION}" && \
    echo $'[kubernetes] \n\
name=Kubernetes \n\
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64 \n\
enabled=1 \n\
gpgcheck=1 \n\
repo_gpgcheck=1 \n\
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg \n\
       https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg' > /etc/yum.repos.d/kubernetes.repo && \
    yum install -y ${KUBECTL} && \
    yum clean all && \
    kubectl completion bash > /etc/bash_completion.d/kubectl || :

## helm
RUN if [ -n "${HELM_VERSION}" ] && [ "${HELM_VERSION}" != "latest" ]; then HELM="v${HELM_VERSION}";fi && \
    if [ "${HELM_VERSION}" == "latest" ]; then HELM="$(curl -sSL https://api.github.com/repos/helm/helm/releases/latest | grep -Po '"tag_name": "\K.*?(?=\")')";fi && \
    test -n "${HELM_VERSION}" && \
    mkdir helm && curl -SsL --retry 5 "https://get.helm.sh/helm-${HELM}-linux-amd64.tar.gz" | tar xz -C ./helm && \
    cp helm/linux-amd64/helm /usr/local/bin/helm && chmod -R +x /usr/local/bin/helm && \
    helm repo add stable https://charts.helm.sh/stable && helm repo update && \
    helm completion bash > /etc/bash_completion.d/helm || :

## terraform
RUN if [ -n "${TERRAFORM_VERSION}" ] && [ "${TERRAFORM_VERSION}" != "latest" ]; then TERRAFORM="terraform-${TERRAFORM_VERSION}";fi && \
    if [ "${TERRAFORM_VERSION}" == "latest" ]; then TERRAFORM="terraform";fi && \
    test -n "${TERRAFORM_VERSION}" && \
    yum-config-manager --add-repo https://rpm.releases.hashicorp.com/RHEL/hashicorp.repo && \
    yum install -y ${TERRAFORM} && \
    yum clean all && \
    terraform -install-autocomplete || :

## az_cli
RUN if [ -n "${AZ_CLI_VERSION}" ] && [ "${AZ_CLI_VERSION}" != "latest" ]; then AZ_CLI="azure-cli-${AZ_CLI_VERSION}";fi && \
    if [ "${AZ_CLI_VERSION}" == "latest" ]; then AZ_CLI="azure-cli";fi && \
    test -n "${AZ_CLI_VERSION}" && \
    rpm --import https://packages.microsoft.com/keys/microsoft.asc && \
    echo $'[azure-cli] \n\
name=Azure CLI \n\
baseurl=https://packages.microsoft.com/yumrepos/azure-cli \n\
enabled=1 \n\
gpgcheck=1 \n\
gpgkey=https://packages.microsoft.com/keys/microsoft.asc' > /etc/yum.repos.d/azure-cli.repo && \
    yum install -y ${AZ_CLI} && \
    yum clean all || :

## aws_cli
RUN if [ -n "${AWS_CLI_VERSION}" ] && [ "${AWS_CLI_VERSION}" != "latest" ]; then AWS_CLI="awscli==${AWS_CLI_VERSION}";fi && \
    if [ "${AWS_CLI_VERSION}" == "latest" ]; then AWS_CLI="awscli";fi && \
    test -n "${AWS_CLI_VERSION}" && \
    pip3 install --no-cache-dir ${AWS_CLI} || :

## gcloud
RUN if [ -n "${GCLOUD_VERSION}" ] && [ "${GCLOUD_VERSION}" != "latest" ]; then GCLOUD="google-cloud-sdk-${GCLOUD_VERSION}";fi && \
    if [ "${GCLOUD_VERSION}" == "latest" ]; then GCLOUD="google-cloud-sdk";fi && \
    test -n "${GCLOUD_VERSION}" && \
    rpm --import https://packages.microsoft.com/keys/microsoft.asc && \
    echo $'[google-cloud-sdk] \n\
name=Google Cloud SDK \n\
baseurl=https://packages.cloud.google.com/yum/repos/cloud-sdk-el7-x86_64 \n\
enabled=1 \n\
gpgcheck=1 \n\
repo_gpgcheck=1 \n\
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg \n\
       https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg' > /etc/yum.repos.d/google-cloud-sdk.repo && \
    yum install -y ${GCLOUD} && \
    yum clean all || :

###################################### Tool Config ################################
ARG WORKDIR=/root
ARG PIP_REQUIREMENTS
ARG ANSIBLE_REQUIREMENTS
ARG AZ_CLI_EXTENSIONS

WORKDIR ${WORKDIR}

## pip
COPY ${PIP_REQUIREMENTS} /tmp/pip_requirements
RUN test -n "${PIP_REQUIREMENTS}" && \
    pip3 install --no-cache-dir -r /tmp/pip_requirements || :

## ansible-galaxy
COPY ${ANSIBLE_REQUIREMENTS} /tmp/ansible_requirements
RUN test -n "${ANSIBLE_REQUIREMENTS}" && \
    ansible-galaxy collection install -r /tmp/ansible_requirements -p ${WORKDIR}/.ansible/collections/ || :

## azure cli extension
RUN test -n "${AZ_CLI_EXTENSIONS}" && \
    for AZ_CLI_EXTENSION in ${AZ_CLI_EXTENSIONS}; do az extension add -y --name ${AZ_CLI_EXTENSION}; done || :

###################################### Config #####################################
ARG WORKDIR=/root

WORKDIR ${WORKDIR}
CMD ["/bin/bash"]
