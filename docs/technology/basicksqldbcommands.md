# Basic KSQLDB CLI commands

To launch the CLI make sure you have the right version on macOS

```bash
/usr/libexec/java_home -V                           
Matching Java Virtual Machines (2):
    17.0.2 (arm64) "Oracle Corporation" - "Java SE 17.0.2" /Library/Java/JavaVirtualMachines/jdk-17.0.2.jdk/Contents/Home
    11.0.15 (x86_64) "Oracle Corporation" - "Java SE 11.0.15" /Library/Java/JavaVirtualMachines/jdk-11.0.15.jdk/Contents/Home
/Library/Java/JavaVirtualMachines/jdk-17.0.2.jdk/Contents/Home
```

Select version 11

```bash
export JAVA_HOME=`/usr/libexec/java_home -v 11.0.15`

java -version
java version "11.0.15" 2022-04-19 LTS
Java(TM) SE Runtime Environment 18.9 (build 11.0.15+8-LTS-149)
Java HotSpot(TM) 64-Bit Server VM 18.9 (build 11.0.15+8-LTS-149, mixed mode)

```
Launch the local KSQLDB CLI pointing to Confluent Cloud:

```bash

ksql -u 6U6B2MYHDJNNGZ2E -p qgPSo1crs7jbyl4K8/xOeIGJq8VfEToRa9sONzXCKirpPqS8JqMDVTxQqfzjsUf8 https://pksqlc-j9z9m.australiaeast.azure.confluent.cloud:443


```

Delete the stream

```bash
drop stream insuranceEnquiries
```

Delete stream and drop the topic

```bash
drop stream <mystream> delete topic;
```

Set the column width in the ksqlDB CLI

```sql
SET CLI COLUMN-WIDTH 15
```

KSQL should start reading from the beginning of a topic:

```sql
SET 'auto.offset.reset'='earliest';
```



```
ksql> SELECT CUSTOMERID, KSQL_COL_1 FROM TBL_ARRAY_DEMO_0010 EMIT CHANGES;
+---------------------------+----------------------------------------------------------------------------------+
|CUSTOMERID                 |KSQL_COL_1                                                                        |
+---------------------------+----------------------------------------------------------------------------------+
|6454214343400548           |[{orderID=800000, ItemName=Apple iPad, StorePickUpFlag=No, OrderDate=09/01/2018}, |
|                           |{orderID=799994, ItemName=Nintendo Switch, StorePickUpFlag=Yes, OrderDate=09/01/20|
|                           |18}, {orderID=799993, ItemName=Jelly Comb folding Bluetooth keyboard, StorePickUpF|
|                           |lag=No, OrderDate=06/24/2019}]                                                    |
|5759544343400808           |[{orderID=799997, ItemName=Arlo Pro, StorePickUpFlag=Yes, OrderDate=06/24/2019}, {|
|                           |orderID=799992, ItemName=Apple TV 4K Streaming Console, StorePickUpFlag=No, OrderD|
|                           |ate=01/28/2020}]                                                                  |
|8484214343400844           |[{orderID=799999, ItemName=Apple TV 4K Streaming Console, StorePickUpFlag=No, Orde|
|                           |rDate=09/01/2018}, {orderID=799998, ItemName=Nintendo Switch, StorePickUpFlag=Yes,|
|                           | OrderDate=01/28/2020}, {orderID=799996, ItemName=Tile Mate Item Finder 4-Pack Com|
|                           |bo, StorePickUpFlag=No, OrderDate=01/28/2020}, {orderID=799995, ItemName=Apple iPa|
|                           |d, StorePickUpFlag=Yes, OrderDate=01/28/2020}, {orderID=799991, ItemName=Jelly Com|
|                           |b folding Bluetooth keyboard, StorePickUpFlag=No, OrderDate=06/24/2019}]          |

```

## KSQLDB REST API

Get the KSQLDB cluster identifier:

```bash
confluent ksql cluster list
       ID       |       Name       | Topic Prefix |   Kafka    | Storage |                           Endpoint                           | Status  
----------------+------------------+--------------+------------+---------+--------------------------------------------------------------+---------
  lksqlc-gqwzkr | ksqlDB_cluster_0 | pksqlc-j9z9m | lkc-7yq8z2 |     125 | https://pksqlc-j9z9m.australiaeast.azure.confluent.cloud:443 | UP      

```

Create an API key pair.

```bash
confluent api-key create --resource lksqlc-gqwzkr
It may take a couple of minutes for the API key to be ready.
Save the API key and secret. The secret is not retrievable later.
+---------+------------------------------------------------------------------+
| API Key | LHWK74VPGWZXPO6F                                                 |
| Secret  | 0Zfp75Dtyr9JiNcZ8CNXlfCqPfun55iHGZleEyQZ9HvmyExwN3HUOQlbwru0xo7Y |
+---------+------------------------------------------------------------------+
```


List the streams.

```http
POST /ksql HTTP/1.1
Host: pksqlc-j9z9m.australiaeast.azure.confluent.cloud:443
Accept: application/vnd.ksql.v1+json
Authorization: Basic TEhXSzc0VlBHV1pYUE82RjowWmZwNzVEdHlyOUppTmNaOENOWGxmQ3FQZnVuNTVpSEdabGVFeVFaOUh2bXlFeHdOM0hVT1FsYndydTB4bzdZ
Content-Type: application/json
Content-Length: 60

{
    "ksql": "LIST STREAMS;",
    "streamsProperties": {}
}

```

The response:

```json
[
    {
        "@type": "streams",
        "statementText": "LIST STREAMS;",
        "streams": [
            {
                "type": "STREAM",
                "name": "all-types-stream",
                "topic": "all-types",
                "keyFormat": "AVRO",
                "valueFormat": "AVRO",
                "isWindowed": false
            },
            {
                "type": "STREAM",
                "name": "KSQL_PROCESSING_LOG",
                "topic": "pksqlc-j9z9m-processing-log",
                "keyFormat": "KAFKA",
                "valueFormat": "JSON",
                "isWindowed": false
            },
            {
                "type": "STREAM",
                "name": "ADVICEEVENTSSTREAM",
                "topic": "advice-events",
                "keyFormat": "AVRO",
                "valueFormat": "AVRO",
                "isWindowed": false
            },
            {
                "type": "STREAM",
                "name": "DEV_DIGITAL_ADVICE_STREAM_COVERSRECOMMENDED",
                "topic": "dev.digital.advice.coversrecommended",
                "keyFormat": "AVRO",
                "valueFormat": "AVRO",
                "isWindowed": false
            }
        ],
        "warnings": []
    }
]

```