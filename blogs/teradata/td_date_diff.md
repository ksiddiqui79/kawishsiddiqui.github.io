title: date_diff Teradata UDF (SQL)
description: date_diff Teradata UDF (SQL)
keywords: utility, vacuum, analyze, data lake, datalake, formation, case study, shell, bash, sh, split, size, console, aws, files, tips, cheat, shell command, bash command, script, shell script, sql, teradata, redshift, vba, database, data warehouse, etl, custom, big data
#date_diff Teradata UDF (SQL)
This function will return the difference between two timestamps.
Input parameters are :
Desire Difference (e.g. year, month, day, minute, hour, second)
Start timestamp (e.g. ‘2000-10-18 22:03:07’)
End timestamp(e.g. ‘2018-12-18 08:10:07’)

##Syntax date_diff():

```sql
date_diff(‘year’, TIMESTAMP ‘2000-10-18 22:03:07’, TIMESTAMP ‘2018-12-18 08:10:07’)

```
## Function Code
```sql
REPLACE FUNCTION syslib.date_diff(vDiff VARCHAR(255), StartTimestamp TIMESTAMP, EndTimeStamp TIMESTAMP)
  RETURNS INT
  LANGUAGE SQL
  CONTAINS SQL
  DETERMINISTIC
  SQL SECURITY DEFINER
  COLLATION INVOKER
  INLINE TYPE 1
  RETURN 
      ROUND(
      CASE WHEN UPPER(vDiff) = 'YEAR' THEN
              (EXTRACT(DAY FROM (EndTimeStamp  - StartTimestamp DAY(4) TO SECOND)) / 365)
          WHEN  UPPER(vDiff) = 'MONTH' THEN
              (EXTRACT(DAY FROM (EndTimeStamp  - StartTimestamp DAY(4) TO SECOND)) / 365)*12
          WHEN  UPPER(vDiff) = 'DAY' THEN
              (EXTRACT(DAY FROM (EndTimeStamp  - StartTimestamp DAY(4) TO SECOND)))
          WHEN  UPPER(vDiff) = 'HOUR' THEN
              (EXTRACT(DAY FROM (EndTimeStamp  - StartTimestamp DAY(4) TO SECOND)) * (24))
             + (EXTRACT(HOUR FROM (EndTimeStamp - StartTimestamp DAY(4) TO SECOND)))
          WHEN  UPPER(vDiff) = 'MINUTE' THEN
              (EXTRACT(DAY FROM (EndTimeStamp  - StartTimestamp DAY(4) TO SECOND)) * (24*60))
             + (EXTRACT(HOUR FROM (EndTimeStamp - StartTimestamp DAY(4) TO SECOND)) * 60)
             + (EXTRACT(MINUTE FROM (EndTimeStamp - StartTimestamp DAY(4) TO SECOND)) )
          WHEN  UPPER(vDiff) = 'SECOND' THEN
              (EXTRACT(DAY FROM (EndTimeStamp  - StartTimestamp DAY(4) TO SECOND)) * (246060))
             + (EXTRACT(HOUR FROM (EndTimeStamp - StartTimestamp DAY(4) TO SECOND)) * (60*60))
             + (EXTRACT(MINUTE FROM (EndTimeStamp - StartTimestamp DAY(4) TO SECOND)) * 60)
             + EXTRACT(SECOND FROM (EndTimeStamp - StartTimestamp DAY(4) TO SECOND))
      ELSE -1
      END
     )
     ; 
```
##Testing SQL

```sql
WITH tsval AS 
   (
      SELECT TIMESTAMP '2000-10-18 22:03:07' starttime
           , TIMESTAMP '2018-12-18 22:03:07' endtime 
   ),
   dv AS
   (
      SELECT 'YEAR' (VARCHAR(255)) dval FROM (SELECT 1 a) a UNION
      SELECT 'MONTH' (VARCHAR(255)) dval FROM (SELECT 1 a) a UNION
      SELECT 'day' (VARCHAR(255)) dval FROM (SELECT 1 a) a UNION
      SELECT 'hOuR' (VARCHAR(255)) dval FROM (SELECT 1 a) a UNION
      SELECT 'SeCoNd' (VARCHAR(255)) dval FROM (SELECT 1 a) a UNION
      SELECT 'MiNute' (VARCHAR(255)) dval FROM (SELECT 1 a) a  
   )
     SELECT dval , starttime, endtime, syslib.date_diff(dval, starttime, endtime) 
     FROM tsval, dv ;
```