FROM postgres:11.6-alpine

COPY ./init-products-db.sh /docker-entrypoint-initdb.d/init-products-db.sh

RUN sed -i 's/\r$//' /docker-entrypoint-initdb.d/init-products-db.sh \
    && chmod +x /docker-entrypoint-initdb.d/init-products-db.sh