# Multi Schema Topic


Create the stream that is backed by the topic with multile schemas. Remember to use full quotes around a stream name that has a hyphen. 

```bash
CREATE STREAM "all-types-stream" (adviceId VARCHAR KEY) WITH (kafka_topic = 'all-types', key_format = 'AVRO', value_format = 'AVRO', partitions = 6);
```

The config.properties file

```bash
bootstrap.servers=pkc-4n66v.australiaeast.azure.confluent.cloud:9092
ssl.endpoint.identification.algorithm=https
security.protocol=SASL_SSL
sasl.mechanism=PLAIN
sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required username="EJKFP6J7YE3QBMTH" password="rpZgYZOjm9R0UEK9l2e7SKwGPDnT3YQDqKULNIDuyS7nc28AYcGEOHRXF0ZePZO3";
```

Produce an avro message with the key


```bash
kafka-avro-console-producer --broker-list pkc-4n66v.australiaeast.azure.confluent.cloud:9092 --producer.config config.properties --topic all-types --property parse.key=true --property key.schema.id=100039 --bootstrap-server pkc-4n66v.australiaeast.azure.confluent.cloud:9092 --property auto.register=false --property use.latest.version=true  --property value.schema.id=100037 --property schema.registry.url=https://psrc-5mn3g.ap-southeast-2.aws.confluent.cloud --property basic.auth.credentials.source=USER_INFO --property schema.registry.basic.auth.user.info='ETA6QH7YJ7FUUSSN:LxljydcAi0TIWXjRBD5I2eFyZZtZXeFq1hZTPppxZ7S5Cg2FDTj6lfkcLNM3P1tL'
```

Place a tab between the key and the value

```json
{"adviceId":"1000-1000-1000-1000"}	{"product_id":1,"product_name":"Cocoa Pops","product_price":2.5}
{"adviceId":"1000-1000-1000-1001"}	{"product_id":2,"product_name":"Corn Flakes","product_price":2.7}
{"adviceId":"1000-1000-1000-1002"}	{"product_id":2,"product_name":"Honeysmacks","product_price":5.8}
{"adviceId":"1000-1000-1000-1002"}	{"product_id":2,"product_name":"Honeysmacks","product_price":9.7}        
{"adviceId":"1000-1000-1000-1003"}      {"product_id":3,"product_name":"Fruit Loops","product_price":7.45}
{"adviceId":"1000-1000-1000-1003"}      {"product_id":3,"product_name":"Fruit Loops","product_price":7.49} 

```

To consume the avro message

```bash
kafka-avro-console-consumer --bootstrap-server pkc-4n66v.australiaeast.azure.confluent.cloud:9092 --consumer.config config.properties --topic all-types --property schema.registry.url=https://psrc-5mn3g.ap-southeast-2.aws.confluent.cloud --property basic.auth.credentials.source=USER_INFO --property schema.registry.basic.auth.user.info='ETA6QH7YJ7FUUSSN:LxljydcAi0TIWXjRBD5I2eFyZZtZXeFq1hZTPppxZ7S5Cg2FDTj6lfkcLNM3P1tL'
```


In KSQLDB CLI


```sql
set 'auto.offset.reset' = 'earliest';
```


```sql
ksql> select * from "all-types-stream" emit changes;
+----------------------------------------------------------+----------------------------------------------------------+----------------------------------------------------------+
|ADVICEID                                                  |CUSTOMER                                                  |PRODUCT                                                   |
+----------------------------------------------------------+----------------------------------------------------------+----------------------------------------------------------+
```

It returns blank. KSQLDB does not support multiple schemas in a single topic.


Get the current dynamic configurations of kafka

```bash
kafka-configs --bootstrap-server pkc-4n66v.australiaeast.azure.confluent.cloud:9092 --describe --entity-type topics --entity-name dev.digital.advice.coversrecommended --command-config config.properties

Dynamic configs for topic dev.digital.advice.coversrecommended are:
  cleanup.policy=delete sensitive=false synonyms={DYNAMIC_TOPIC_CONFIG:cleanup.policy=delete, DEFAULT_CONFIG:log.cleanup.policy=delete}

```

Change the retention to 100ms

```bash
kafka-configs --bootstrap-server pkc-4n66v.australiaeast.azure.confluent.cloud:9092 --entity-type topics --entity-name dev.digital.advice.coversrecommended --command-config config.properties --alter --add-config retention.ms=100

Completed updating config for topic dev.digital.advice.coversrecommended.
```

This will set the retention low enough for the messages to be purged from the topic.

You can confirm that the configuration was set:

```bash
kafka-configs --bootstrap-server pkc-4n66v.australiaeast.azure.confluent.cloud:9092 --describe --entity-type topics --entity-name dev.digital.advice.coversrecommended --command-config config.properties


Dynamic configs for topic dev.digital.advice.coversrecommended are:
  cleanup.policy=delete sensitive=false synonyms={DYNAMIC_TOPIC_CONFIG:cleanup.policy=delete, DEFAULT_CONFIG:log.cleanup.policy=delete}
  retention.ms=100 sensitive=false synonyms={DYNAMIC_TOPIC_CONFIG:retention.ms=100}

```

Try to then load all the messages in the topic from the beginning:

```bash
kafka-avro-console-consumer --bootstrap-server pkc-4n66v.australiaeast.azure.confluent.cloud:9092 --consumer.config config.properties --topic dev.digital.advice.coversrecommended --property schema.registry.url=https://psrc-5mn3g.ap-southeast-2.aws.confluent.cloud --property basic.auth.credentials.source=USER_INFO --property schema.registry.basic.auth.user.info='ETA6QH7YJ7FUUSSN:LxljydcAi0TIWXjRBD5I2eFyZZtZXeFq1hZTPppxZ7S5Cg2FDTj6lfkcLNM3P1tL' --from-beginning
```

Now you need to remove the retention configuration that was previously added.

```bash
afka-configs --bootstrap-server pkc-4n66v.australiaeast.azure.confluent.cloud:9092 --entity-type topics --entity-name dev.digital.advice.coversrecommended --command-config config.properties --alter --delete-config retention.ms    


```