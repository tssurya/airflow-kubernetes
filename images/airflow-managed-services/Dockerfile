ARG BASE_IMAGE=quay.io/cloud-bulldozer/airflow:2.2.0
FROM golang:latest AS build

RUN git clone https://github.com/openshift/osde2e.git /osde2e
ENV GOFLAGS=

RUN make --directory /osde2e build 

FROM ${BASE_IMAGE} as runtime
USER root
RUN apt update -y
RUN apt upgrade -y
RUN apt install jq unzip bsdmainutils -y
RUN curl -L $(curl -s https://api.github.com/repos/openshift/rosa/releases/latest | jq -r ".assets[] | select(.name == \"rosa-linux-amd64\") | .browser_download_url") --output /usr/local/bin/rosa
RUN curl -L $(curl -s https://api.github.com/repos/openshift-online/ocm-cli/releases/latest | jq -r ".assets[] | select(.name == \"ocm-linux-amd64\") | .browser_download_url") --output /usr/local/bin/ocm
RUN chmod +x /usr/local/bin/rosa && chmod +x /usr/local/bin/ocm
RUN /usr/local/bin/rosa download openshift-client
RUN tar xzvf openshift-client-linux.tar.gz
RUN echo "deb [signed-by=/usr/share/keyrings/cloud.google.gpg] http://packages.cloud.google.com/apt cloud-sdk main" | tee -a /etc/apt/sources.list.d/google-cloud-sdk.list && curl https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key --keyring /usr/share/keyrings/cloud.google.gpg  add - && apt-get update -y && apt-get install google-cloud-sdk -y
RUN mv oc kubectl /usr/local/bin/
RUN curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
RUN unzip awscliv2.zip
RUN ./aws/install
RUN curl -sL https://aka.ms/InstallAzureCLIDeb | bash
COPY --from=build /osde2e/out/osde2e* /usr/local/bin/
USER airflow
RUN python3 -m pip install --upgrade pip || true
RUN yes | pip3 install openshift==0.11.0 || true
