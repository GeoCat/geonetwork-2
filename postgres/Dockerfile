FROM geonetwork:2.10.4

RUN apt-get update && apt-get install -y postgresql-client

COPY ./config.xml $BASE_DIR/webapps/geonetwork/WEB-INF/config.xml

#Initializing database & connection string for GN
COPY ./docker-entrypoint.sh /entrypoint.sh
ENTRYPOINT ["/entrypoint.sh"]
