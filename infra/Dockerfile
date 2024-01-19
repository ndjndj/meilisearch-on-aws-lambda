FROM gcr.io/distroless/cc-debian12:latest AS cc
FROM alpine:3.16

COPY --from=public.ecr.aws/awsguru/aws-lambda-adapter:0.7.0 /lambda-adapter /opt/extensions/lambda-adapter

# for execute meilisearch
ENV LD_LIBRARY_PATH="/usr/local/lib"
COPY --from=cc --chown=root:root --chmod=755 /lib/*-linux-gnu/* /usr/local/lib/

RUN apk update --quiet \
 && apk add -q --no-cache libgcc tini curl \
 && mkdir /lib64 \ 
 && ln -s /usr/local/lib/ld-linux-*.so.2 /lib64/

# Install Meilisearch
ARG version="v1.6.0"
ARG os="meilisearch-linux-amd64"
ARG url="https://github.com/meilisearch/meilisearch/releases/download"
RUN curl -OL ${url}/${version}/${os} | sh \
 && mv ${os} meilisearch \
 && chmod +x meilisearch \
 && mv meilisearch /usr/bin/

ENV MEILI_HTTP_ADDR 0.0.0.0:8080
ENV MEILI_SERVER_PROVIDER docker
ENV MEILI_DB_PATH /mnt/meili_data/data.ms
ENV MEILI_MASTER_KEY meilisearch-master-key 
ENV MEILI_ENV production

COPY ./meili_data /mnt/meili_data

# This directory should hold all the data related to meilisearch so we're going
# to move our PWD in there.
# We don't want to put the meilisearch binary
WORKDIR /mnt/meili_data

EXPOSE 8080/tcp

ENTRYPOINT ["tini", "--"]
CMD /usr/bin/meilisearch