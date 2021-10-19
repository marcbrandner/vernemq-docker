# =============================================================================
# ARGUMENTS USED BY BOTH STAGES
# =============================================================================

ARG ERLANG_VERSION="23.3.2.0"
ARG ALPINE_VERSION="3.13.6"
ARG VMQ_VERSION="1.12.3"

# =============================================================================
# BUILD STAGE
# =============================================================================

FROM docker.io/erlang:${ERLANG_VERSION}-alpine as build

ARG VMQ_VERSION

WORKDIR /vernemq

RUN apk add --update \
        build-base \
        bsd-compat-headers \
        git \
        openssl-dev \
        snappy-dev \
        curl \
    && rm -rf /var/cache/apk/*

RUN git clone -b ${VMQ_VERSION} --single-branch https://github.com/vernemq/vernemq.git . \
    && make rel

# =============================================================================
# RUN STAGE
# =============================================================================

ARG ALPINE_VERSION

FROM docker.io/alpine:${ALPINE_VERSION}

ARG VMQ_VERSION

LABEL org.opencontainers.image.source https://github.com/marcbrandner/vernemq-docker

RUN apk --no-cache --update --available upgrade && \
    apk add --no-cache ncurses-libs openssl libstdc++ jq curl bash snappy-dev nano && \
    addgroup --gid 10000 vernemq && \
    adduser --uid 10000 -H -D -G vernemq -h /vernemq vernemq && \
    install -d -o vernemq -g vernemq /vernemq

ENV DOCKER_VERNEMQ_KUBERNETES_LABEL_SELECTOR="app=vernemq" \
    DOCKER_VERNEMQ_LOG__CONSOLE=console \
    PATH="/vernemq/bin:$PATH" \
    VERNEMQ_VERSION=${VMQ_VERSION}

WORKDIR /vernemq

COPY --from=build --chown=10000:10000 /vernemq/_build/default/rel/vernemq ./

RUN wget https://raw.githubusercontent.com/vernemq/docker-vernemq/${VMQ_VERSION}/bin/vernemq.sh \
    && mv vernemq.sh /usr/sbin/start_vernemq \
    && chown 10000:10000 /usr/sbin/start_vernemq \
    && chmod +x /usr/sbin/start_vernemq

RUN wget https://raw.githubusercontent.com/vernemq/docker-vernemq/${VMQ_VERSION}/files/vm.args \
    && mv vm.args /vernemq/etc/vm.args \
    && chown 10000:10000 /vernemq/etc/vm.args

RUN ln -s /vernemq/etc /etc/vernemq && \
    ln -s /vernemq/data /var/lib/vernemq && \
    ln -s /vernemq/log /var/log/vernemq

# Ports
# 1883  MQTT
# 8883  MQTT/SSL
# 8080  MQTT WebSockets
# 44053 VerneMQ Message Distribution
# 4369  EPMD - Erlang Port Mapper Daemon
# 8888  Prometheus Metrics
# 9100 9101 9102 9103 9104 9105 9106 9107 9108 9109  Specific Distributed Erlang Port Range

EXPOSE 1883 8883 8080 44053 4369 8888 \
       9100 9101 9102 9103 9104 9105 9106 9107 9108 9109

VOLUME ["/vernemq/log", "/vernemq/data", "/vernemq/etc"]

HEALTHCHECK CMD vernemq ping | grep -q pong

USER vernemq
CMD ["start_vernemq"]