# Kafka Commands

To subscribe to a topic that is avro format and integrated to the schema registry:

```bash
confluent kafka topic consume updateEnquirySendAnswerResponse --value-format avro --print-key --delimiter ":" --sr-endpoint "https://psrc-5mn3g.ap-southeast-2.aws.confluent.cloud" --sr-api-key NPUSPNJQWEGJVEPZ --sr-api-secret kkptVNihVsmmwBj8qRiOiMdWkFq+fvRUapKS+0A0GH16/FJxOZMjNex8XSB/qq0P

```


```bash
kafka-avro-console-consumer --topic updateEnquirySendAnswerResponse --bootstrap-server pkc-ldvj1.ap-southeast-2.aws.confluent.cloud:9092 --consumer.config config.properties --property schema.registry.url=https://psrc-5mn3g.ap-southeast-2.aws.confluent.cloud --property basic.auth.credentials.source=USER_INFO --property schema.registry.basic.auth.user.info='QJK7UM2S7VE5XKTB:ppuU9n5N2rU8bYv4arTCan8wK8nzRDHUpY+7GdLw0Djz6dm0jzEX741cCrbJF/h8'
```

Simplest way is to configure

```bash
confluent kafka topic consume updateEnquirySendAnswerResponse --value-format avro --print-key --delimiter ":"
Starting Kafka Consumer. Use Ctrl-C to exit.
```


config.properties

```conf
bootstrap.servers=pkc-ldvj1.ap-southeast-2.aws.confluent.cloud:9092
ssl.endpoint.identification.algorithm=https
security.protocol=SASL_SSL
sasl.mechanism=PLAIN
sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required username="OC22JMIFGM2WIEWM" password="1o999qhFWJoms71oq21bwyCkxQq+NJsUUKEryX68GuwXTnm4dRM4W+koEp2xfaaQ";
```


~/.confluent/java.config

```conf
# Required connection configs for Kafka producer, consumer, and admin
bootstrap.servers=pkc-ldvj1.ap-southeast-2.aws.confluent.cloud:9092
security.protocol=SASL_SSL
sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required username='OC22JMIFGM2WIEWM' password='1o999qhFWJoms71oq21bwyCkxQq+NJsUUKEryX68GuwXTnm4dRM4W+koEp2xfaaQ';
sasl.mechanism=PLAIN
# Required for correctness in Apache Kafka clients prior to 2.6
client.dns.lookup=use_all_dns_ips

# Best practice for higher availability in Apache Kafka clients prior to 3.0
session.timeout.ms=45000

# Best practice for Kafka producer to prevent data loss 
acks=all

# Required connection configs for Confluent Cloud Schema Registry
schema.registry.url=https://pksqlc-w19ww.ap-southeast-2.aws.confluent.cloud
basic.auth.credentials.source=USER_INFO
basic.auth.user.info=QJK7UM2S7VE5XKTB:ppuU9n5N2rU8bYv4arTCan8wK8nzRDHUpY+7GdLw0Djz6dm0jzEX741cCrbJF/h8

```

Consume the error topic of the HTTP connector


```bash

confluent kafka topic consume success-lcc-o328o9

confluent kafka topic consume success-lcc-o328o9 --value-format json --print-key --delimiter ":"


confluent kafka topic consume error-lcc-o328o9 --value-format json --print-key --delimiter ":"


```


Launch REST API proxy

```bash

kafka-rest-start ccloud-kafka-rest.properties

```

Print the entire schema graph (new feature in 2.13.2)

```bash

confluent schema-registry describe

```


Important change history

```
  - Add ``--config``, ``--config-file``, and ``--schema-id`` flags to ``confluent kafka topic produce`` and ``confluent kafka topic consume``
```


Temporarily set toipc retention time to 1 second so it removes all events from the topic:

```bash

kafka-configs --zookeeper localhost:2181 --alter --entity-type topics --entity-name "dev.etl.datagen.orders.src.0010" --add-config retention.ms=1000

kafka-configs --zookeeper localhost:2181 --alter --entity-type topics --entity-name "dev.etl.datagen.orders.src.0010" --add-config retention.ms=604800000

```


Post to a topic that doesn't have a schema, basic text

```bash
 kafka % kafka-console-producer --broker-list pkc-4n66v.australiaeast.azure.confluent.cloud:9092 --producer.config config.properties --topic test-topic
```