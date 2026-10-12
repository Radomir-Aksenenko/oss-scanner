FROM golang:1.26-bookworm

ENV DEBIAN_FRONTEND=noninteractive \
    CGO_ENABLED=1

RUN apt-get update && apt-get install -y --no-install-recommends \
      build-essential ca-certificates git iproute2 pkg-config python3 \
    && rm -rf /var/lib/apt/lists/*

# The scanner supplies the checkout as the build context.
COPY . /src
WORKDIR /src

# Fetch all modules now: the audit runs without network access.
RUN go mod download

RUN go build -o /usr/local/bin/obsidian-server ./cmd/server \
    && go build -o /usr/local/bin/obsidian-client ./cmd/client \
    && go build -o /usr/local/bin/obsidian-probe ./cmd/probe

# Keep tests available even if a baseline test fails.
RUN go test -short -timeout 15m ./obsidian/... ./cmd/... ./pkg/... \
    || echo 'WARNING: baseline tests failed'

# New reproducers and patches must also build with the network disabled.
ENV GOPROXY=off \
    GOSUMDB=off
