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
