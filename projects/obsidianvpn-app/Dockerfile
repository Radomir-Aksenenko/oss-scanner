FROM debian:bookworm

ENV DEBIAN_FRONTEND=noninteractive \
    FLUTTER_ROOT=/opt/flutter \
    PUB_CACHE=/opt/pub-cache \
    PATH=/opt/flutter/bin:/usr/local/go/bin:$PATH

RUN apt-get update && apt-get install -y --no-install-recommends \
      ca-certificates curl git unzip xz-utils python3 build-essential \
    && rm -rf /var/lib/apt/lists/*

# Flutter, same version as the project's CI.
RUN git clone --depth 1 --branch 3.44.8 https://github.com/flutter/flutter.git /opt/flutter \
    && git config --global --add safe.directory /opt/flutter \
    && flutter --version

# The scanner supplies the checkout as the build context.
COPY . /src
WORKDIR /src

# The Go core is a git submodule (core/); fetch it explicitly in case the checkout has none.
RUN if [ ! -f core/go.mod ]; then \
      rm -rf core && git clone --depth 1 https://github.com/Radomir-Aksenenko/ObsidianVPN.git core; \
    fi

# Fetch Dart dependencies now: the audit runs without network access.
RUN cd app && flutter pub get

# Keep tests available even if a baseline test fails.
RUN cd app && (flutter analyze || echo 'WARNING: flutter analyze reported issues') \
    && (flutter test || echo 'WARNING: baseline tests failed')
