FROM getmeili/meilisearch:v1.5 as meili

COPY --from=public.ecr.aws/awsguru/aws-lambda-adapter:0.7.0 /lambda-adapter /opt/extensions/lambda-adapter

FROM alpine:3.16

# for Lambda Web Adapter 
ENV MEILI_HTTP_ADDR 0.0.0.0:8080 
ENV MEILI_SERVER_PROVIDER docker
ENV MEILI_DB_PATH ./
ENV MEILI_MASTER_KEY meilisearch-master-key
ENV MEILI_ENV production

RUN apk update --quiet \
    && apk add -q --no-cache libgcc tini curl

COPY --from=meili /bin/meilisearch /bin/meilisearch 
COPY --from=meili /bin/meilitool /bin/meilitool

# To stay compatible with the older version of the container (pre v0.27.0) we're
# going to symlink the meilisearch binary in the path to `/meilisearch`
RUN ln -s /bin/meilisearch /meilisearch

COPY ./meili_data /mnt/meili_data

WORKDIR /mnt/meili_data

EXPOSE 8080/tcp

ENTRYPOINT ["tini", "--"]
CMD /bin/meilisearch