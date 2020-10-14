# Building on Docker buildx
# docker buildx build --platform linux/arm64,linux/amd64,linux/ppc64le -t repo/keycloak-gatekeeper:v9.0.0 . --push
# docker buildx build --platform linux/arm64,linux/amd64,linux/ppc64le --build-arg VERSION=8.0.0 -t repo/keycloak-gatekeeper:v8.0.0 . --push
# docker buildx build --platform linux/arm64,linux/amd64,linux/ppc64le --build-arg VERSION=master -t repo/keycloak-gatekeeper:latest . --push

# Build on Docker
# For the default os/arch (linux/amd64) and version (9.0.0)
# docker build -t repo/keycloak-gatekeeper:v9.0.0 .

# Builder container
FROM golang:1.14 as builder

ARG TARGETOS
ARG TARGETARCH
ARG VERSION=9.0.0

WORKDIR $GOPATH/src/app

RUN git clone https://github.com/keycloak/keycloak-gatekeeper.git -b $VERSION --depth=1 && \
    cd keycloak-gatekeeper && \
    GOOS=$TARGETOS GOARCH=$TARGETARCH make && \
    mv bin/keycloak-gatekeeper /keycloak-gatekeeper

# Application Container
FROM registry.access.redhat.com/ubi8-minimal

ARG TARGETOS
ARG TARGETARCH
ARG VERSION=9.0.0
ENV NAME keycloak-gatekeeper
ENV KEYCLOAK_VERSION $VERSION
ENV GOOS $TARGETOS
ENV GOARCH $TARGETARCH

LABEL Name=keycloak-gatekeeper \
    Release=https://github.com/keycloak/keycloak-gatekeeper \
    Url=https://github.com/keycloak/keycloak-gatekeeper \
    Help=https://issues.jboss.org/projects/KEYCLOAK

USER root

WORKDIR "/opt/gatekeeper"

RUN echo "gatekeeper:x:1000:gatekeeper" >> /etc/group
RUN echo "gatekeeper:x:1000:1000:gatekeeper user:/opt/gatekeeper:/sbin/nologin" >> /etc/passwd
RUN chown -R gatekeeper:gatekeeper /opt/gatekeeper && chmod -R g+rw /opt/gatekeeper

RUN microdnf update -y && microdnf install -y ca-certificates

COPY --from=builder /keycloak-gatekeeper /opt/gatekeeper/keycloak-gatekeeper

RUN microdnf clean all

USER 1000

ENTRYPOINT [ "/opt/gatekeeper/keycloak-gatekeeper" ]
