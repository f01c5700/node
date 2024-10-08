FROM golang:1.21 AS op

WORKDIR /app

ENV REPO=https://github.com/ethereum-optimism/optimism.git
ENV VERSION=v1.9.1
ENV COMMIT=4797ddb70e05d4952685bad53e608cb5606284e6
RUN git clone $REPO --branch op-node/$VERSION --single-branch . && \
    git switch -c branch-$VERSION && \
    bash -c '[ "$(git rev-parse HEAD)" = "$COMMIT" ]'

RUN cd op-node && \
    make VERSION=$VERSION op-node

FROM golang:1.21 AS geth

WORKDIR /app

ENV REPO=https://github.com/f01c5700/op-geth
ENV VERSION=cv1.0.3
ENV COMMIT=9c2667bf4c3d5b4a00380b359eff47c148018e0c
RUN git clone $REPO --branch $VERSION --single-branch . && \
    git switch -c branch-$VERSION && \
    bash -c '[ "$(git rev-parse HEAD)" = "$COMMIT" ]'

RUN go run build/ci.go install -static ./cmd/geth

FROM ubuntu:22.04

RUN apt-get update && \
    apt-get install -y jq curl supervisor && \
    rm -rf /var/lib/apt/lists
RUN mkdir -p /var/log/supervisor

WORKDIR /app

COPY --from=op /app/op-node/bin/op-node ./
COPY --from=geth /app/build/bin/geth ./
COPY supervisord.conf /etc/supervisor/conf.d/supervisord.conf
COPY geth/geth-entrypoint ./execution-entrypoint
COPY op-node-entrypoint .

CMD ["/usr/bin/supervisord"]
