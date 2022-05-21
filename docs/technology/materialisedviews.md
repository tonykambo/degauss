# Constructing a materialised view in Confluent Cloud (Kafka) using ksqlDB

- Using Kafka to store a copy of customer data from a CRM system. 
- Cater for new customers added to the system. 
- Cater for customer details being changed. 
- Providing a materialised view where only the latest changes to the customer are retrieved. 

Set up a Confluent Cloud account with ksqlDB.

```bash
docker run --rm -it confluentinc/ksqldb-cli:0.23.1 ksql -u <KSQL_USER_ID> -p <KSQL_PASSWORD> "https://<confluent-cloud-hostname>.aws.confluent.cloud:443"
```

```sql
create stream customers (
    id INT KEY, 
    firstname VARCHAR, 
    lastname VARCHAR, 
    email VARCHAR, 
    residency VARCHAR, 
    dateOfBirth TIMESTAMP, 
    sexAtBirth VARCHAR, 
    occupation VARCHAR) 
    WITH (
        KAFKA_TOPIC = 'customers', 
        value_format = 'JSON', 
        PARTITIONS = 6
    ); 
```

Show the stream and topics created:

```sql

ksql> show streams;

 Stream Name                | Kafka Topic                 | Key Format | Value Format | Windowed 
-------------------------------------------------------------------------------------------------
 CUSTOMERS                  | customers                   | KAFKA      | JSON         | false    
 KSQL_PROCESSING_LOG        | pksqlc-vn80j-processing-log | KAFKA      | JSON         | false    
 RIDERLOCATIONS             | quickstart-locations        | KAFKA      | JSON         | false    
 STARTENQUIRYREQUESTSTREAM  | startEnquiryRequest-topic   | KAFKA      | JSON         | false    
 STARTENQUIRYRESPONSESTREAM | startEnquiryResponse-topic  | KAFKA      | JSON         | false    
 USERS                      | users                       | KAFKA      | JSON         | false    
-------------------------------------------------------------------------------------------------

ksql> show topics;

 Kafka Topic                        | Partitions | Partition Replicas 
----------------------------------------------------------------------
 customers                          | 6          | 3                  
 pksqlc-vn80j-processing-log        | 8          | 3                  
 pksqlc-vn80jCURRENTLOCATION        | 1          | 3                  
 pksqlc-vn80jRIDERSNEARMOUNTAINVIEW | 1          | 3                  
 quickstart-locations               | 1          | 3                  
 startEnquiryRequest-topic          | 1          | 3                  
 startEnquiryResponse-topic         | 1          | 3                  
 testtopic                          | 6          | 3                  
 users                              | 1          | 3                  
----------------------------------------------------------------------

```

Adding initial records into the stream:

```sql

insert into customers (id, firstname, lastname, email, residency, dateofbirth, sexatbirth, occupation) values (1,'John','Smith','john@nowhere.com','NZCitizen','1955-11-05T06:00:00', 'Male','SelfEmployed');

insert into customers (id, firstname, lastname, email, residency, dateofbirth, sexatbirth, occupation) values (2,'Peter','Jones','peter@thisthat.com','NZCitizen','1945-10-06T06:00:00', 'Male','Employed');

insert into customers (id, firstname, lastname, email, residency, dateofbirth, sexatbirth, occupation) values (3,'Felicity','Hoffman','felicity@thehoff.org','NZCitizen','1965-12-24T06:00:00', 'Female','Student');
```


In order to update the details of a customer, another record is inserted with the same customer id and the new value. 

```sql

ksql> insert into customers (id, firstname, lastname, email, residency, dateofbirth, sexatbirth, occupation) values (1,'Johnathan','Smith','john@nowhere.com','NZCitizen','1955-11-05T06:00:00', 'Male','SelfEmployed');

ksql> insert into customers (id, firstname, lastname, email, residency, dateofbirth, sexatbirth, occupation) values (3,'Felicity','Hofferman','felicity@thehoffman.org','NZCitizen','1965-12-24T06:00:00', 'Female','Student');

```

Both customer id 1 and 3 were changed. 

When running a query against the stream all records will be returned:

```sql
ksql> select id, firstname, lastname, email from customers where id = 1;
+------------------------------+------------------------------+------------------------------+------------------------------+
|ID                            |FIRSTNAME                     |LASTNAME                      |EMAIL                         |
+------------------------------+------------------------------+------------------------------+------------------------------+
|1                             |John                          |Smith                         |john@nowhere.com              |
|1                             |Johnathan                     |Smith                         |john@nowhere.com              |
```

To create a materialised view that displays only the last update:

```sql
ksql> create table latest_customers as select id, LATEST_BY_OFFSET(firstname) as firstname, LATEST_BY_OFFSET(lastname) as lastname, LATEST_BY_OFFSET(email) as email FROM customers GROUP BY id EMIT CHANGES;
```

```sql

ksql> show tables;

 Table Name             | Kafka Topic                        | Key Format | Value Format | Windowed 
----------------------------------------------------------------------------------------------------
 CURRENTLOCATION        | pksqlc-vn80jCURRENTLOCATION        | KAFKA      | JSON         | false    
 LATEST_CUSTOMERS       | pksqlc-vn80jLATEST_CUSTOMERS       | KAFKA      | JSON         | false    
 RIDERSNEARMOUNTAINVIEW | pksqlc-vn80jRIDERSNEARMOUNTAINVIEW | KAFKA      | JSON         | false    
----------------------------------------------------------------------------------------------------

```

```sql
ksql> show topics;

 Kafka Topic                        | Partitions | Partition Replicas 
----------------------------------------------------------------------
 customers                          | 6          | 3                  
 pksqlc-vn80j-processing-log        | 8          | 3                  
 pksqlc-vn80jCURRENTLOCATION        | 1          | 3                  
 pksqlc-vn80jLATEST_CUSTOMERS       | 6          | 3                  
 pksqlc-vn80jRIDERSNEARMOUNTAINVIEW | 1          | 3                  
 quickstart-locations               | 1          | 3                  
 startEnquiryRequest-topic          | 1          | 3                  
 startEnquiryResponse-topic         | 1          | 3                  
 testtopic                          | 6          | 3                  
 users                              | 1          | 3                  
----------------------------------------------------------------------
```

When running the query for customer id 1 against the materialised view:

```sql
ksql> select id, firstname, lastname, email from latest_customers where id=1;
+------------------------------+------------------------------+------------------------------+------------------------------+
|ID                            |FIRSTNAME                     |LASTNAME                      |EMAIL                         |
+------------------------------+------------------------------+------------------------------+------------------------------+
|1                             |Johnathan                     |Smith                         |john@nowhere.com              |


ksql> select * from latest_customers where id=3;
+------------------------------+------------------------------+------------------------------+------------------------------+
|ID                            |FIRSTNAME                     |LASTNAME                      |EMAIL                         |
+------------------------------+------------------------------+------------------------------+------------------------------+
|3                             |Felicity                      |Hofferman                     |felicity@thehoffman.org       |

```

Only one the latest representation is returned. 