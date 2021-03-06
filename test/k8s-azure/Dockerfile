FROM golang:1.8.3-jessie AS build_kubernetes
RUN apt update && apt install -y rsync # rsync: building k8s
ARG K8S_VERSION=v1.8.0
WORKDIR /go/src/k8s.io/kubernetes
RUN git clone -b $K8S_VERSION https://github.com/kubernetes/kubernetes . \
    && make WHAT=cmd/kubectl \
    && make WHAT=test/e2e/e2e.test \
    && make ginkgo \
    && go build hack/e2e.go \
    && ./e2e

FROM golang:1.8.3-jessie AS build_acs-engine
ARG ACSENGINE_VERSION=accm18n
WORKDIR /go/src/github.com/Azure/acs-engine
RUN git clone -b $ACSENGINE_VERSION https://github.com/karataliu/acs-engine . \
    && make

FROM microsoft/dotnet:2.0-sdk-jessie AS build_kautil
WORKDIR /build
COPY kautil .
RUN dotnet publish -c release -r debian.8-x64 -o publish \
    && chmod 755 publish/kautil

FROM buildpack-deps:jessie-scm
# libunwind8: used by dotnet core, libxml-simple-perl, used by junit-combine
RUN apt update && apt install -y --no-install-recommends libunwind8 libxml-simple-perl
ENV GOPATH="/home/k8s-ci/go"
COPY --from=build_kubernetes /go ${GOPATH}
COPY --from=build_acs-engine /go/src/github.com/Azure/acs-engine/bin/acs-engine /usr/local/bin/
COPY --from=build_kautil /build/publish /opt/k8s-azure/kautil
COPY k8s-azure junit-combine /opt/k8s-azure/
COPY manifest /opt/k8s-azure/manifest
RUN ["/bin/bash", "-c", "ln -s /opt/k8s-azure/{k8s-azure,junit-combine,kautil/kautil} /usr/local/bin"]
ENV K8S_AZURE_MANIFEST_DIR='/opt/k8s-azure/manifest'
RUN useradd -m k8s-ci
