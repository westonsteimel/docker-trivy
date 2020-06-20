ARG TRIVY_VERSION="master"

FROM golang:alpine AS builder

RUN	apk upgrade && apk --no-cache add \
	ca-certificates \
    git \
    upx \
    binutils

ARG TRIVY_VERSION
ENV TRIVY_VERSION "${TRIVY_VERSION}"

WORKDIR /src/

RUN git clone --depth 1 --branch "${TRIVY_VERSION}" https://github.com/aquasecurity/trivy.git

WORKDIR /src/trivy

RUN go mod download
RUN CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build -ldflags "-s -w -X main.version=${TRIVY_VERSION}" -a -o /trivy ./cmd/trivy/
RUN strip /trivy \
    && upx --lzma --best /trivy

RUN addgroup trivy \
    && adduser -G trivy -s /bin/sh -D trivy

FROM alpine:edge
ARG TRIVY_VERSION

RUN apk upgrade && apk --no-cache add git rpm

COPY --from=builder /etc/passwd /etc/passwd
COPY --from=builder /etc/ssl/certs /etc/ssl/certs
COPY --from=builder /trivy /usr/local/bin/trivy

USER trivy

WORKDIR /home/trivy/

ENTRYPOINT ["/usr/local/bin/trivy"]

LABEL org.opencontainers.image.title="trivy" \
    org.opencontainers.image.description="trivy in Docker" \ 
    org.opencontainers.image.url="https://github.com/westonsteimel/docker-trivy" \ 
    org.opencontainers.image.source="https://github.com/westonsteimel/docker-trivy" \
    org.opencontainers.image.version="${TRIVY_VERSION}" \
    version="${TRIVY_VERSION}"
