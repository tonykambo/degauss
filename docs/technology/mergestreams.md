# Merging streams;


Create a merged stream starting with the european stream. The 'data_source' attribute is a pseudo attribute used to distinguish the source of the streams.

```sql
create stream rr_world as select 'Europe' as data_source, * from rr_europe_raw;
```

```sql
insert into rr_world select 'Americas' as data_source, * from rr_america_raw;


create stream advice_cdm () from (topic='advicecdm');

insert into advice_cdm as select * as data_sour ce, * from advicecoversissued

create stream adviceforquoting as select * as data_source * from advice_cdml;

```

Conditional stream

```sql

CREATE STREAM carsize_buckets
AS
SELECT CARID, 
CASE WHEN CARLENGTH < 2.0 THEN 'small'
    WHEN CARLENGTH < 4.0 THEN 'medium'
    ELSE 'large' END AS carsize_result
FROM carsizes
EMIT CHANGES;

```