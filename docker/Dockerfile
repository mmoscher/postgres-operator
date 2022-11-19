ARG ALPINE_VERSION=3.15
ARG BUILDPLATFORM=linux/amd64
FROM --platform=$BUILDPLATFORM ${DOCKER_REGISTRY:+$DOCKER_REGISTRY/}golang:1.17-alpine${ALPINE_VERSION} as go-builder

ARG TARGETOS
ARG TARGETARCH

ENV GO111MODULE=on

WORKDIR /src
COPY . .

RUN go get -d k8s.io/client-go@kubernetes-1.22.4
RUN go install github.com/golang/mock/mockgen@v1.6.0
RUN go mod tidy
RUN go mod vendor

RUN GOOS=${TARGETOS} GOARCH=${TARGETARCH} CGO_ENABLED=0 go build -o build/linux/postgres-operator -v -ldflags "-X=main.version=modac" cmd/main.go

FROM alpine:3.15 AS postgres-operator
LABEL maintainer="Team ACID @ Zalando <team-acid@zalando.de>"

# We need root certificates to deal with teams api over https
RUN apk --no-cache add curl
RUN apk --no-cache add ca-certificates

COPY --from=go-builder /src/build/linux/postgres-operator /

RUN addgroup -g 1000 pgo
RUN adduser -D -u 1000 -G pgo -g 'Postgres Operator' pgo

USER 1000:1000

ENTRYPOINT ["/postgres-operator"]
