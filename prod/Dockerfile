FROM gcr.io/distroless/cc-debian12:latest AS cc
FROM alpine:3.16

# Lambda Web Adapter
COPY --from=public.ecr.aws/awsguru/aws-lambda-adapter:0.7.0 /lambda-adapter /opt/extensions/lambda-adapter

# for execute meilisearch
ENV LD_LIBRARY_PATH="/usr/local/lib"
COPY --from=cc --chown=root:root --chmod=755 /lib/*-linux-gnu/* /usr/local/lib/

RUN apk update --quiet \
 && apk add -q --no-cache libgcc tini curl \
 && mkdir /lib64 \ 
 && ln -s /usr/local/lib/ld-linux-*.so.2 /lib64/

# Install Meilisearch
# any version
ARG version="v1.5.1"
ARG os="meilisearch-linux-amd64"
ARG url="https://github.com/meilisearch/meilisearch/releases/download"
RUN curl -OL ${url}/${version}/${os} | sh \
 && mv ${os} meilisearch \
 && chmod +x meilisearch \
 && mv meilisearch /usr/bin/

ENV MEILI_HTTP_ADDR 0.0.0.0:8080
ENV MEILI_SERVER_PROVIDER docker
ENV MEILI_DB_PATH <like /mnt/foo/data.ms>
ENV MEILI_MASTER_KEY <any meilisearch-master-key>
ENV MEILI_ENV production

COPY ./meili_data /mnt/meili_data

WORKDIR /mnt/meili_data

EXPOSE 8080/tcp

ENTRYPOINT ["tini", "--"]
CMD /usr/bin/meilisearch
#CMD ["sleep", "infinity"]