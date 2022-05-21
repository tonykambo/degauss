# KSQL Tables

The schema:

```
ksql> describe advice;

Name                 : ADVICE
 Field             | Type                                                                  
-------------------------------------------------------------------------------------------
 ENQUIRYID         | VARCHAR(STRING)  (primary key)                                        
 RECOMMENDEDCOVERS | ARRAY<STRUCT<COVERYTYPE VARCHAR(STRING), COVERAMOUNT DECIMAL(10, 2)>> 
 INTERVIEWANSWERS  | ARRAY<STRUCT<QUESTIONID VARCHAR(STRING), ANSWER VARCHAR(STRING)>>     
-------------------------------------------------------------------------------------------
For runtime statistics and query details run: DESCRIBE <Stream,Table> EXTENDED;
```

To insert an array of structs into a table:

```sql
insert into advice (enquiryId, recommendedcovers, interviewanswers) VALUES ('1000-1000-1000-1000', ARRAY[STRUCT(coveryType := 'Life', coverAmount := 25350), STRUCT(coverType := 'Trauma',coverAmount := 0)],ARRAY[STRUCT(questionId := 'customerAge', answer := '30'), STRUCT(questionId := 'height', answer := '172')]);
```

```


ksql> SET CLI COLUMN-WIDTH 15;
ksql> insert into advice (enquiryId, recommendedcovers, interviewanswers) VALUES ('1000-1000-1000-1000', ARRAY[STRUCT(coveryType := 'Life', coverAmount := 25350)],ARRAY[STRUCT(questionId := 'customerAge', answer := '30')]);  
ksql> insert into advice (enquiryId, recommendedcovers, interviewanswers) VALUES ('1000-1000-1000-1000', ARRAY[STRUCT(coveryType := 'Life', coverAmount := 25350), STRUCT(coverType := 'Trauma',coverAmount := 0)],ARRAY[STRUCT(questionId := 'customerAge', answer := '30'), STRUCT(questionId := 'height', answer := '172')]);
ksql> insert into advice (enquiryId, recommendedcovers, interviewanswers) VALUES ('1000-1000-1000-1000', ARRAY[STRUCT(coveryType := 'Life', coverAmount := 90000), STRUCT(coverType := 'Trauma',coverAmount := 120000)],ARRAY[STRUCT(questionId := 'customerAge', answer := '70'), STRUCT(questionId := 'height', answer := '192')]);
ksql> insert into advice (enquiryId, recommendedcovers, interviewanswers) VALUES ('1000-1000-1000-1000', ARRAY[STRUCT(coverType := 'Life', coverAmount := 95000), STRUCT(coverType := 'Trauma',coverAmount := 121000)],ARRAY[STRUCT(questionId := 'customerAge', answer := '71'), STRUCT(questionId := 'height', answer := '193')]);
ksql> insert into advice (enquiryId, recommendedcovers, interviewanswers) VALUES ('1000-1000-1000-1000', ARRAY[STRUCT(coverType := 'Life', coverAmount := 96000), STRUCT(coverType := 'Trauma',coverAmount := 126000)],ARRAY[STRUCT(questionId := 'customerAge', answer := '76'), STRUCT(questionId := 'height', answer := '196')]);

```