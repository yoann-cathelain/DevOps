FROM postgres:14.1-alpine

ENV POSTGRES_DB=aled \
   POSTGRES_USER=aled \
   POSTGRES_PASSWORD=aled

COPY ./sql/createSchema.sql /docker-entrypoint-initdb.d/
COPY ./sql/insertData.sql /docker-entrypoint-initdb.d/