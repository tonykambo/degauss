# Basic KSQLDB CLI commands

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

