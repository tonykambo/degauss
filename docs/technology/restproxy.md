# The Rest Proxy 


The `ccloud-kafka-rest.properties` file:

```bash
# Kafka
bootstrap.servers=pkc-4n66v.australiaeast.azure.confluent.cloud:9092
security.protocol=SASL_SSL
sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule \
required username="FRORDMSLBDBFH43W" password="hLRs3mNZF1bpr5p3UZM6p4TqdG0M+Lhg/TlhFS32BsX7fMrpD/h7lzUiZ6VfHtct";
ssl.endpoint.identification.algorithm=https
sasl.mechanism=PLAIN
client.bootstrap.servers=pkc-4n66v.australiaeast.azure.confluent.cloud:9092
client.security.protocol=SASL_SSL
client.sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule \
required username="FRORDMSLBDBFH43W" password="hLRs3mNZF1bpr5p3UZM6p4TqdG0M+Lhg/TlhFS32BsX7fMrpD/h7lzUiZ6VfHtct";
client.ssl.endpoint.identification.algorithm=https
client.sasl.mechanism=PLAIN
# Confluent Cloud Schema Registry
schema.registry.url=https://psrc-5mn3g.ap-southeast-2.aws.confluent.cloud
basic.auth.credentials.source=USER_INFO
schema.registry.basic.auth.user.info=ETA6QH7YJ7FUUSSN:LxljydcAi0TIWXjRBD5I2eFyZZtZXeFq1hZTPppxZ7S5Cg2FDTj6lfkcLNM3P1tL
```

Run it as follows

```bash
kafka-rest-start ccloud-kafka-rest.properties
```
