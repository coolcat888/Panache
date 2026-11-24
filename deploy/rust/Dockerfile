# Railway 部署用改写版 Dockerfile（基于上游 deploy/rust/Dockerfile）。
# 改动：上游要求预先在本地构建 app/build/web；本版在镜像内下载官方
# Flutter 3.47.5 SDK 并构建 Flutter Web，再构建 Rust 服务。构建上下文为仓库根目录。
FROM debian:bookworm-slim AS web-build
ARG FLUTTER_VERSION=3.47.5
ENV PATH=/opt/flutter/bin:${PATH}
WORKDIR /source
RUN apt-get update && apt-get install -y --no-install-recommends \
        curl git xz-utils unzip python3 ca-certificates \
    && rm -rf /var/lib/apt/lists/* \
    && curl -fsSL -o /tmp/flutter.tar.xz \
        "https://storage.googleapis.com/flutter_infra_release/releases/stable/linux/flutter_linux_${FLUTTER_VERSION}-stable.tar.xz" \
    && mkdir -p /opt/flutter \
    && tar -xJf /tmp/flutter.tar.xz -C /opt \
    && rm /tmp/flutter.tar.xz \
    && git config --global --add safe.directory /opt/flutter
COPY app ./app
COPY assets ./assets
COPY vendor ./vendor
COPY scripts ./scripts
COPY VERSION ./VERSION
RUN python3 scripts/build-flutter-web.py

FROM rust:1.94.1-bookworm AS build
WORKDIR /source
COPY Cargo.toml Cargo.lock ./
COPY crates ./crates
COPY VERSION ./VERSION
RUN cargo build --locked --release -p handwriting-server

FROM debian:bookworm-slim
RUN useradd --uid 10001 --create-home handwriting && mkdir /data && chown handwriting /data
WORKDIR /opt/handwriting
COPY --from=build /source/target/release/handwriting-server ./
COPY --from=web-build /source/app/build/web ./web
COPY assets/font_assets ./font_assets
COPY VERSION ./VERSION
COPY LICENSE ./LICENSE
COPY crates/handwriting-core/NOTICE ./NOTICE
COPY crates/handwriting-core/licenses ./licenses
COPY assets/licenses ./licenses
USER handwriting
ENV HANDWRITING_LISTEN_ADDR=0.0.0.0:5006 HANDWRITING_DATA_DIR=/data
EXPOSE 5006
ENTRYPOINT ["/opt/handwriting/handwriting-server"]
