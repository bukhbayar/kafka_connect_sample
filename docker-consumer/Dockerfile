FROM confluentinc/cp-kafka-connect-base:7.2.0

ENV LB_LIBRARY_PATH=/usr/share/java/instant_client/

RUN confluent-hub install --no-prompt confluentinc/kafka-connect-jdbc:10.5.2

COPY oracle_instantclient/* /usr/share/java/instant_client/
COPY oracle_instantclient/ojdbc8.jar /usr/share/confluent-hub-components/confluentinc-kafka-connect-jdbc/lib

curl "https://oss.sonatype.org/service/local/repositories/snapshots/content/io/debezium/debezium-connector-oracle/1.2.0-SNAPSHOT/debezium-connector-oracle-1.2.0-20200612.041347-34-plugin.tar.gz" | \
    tar xvfz - --directory /usr/share/java/

USER root

RUN yum -y update

RUN yum -y install libaio