ARG PANDEPTH_VERSION=2.26
ARG PANDEPTH_TAG_COMMIT=afc2e5f10c60384dc5574846fadf8dba67476b44
ARG DEBIAN_MIRROR=http://deb.debian.org/debian
ARG DEBIAN_SECURITY_MIRROR=http://deb.debian.org/debian-security

FROM debian:12-slim AS builder

ARG PANDEPTH_VERSION
ARG PANDEPTH_TAG_COMMIT
ARG DEBIAN_MIRROR
ARG DEBIAN_SECURITY_MIRROR
ENV DEBIAN_FRONTEND=noninteractive

RUN set -eux; \
    sed -i \
      -e "s|http://deb.debian.org/debian-security|${DEBIAN_SECURITY_MIRROR}|g" \
      -e "s|http://deb.debian.org/debian|${DEBIAN_MIRROR}|g" \
      /etc/apt/sources.list.d/debian.sources; \
    for attempt in 1 2 3 4 5 6; do \
      if apt-get -o Acquire::Retries=3 update \
       && apt-get -o Acquire::Retries=3 install -y --no-install-recommends \
            ca-certificates \
            g++ \
            git \
            libdeflate-dev \
            libhts-dev \
            zlib1g-dev; then \
        break; \
      fi; \
      if [ "$attempt" = "6" ]; then exit 1; fi; \
      apt-get clean; \
      rm -rf /var/lib/apt/lists/*; \
      sleep 5; \
    done; \
    rm -rf /var/lib/apt/lists/*

WORKDIR /tmp/build

RUN set -eux; \
    git clone --depth 1 --branch "v${PANDEPTH_VERSION}" https://github.com/HuiyangYu/PanDepth.git pandepth; \
    cd pandepth; \
    test "$(git rev-parse HEAD)" = "${PANDEPTH_TAG_COMMIT}"; \
    sed -i \
      -e 's|#include "./zlib.h"|#include <zlib.h>|g' \
      -e 's|#include  "./zlib.h"|#include <zlib.h>|g' \
      -e 's|#include "./sam.h"|#include <htslib/sam.h>|g' \
      -e 's|#include "./hts.h"|#include <htslib/hts.h>|g' \
      -e 's|#include "./cram.h"|#include <htslib/cram.h>|g' \
      src/PanDepth.cpp include/comm.h; \
    mkdir -p \
      /opt/pandepth/bin \
      /opt/pandepth/share/doc/pandepth \
      /opt/pandepth/share/licenses/pandepth; \
    g++ --std=c++11 -O3 \
      src/PanDepth.cpp \
      -Iinclude \
      -lhts \
      -ldeflate \
      -lz \
      -pthread \
      -o /opt/pandepth/bin/pandepth; \
    install -m 0644 README.md /opt/pandepth/share/doc/pandepth/README.md; \
    install -m 0644 LICENSE /opt/pandepth/share/licenses/pandepth/LICENSE; \
    /opt/pandepth/bin/pandepth -h 2>&1 | grep -F "show this help [v${PANDEPTH_VERSION}]" >/dev/null

FROM debian:12-slim

LABEL maintainer="taffish@163.com"

ARG PANDEPTH_VERSION
ARG DEBIAN_MIRROR
ARG DEBIAN_SECURITY_MIRROR
ENV DEBIAN_FRONTEND=noninteractive

COPY --from=builder /opt/pandepth /opt/pandepth

RUN set -eux; \
    sed -i \
      -e "s|http://deb.debian.org/debian-security|${DEBIAN_SECURITY_MIRROR}|g" \
      -e "s|http://deb.debian.org/debian|${DEBIAN_MIRROR}|g" \
      /etc/apt/sources.list.d/debian.sources; \
    for attempt in 1 2 3 4 5 6; do \
      if apt-get -o Acquire::Retries=3 update \
       && apt-get -o Acquire::Retries=3 install -y --no-install-recommends \
            ca-certificates \
            gzip \
            libdeflate0 \
            libhts3 \
            libstdc++6 \
            zlib1g; then \
        break; \
      fi; \
      if [ "$attempt" = "6" ]; then exit 1; fi; \
      apt-get clean; \
      rm -rf /var/lib/apt/lists/*; \
      sleep 5; \
    done; \
    rm -rf /var/lib/apt/lists/*

RUN set -eux; \
    ln -sf /opt/pandepth/bin/pandepth /usr/local/bin/pandepth

ENV PATH=/opt/pandepth/bin:${PATH}

RUN set -eux; \
    command -v pandepth >/dev/null; \
    command -v gzip >/dev/null; \
    command -v ldd >/dev/null; \
    pandepth -h 2>&1 | grep -F "show this help [v${PANDEPTH_VERSION}]" >/dev/null; \
    ldd "$(command -v pandepth)" | grep -F "libhts.so.3" >/dev/null; \
    ldd "$(command -v pandepth)" | grep -F "libdeflate.so.0" >/dev/null; \
    mkdir -p /tmp/pandepth-check; \
    cd /tmp/pandepth-check; \
    printf '@HD\tVN:1.6\tSO:unsorted\n@SQ\tSN:chr1\tLN:100\nr1\t0\tchr1\t11\t60\t10M\t*\t0\t0\tACGTACGTAA\tFFFFFFFFFF\n' > toy.sam; \
    pandepth -i toy.sam -o toy -t 1 -a >/tmp/pandepth-check.log 2>&1; \
    test -s toy.chr.stat.gz; \
    test -s toy.SiteDepth.gz; \
    gzip -cd toy.chr.stat.gz | grep -F "chr1" >/dev/null; \
    gzip -cd toy.chr.stat.gz | grep -F "10.00" >/dev/null; \
    printf 'chr1\t10\t20\n' > toy.bed; \
    pandepth -i toy.sam -b toy.bed -o bed -t 1 >/tmp/pandepth-check-bed.log 2>&1; \
    test -s bed.bed.stat.gz; \
    gzip -cd bed.bed.stat.gz | grep -F "chr1_10_20" >/dev/null; \
    rm -rf /tmp/pandepth-check /tmp/pandepth-check.log /tmp/pandepth-check-bed.log

WORKDIR /root

ENV TAFFISH_ENV=TAFFISH
ENV TAFFISH_NAME=pandepth
