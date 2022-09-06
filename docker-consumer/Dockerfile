FROM confluentinc/cp-kafka-connect-base:7.2.0

ENV LB_LIBRARY_PATH=/instant_client/

RUN confluent-hub install --no-prompt confluentinc/kafka-connect-jdbc:10.5.2

COPY common/oracle_instantclient/* /usr/share/java/debezium-connector-oracle/
COPY common/oracle_instantclient/ojdbc8.jar /usr/share/confluent-hub-components/confluentinc-kafka-connect-jdbc/lib

USER root

RUN yum -y update

RUN yum -y install libaio