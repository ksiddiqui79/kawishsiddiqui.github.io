title: Teradata functions in Redshift UDF
description: Analyze & Vacuum Schema Utility.
keywords: wlm, workload, monitoring, rule, qmr, encrypt, decryption, decrypt, encryption, udf, amazon, function, utility, vacuum, analyze, data lake, datalake, formation, case study, shell, bash, sh, split, size, console, aws, files, tips, cheat, shell command, bash command, script, shell script, sql, teradata, redshift, vba, database, data warehouse, etl, custom, big data

<script>
 </script>
 ---
!!! note "Disclaimer"  
     This code must not be used for production use as is. This sample is developed only to educate the users in implementing Teradata functions equalent in Redshift using UDFs feature.


# OTRANSLATE

```SQL
CREATE FUNCTION AWS_TERADATA_EXT.OTRANSLATE (VARCHAR, VARCHAR, VARCHAR )
  RETURNS VARCHAR
STABLE
AS $$
  SELECT DECODE(TRIM(TRANSLATE ($1, $2, $3)), '', NULL, TRANSLATE ($1, $2, $3))
$$ LANGUAGE SQL;
```

## Vacuum

[This script](https://github.com/awslabs/amazon-redshift-utils/blob/master/src/AnalyzeVacuumUtility/analyze-vacuum-schema.py) runs vacuum in two phases,

### Phase 1: 

Identify and run vacuum based on the alerts recorded in ``stl_alert_event_log``. ``stl_alert_event_log``, records an alert when the query optimizer identifies conditions that might indicate performance issues. We can use the ``stl_alert_event_log`` table to identify the top 25 tables that need vacuum. The script uses SQL to get the list of tables and number of alerts, which indicate that vacuum is required.

Variables affecting this phase:

* `goback_no_of_days`: To control number days to look back from CURRENT_DATE

### Phase 2: 

Identify and run vacuum based on certain thresholds related to table statistics (Like unsorted > 10% and Stats Off > 10% and limited to specific table sizes.

Variables affecting this phase:

* `stats_off_pct`: To control the threshold of statistics inaccuracy
* `min_unsorted_pct`: To control the lower limit of unsorted blocks
* `max_unsorted_pct`: To control the upper limit of unsorted blocks (preventing vacuum on super large tables)
* `max_tbl_size_mb`: To control when a table is too large to be vacuumed by this utility


## Analyze

This script runs Analyze in two phases:

### Phase 1: 

Run ANALYZE based on the alerts recorded in ``stl_explain`` & ``stl_alert_event_log``. 

### Phase 2: 

Run ANALYZE based the `stats_off` metric in `svv_table_info`. If table has a `stats_off_pct` > 10%, then the script runs ANALYZE command to update the statistics.

## Summary of Parameters:

| Parameter | Mandatory | Default Value |
| :--- | :---: | :--- |
| --db | Yes | |
| --db-user | Yes | |
|--db-pwd | Yes | |
|--db-host | Yes | |
|--db-port | No | 5439 |
|--db-conn-opts | No | |
|--schema-name | No | Public |
|--table-name | No | |
|--output-file | Yes | |
|--debug | No | False |
|--slot-count | No | 1 |
|--ignore-errors | No | False |
|--query_group | No | None |
|--analyze-flag | No | False |
|--vacuum-flag | No | False |
|--vacuum-parameter | No | FULL |
|--min-unsorted-pct | No | 0.05 |
|--max-unsorted-pct | No | 0.5 |
|--stats-off-pct | No | 0.1 |
|--stats-threshold | No | 0.1 |
|--max-table-size-mb | No | 700*1024 |
|--predicate-cols | No | False |

The above parameter values depend on the cluster type, table size, available system resources and available ‘Time window’ etc. The default values provided here are based on ds2.8xlarge, 8 node cluster. It may take some trial and error to come up with correct parameter values to vacuum and analyze your table(s). If table size is greater than certain size (`max_table_size_mb`) and has a large unsorted region (`max_unsorted_pct`), consider performing a deep copy, which will be much faster than a vacuum.

As VACUUM & ANALYZE operations are resource intensive, you should ensure that this will not adversely impact other database operations running on your cluster. AWS has thoroughly tested this software on a variety of systems, but cannot be responsible for the impact of running the utility against your database.

### Parameter Guidance

### Schema Name

The utility will accept a valid schema name, or alternative a regular expression pattern which will be used to match to all schemas in the database. This uses Posix regular expression syntax. You can use `(.*)` to match all schemas. 

#### Slot Count

Sets the number of query slots a query will use.

Workload management (WLM) reserves slots in a service class according to the concurrency level set for the queue (for example, if concurrency level is set to 5, then the service class has 5 slots). WLM allocates the available memory for a service class equally to each slot. For more information, see Implementing Workload Management.

For operations where performance is heavily affected by the amount of memory allocated, such as Vacuum, increasing the value of wlm_query_slot_count can improve performance. In particular, for slow Vacuum commands, inspect the corresponding record in the `SVV_VACUUM_SUMMARY` view. If you see high values (close to or higher than 100) for `sort_partitions` and `merge_increments` in the `SVV_VACUUM_SUMMARY` view, consider increasing the value for `wlm_query_slot_count` the next time you run Vacuum against that table.

Increasing the value of wlm_query_slot_count limits the number of concurrent queries that can be run.

Note:

If the value of `wlm_query_slot_count` is larger than the number of available slots (concurrency level) for the queue targeted by the user, the utilty will fail. If you encounter an error, decrease `wlm_query_slot_count` to an allowable value.

#### analyze-flag

Flag to turn ON/OFF ANALYZE functionality (True or False). If you want run the script to only perform ANALYZE on a schema or table, set this value ‘False’ : Default = ‘False’.

#### vacuum-flag

Flag to turn ON/OFF VACUUM functionality (True or False). If you want run the script to only perform VACUUM on a schema or table, set this value ‘False’ : Default = ‘False’.

#### vacuum-parameter

Specify vacuum parameters [ `FULL | SORT ONLY | DELETE ONLY | REINDEX` ] Default = FULL

#### min-unsorted-pct

Minimum unsorted percentage (%) to consider a table for vacuum: Default = 5%.

#### max-unsorted-pct

Maximum unsorted percentage(%) to consider a table for vacuum : Default = 50%.

#### stats-off-pct

Minimum stats off percentage(%) to consider a table for analyze : Default = 10%

#### max-table-size-mb 

Maximum table size 700GB in MB : Default = 700*1024 MB

#### predicate-cols

Analyze predicate columns only. Default = False


## Sample Usage

```
python analyze-vacuum-schema.py  --db <> --db-user <> --db-pwd <> --db-port 8192 --db-host aaa.us-west-2.redshift.amazonaws.com --schema-name public  --table-name customer_v6 --output-file /Users/test.log --debug True  --ignore-errors False --slot-count 2 --min-unsorted-pct 5 --max-unsorted-pct 50 --stats-off-pct 10 --max-table-size-mb 700*1024
```

## Install Notes

```
sudo easy_install pip  
sudo pip install 
```

## Limitations

1. Script runs all VACUUM commands sequentially. Currently in Redshift multiple concurrent vacuum operations are not supported. 
2. Script runs all ANALYZE commands sequentially not concurrently.
3. Does not support column level ANALYZE. 
4. Multiple schemas are not supported.
5. Skew factor is not considered.