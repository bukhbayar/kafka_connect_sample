# Using Debezium From Oracle To Oracle
Start the topology as defined in <https://debezium.io/docs/tutorial/>
#


## Step 1: Prepare
Download the [Oracle instant client for Linux](http://www.oracle.com/technetwork/topics/linuxx86-64soft-092277.html)
and put it under the each directories:
- docker-consumer/oracle_instantclient/
- docker-producer/oracle_instantclient/


## Step 2: Starting the services
```shell
./build-and-run.sh
```

## Step 3: Create SOURCE kafka connector for CUSTOMER table
```shell
# Create kafka source connector
curl -i -X POST -H "Accept:application/json" -H  "Content-Type:application/json" http://localhost:8083/connectors/ -d @register-oracle-source.json

# Check status if source kafka connector is created
curl -i -X GET  http://localhost:8083/connectors/inventory-source-connector/status
```

## Step 4: Create SINK kafka connector for CUSTOMER table.
```shell
# Create sink kafka connector
curl -i -X POST -H "Accept:application/json" -H  "Content-Type:application/json" http://localhost:8083/connectors/ -d @register-oracle-sink-customers.json

# Check status if sink kafka connector is created
curl -i -X GET  http://localhost:8083/connectors/jdbc-sink-customers/status
```

## Step 5: Login to SOURCE oracle database

```shell
# remote ssh login to source database
docker exec -it oracle-db-source /bin/bash

# login to source database
sqlplus sys/oracle as sysdba
```

## Step 6: Starting testing by updating CUSTOMERS table in source database
```sql
-- check existing tables before make any changes
SELECT FIRST_NAME FROM INVENTORY.CUSTOMERS c ;

UPDATE INVENTORY.CUSTOMERS c SET c.FIRST_NAME = 'Streaming';

-- this will trigger streaming to target database
COMMIT;
```

## Step 7: Login to TARGET oracle database

```shell
# remote ssh login to target database
docker exec -it oracle-db-target /bin/bash

# login to target database
sqlplus sys/oracle as sysdba
```

## Step 8: Starting testing by checking CUSTOMERS table in target database
```sql
-- Check if databases are created in target database
SELECT * FROM ALL_TABLES at2 WHERE OWNER = 'INVENTORY';

-- See if the streaming data updated here
SELECT FIRST_NAME FROM INVENTORY.CUSTOMERS c;
```

## Step 9: Done & stop the topology

```shell
./build-and-down.sh
```

## Docker Compose environments
- Kafka
- Zookeeper
- Source Kafka connector Connector
- Sink Kafka Connector
- Connect to Source Oracle DB
  - Host: oracle-db-source
  - Port: 1521
  - Service Name: XE
  - user: SYS
  - pass: oracle

- Connect to Target Oracle DB
  - Host: oracle-db-target
  - Port: 3042
  - Service Name: XE
  - user: SYS
  - pass: oracle

#
# Dig deeper if you want
- Update SOURCE kafka connector if you updated `update-source-oracle.json` file.
```shell
# Create source connector
curl -i -X PUT -H "Accept:application/json" -H  "Content-Type:application/json" http://localhost:8083/connectors/inventory-source-connector/config/ -d @update-oracle-source.json

# check if source connector created
curl -i -X GET  http://localhost:8083/connectors/inventory-source-connector/status
```

- Update SINK/TARGET kafka connector if you updated `update-oracle-sink-customers.json` file.
```shell
curl -i -X PUT -H "Accept:application/json" -H  "Content-Type:application/json" http://localhost:8083/connectors/jdbc-sink-customers/config/ -d @update-oracle-sink-customers.json
```

- See existing kafka topics via CLI
```shell
# no need remote ssh to kafka container
docker exec -it kafka /kafka/bin/kafka-topics.sh --bootstrap-server kafka:9092 --list
```

- Inpsect an existing kafka topic via CLI

```shell
# no need remote ssh to kafka container
docker exec -it consumer /kafka/bin/kafka-console-consumer.sh \
    --bootstrap-server kafka:9092 \
    --from-beginning \
    --topic oracle-db-source.INVENTORY.CUSTOMERS \
    --property print.key=true \
    --property schema.registry.url=http://registry:8081 \
    --property key.deserializer=org.apache.kafka.common.serialization.StringDeserializer \
    --property print.key=true \
    --property key.separator="-"
```

- See the connectors

```shell
curl -i -X GET  http://localhost:8083/connectors
```

## Manage Connectors
- See the connector status

```shell
curl -s "http://localhost:8083/connectors?expand=info&expand=status"
```

```shell
    curl -s "http://localhost:8083/connectors?expand=info&expand=status" | \
    jq '. | to_entries[] | [ .value.info.type, .key, .value.status.connector.state,.value.status.tasks[].state,.value.info.config."connector.class"]|join(":|:")' | \
    column -s : -t| sed 's/\"//g'| sort
```

- Status of connector

```shell
curl -i -X GET  http://localhost:8083/connectors/inventory-source-connector/status
#OR
curl -i -X GET  http://localhost:8083/connectors/jdbc-sink-customers/status
```

- Restart a connector

```shell
curl -i -X POST  http://localhost:8083/connectors/inventory-source-connector/restart
#OR
curl -i -X POST  http://localhost:8083/connectors/jdbc-sink-customers/restart
```

- Remove a connector

```shell
curl -i -X DELETE  http://localhost:8083/connectors/inventory-source-connector
#OR
curl -i -X DELETE  http://localhost:8083/connectors/jdbc-sink-customers
```


# DockerHub
```shell
# For example, pushing producer image into docker repository (public)
cd docker-producer
docker build --build-arg DEBEZIUM_VERSION=1.9.5.Final -t <repositoryName>/<producer>:v1 .
docker push <repositoryName>/<producer>:v1 .
```