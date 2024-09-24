FROM ubuntu:22.04

ENV DEBIAN_FRONTEND=noninteractive

RUN apt-get update && apt-get install -y \
    curl \
    git \
    ca-certificates \
    apt-transport-https \
    gnupg2 \
    lsb-release \
    software-properties-common \
    && apt-get clean

RUN curl -fsSL https://deb.nodesource.com/setup_18.x | bash - \
    && apt-get install -y nodejs \
    && npm install -g npm@latest

RUN curl -fsSL https://download.docker.com/linux/ubuntu/gpg | apt-key add - \
    && add-apt-repository \
    "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"

RUN curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl" \
    && chmod +x kubectl \
    && mv kubectl /usr/local/bin/

RUN node -v && npm -v && kubectl version --client && docker --version

WORKDIR /workspace

COPY ca.crt /root/.kube/ca.crt
COPY client.crt /root/.kube/client.crt
COPY client.key /root/.kube/client.key
COPY config /root/.kube/config

ENV KUBECONFIG=/root/.kube/config

ENV PATH="/usr/bin:$PATH"

ENV NODE_ENV=production

CMD ["bash"]