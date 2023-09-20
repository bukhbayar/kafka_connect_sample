ARG DEBEZIUM_VERSION
FROM quay.io/debezium/connect:$DEBEZIUM_VERSION
ENV KAFKA_CONNECT_JDBC_DIR=$KAFKA_CONNECT_PLUGINS_DIR/kafka-connect-jdbc \
    INSTANT_CLIENT_DIR=/instant_client/ \
    DEBEZIUM_CONNECTOR_ORACLE_DIR=$KAFKA_CONNECT_PLUGINS_DIR/debezium-connector-oracle

ARG KAFKA_JDBC_VERSION=10.2.0

# Deploy Kafka Connect JDBC
RUN mkdir $KAFKA_CONNECT_JDBC_DIR && cd $KAFKA_CONNECT_JDBC_DIR &&\
	curl -sO https://packages.confluent.io/maven/io/confluent/kafka-connect-jdbc/$KAFKA_JDBC_VERSION/kafka-connect-jdbc-$KAFKA_JDBC_VERSION.jar

RUN mkdir /kafka/connect/multiple-field-timestamp-converter/
COPY --chown=kafka common/multiple-field-timestamp-converter-1.0.0-jar-with-dependencies.jar /kafka/connect/multiple-field-timestamp-converter/multiple-field-timestamp-converter-1.0.0-jar-with-dependencies.jar

COPY --chown=kafka common/debezium-scripting-1.9.5.Final.jar $KAFKA_CONNECT_JDBC_DIR/debezium-scripting-1.9.5.Final.jar
COPY --chown=kafka common/groovy/ $KAFKA_CONNECT_JDBC_DIR/

USER root
RUN microdnf -y install libaio && microdnf clean all

USER kafka

# Deploy Oracle client and drivers
COPY --chown=kafka common/oracle_instantclient/* $INSTANT_CLIENT_DIR
COPY --chown=kafka common/oracle_instantclient/ojdbc8.jar /kafka/libs