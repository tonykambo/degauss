# Create stream from stream


## Create a stream that explodes an array

In this example `section` is an array that will be exploded into individual messages against the same enquiryId in the new stream. 

```sql
create stream exeploded_updateEnquiry_stream as select enquiryId, explode(sections) as section from UPDATEENQUIRYSENDANSWERRESPONSESTREAM where enquiryId is not null;
```

Describing the stream you can see the individual section field. 

```sql
Name                 : EXEPLODED_UPDATEENQUIRY_STREAM
 Field     | Type                                                                                                                                                                                                                                            
-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
 ENQUIRYID | VARCHAR(STRING)  (key)                                                                                                                                                                                                                          
 SECTION   | STRUCT<NAME VARCHAR(STRING), ISSATISFIED BOOLEAN, ENQUIRYLINES ARRAY<STRUCT<NAME VARCHAR(STRING), ISSATISFIED BOOLEAN, QUESTIONS ARRAY<STRUCT<NAME VARCHAR(STRING), ISSATISFIED BOOLEAN, HASANSWER BOOLEAN, ANSWERS ARRAY<VARCHAR(STRING)>>>>>> 
-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
For runtime statistics and query details run: DESCRIBE <Stream,Table> EXTENDED;

```

Another stream is created from the exploded stream that works off the individual array items.  

## Create a stream from the exploded stream that checks against two trigger values.

```sql
create stream advice_interview_completed as select enquiryId, section->name, section->isSatisfied from exeploded_updateEnquiry_stream where (enquiryId is not null AND section-> isSatisfied = true);
```

```sql
ksql> describe ADVICE_INTERVIEW_COMPLETED 
>;

Name                 : ADVICE_INTERVIEW_COMPLETED
 Field       | Type                   
--------------------------------------
 ENQUIRYID   | VARCHAR(STRING)  (key) 
 NAME        | VARCHAR(STRING)        
 ISSATISFIED | BOOLEAN                
--------------------------------------
For runtime statistics and query details run: DESCRIBE <Stream,Table> EXTENDED;
```

To demonstrate this behaviour we publish the first message where the **Advice** section's **isSatisfied** status is false. 

Launch the consumer for the `updateEnquirySendAnswerResponse` topic first:

```json

confluent kafka topic consume updateEnquirySendAnswerResponse --value-format avro --print-key --delimiter ":"

Starting Kafka Consumer. Use Ctrl-C to exit.
??&1000-1001-1002-1017:{"isSatisfied":false,"isOpen":true,"sections":[{"name":"Advice","isSatisfied":false,"enquiryLines":[{"name":"LIFEINSURED_BASIC_DETAILS","isSatisfied":false,"questions":[{"isSatisfied":true,"hasAnswer":true,"answers":["37"],"name":"LIFEINSURED_AGE"},{"answers":[""],"name":"LIFEINSURED_HEIGHT","isSatisfied":false,"hasAnswer":false}]}]}],"buckets":{"array":[{"max":{"value":{"string":"NON_STANDARD"}},"type":"OPTION","name":"DECISION_LIFE","min":{"value":{"string":"NON_STANDARD"}}},{"type":"OPTION","name":"TRAUMA","min":{"value":{"string":"NON_STANDARD"}},"max":{"value":{"string":"NON_STANDARD"}}},{"type":"OPTION","name":"NON_STANDARD","min":{"value":{"string":"STANDARD"}},"max":{"value":{"string":"STANDARD"}}}]},"branch":"Santa Fe","tag":"v1"}

```

and then the consumer for the **stream advice_interview_completed** stream. 

```sql
confluent kafka topic consume pksqlc-w19wwADVICE_INTERVIEW_COMPLETED --value-format avro --print-key --delimiter ":"
Starting Kafka Consumer. Use Ctrl-C to exit.
??&1000-1001-1002-1017:{"ISSATISFIED":{"boolean":true},"NAME":{"string":"Advice"}}


```

The topic created is prefixed with the cluster name. 

You can also observe the stream directly in the KSQL CLI tool:

```sql
ksql> select * from ADVICE_INTERVIEW_COMPLETED emit changes;
+---------------+---------------+---------------+
|ENQUIRYID      |NAME           |ISSATISFIED    |
+---------------+---------------+---------------+
|1000-1001-1002-|Advice         |true           |
|1017           |               |               |
```


```JSON

POST /topics/updateEnquirySendAnswerResponse HTTP/1.1
Host: localhost:8082
Content-Type: application/vnd.kafka.avro.v2+json
Accept: application/vnd.kafka.v2+json
Content-Length: 931

{
    "key_schema":"{\"type\":\"string\"}",
    "value_schema_id": 100017,
    "records": [
        {
            "key":"1000-1001-1002-1017",
            "value": {"branch":"Santa Fe","tag":"v1","isSatisfied":false,"isOpen":true,"sections":[{"name":"Advice","isSatisfied":false,"enquiryLines":[{"name":"LIFEINSURED_BASIC_DETAILS","isSatisfied":false,"questions":[{"name":"LIFEINSURED_AGE","isSatisfied":true,"hasAnswer":true,"answers":["37"]},{"name":"LIFEINSURED_HEIGHT","isSatisfied":false,"hasAnswer":false,"answers":[""]}]}]}],"buckets":{"array":[{"type":"OPTION","name":"DECISION_LIFE","min":{"value":{"string":"NON_STANDARD"}},"max":{"value":{"string":"NON_STANDARD"}}},{"type":"OPTION","name":"TRAUMA","min":{"value":{"string":"NON_STANDARD"}},"max":{"value":{"string":"NON_STANDARD"}}},{"type":"OPTION","name":"NON_STANDARD","min":{"value":{"string":"STANDARD"}},"max":{"value":{"string":"STANDARD"}}}]}}
        }
    ]
}

```




-- 
## Archive

```sql
ksql> CREATE STREAM COVERSRECOMMENDED WITH (kafka_topic = 'covers_recommended') AS SELECT a.ENQUIRYINTERACTIONID, a.ENQUIRYID, a.BRANCH, a.ISSATISFIED FROM UPDATEENQUIRYSENDANSWERRESPONSESTREAM AS a WHERE a.ISSATISFIED = TRUE EMIT CHANGES;

 Message                                        
------------------------------------------------
 Created query with ID CSAS_COVERSRECOMMENDED_7 
------------------------------------------------
```

```sql
ksql> 
    CREATE STREAM COVERSRECOMMENDED 
    WITH (kafka_topic = 'covers_recommended') AS 
    SELECT a.ENQUIRYINTERACTIONID, 
            a.ENQUIRYID, 
            a.BRANCH, 
            a.ISSATISFIED, 
            a.BUCKETS[0]->min-value,  
    FROM UPDATEENQUIRYSENDANSWERRESPONSESTREAM AS a 
    WHERE a.ISSATISFIED = TRUE EMIT CHANGES;

 Message                                        
------------------------------------------------
 Created query with ID CSAS_COVERSRECOMMENDED_7 
------------------------------------------------
```