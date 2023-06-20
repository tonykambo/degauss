# Keying and Partitioning

## Keying and Re-keying

To create a stream with a new key

```sql

create stream mystreamrekeyed as select & from mystreamraw partition by city_name;
```

Now you can create a table

```sql
create table weathernow with (kafka_topic='WEATHERREKEYED', value_format='AVRO', key='CITY_NAME');
```

So this can now work

```sql
select * from weathernow where city_name = "Melbourne";

```

## Repartitioning

You can't join a stream with a table that is based on different partitions. Joining re-quires co-partitioning, both need to have the same number of partitions and be keyed on the same item. 

```sql
select dp.driver_name, ct.countryname, dp.rating
FROM DRIVER_PROFILE dp
LEFT JOIN COUNTRYTABLE ct on ct.countrycode=dp.countrycode;

```

Error message:

```Can't join DRIVER_PROFILE and COUNTRYTABLE since the nuber of partitions don't match.```

Create a new stream with adjusting the partition.

```sql
create stream driverprofile_rekeyed with (partitions=1) as select * from driver_profile partition by driver_name;

```
