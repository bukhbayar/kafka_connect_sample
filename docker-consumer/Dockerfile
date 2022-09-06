FROM confluentinc/cp-kafka-connect-base:7.2.0

ENV LB_LIBRARY_PATH=/instant_client/

RUN confluent-hub install --no-prompt confluentinc/kafka-connect-jdbc:10.5.2

COPY ../common/oracle_instantclient/* /instant_client/
COPY ../common/oracle_instantclient/ojdbc8.jar /kafka/libs

USER root

RUN yum -y update

RUN yum -y install libaio