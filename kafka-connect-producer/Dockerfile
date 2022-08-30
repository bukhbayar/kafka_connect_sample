ARG DEBEZIUM_VERSION
FROM debezium/connect:$DEBEZIUM_VERSION
ENV INSTANT_CLIENT_DIR=/instant_client/

ARG KAFKA_JDBC_VERSION=10.2.0

USER root
RUN microdnf -y install libaio && microdnf clean all

USER kafka

# Deploy Oracle client and drivers
COPY oracle_instantclient/* $INSTANT_CLIENT_DIR
COPY oracle_instantclient/ojdbc8.jar /kafka/libs