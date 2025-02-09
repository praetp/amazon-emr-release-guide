# Requirements for the EMRFS S3\-optimized committer<a name="emr-spark-committer-reqs"></a>

The EMRFS S3\-optimized committer is used when the following conditions are met:
+ You run Spark jobs that use Spark SQL, DataFrames, or Datasets to write files to Amazon S3\. Starting with Amazon EMR 6\.4\.0, this committer can be used for all common formats including parquet, ORC, and text\-based formats \(including CSV and JSON\)\. For release versions prior to Amazon EMR 6\.4\.0, only the Parquet format is supported\.
+ Multipart uploads are enabled in Amazon EMR\. This is the default\. For more information, see [The EMRFS S3\-optimized committer and multipart uploads](emr-spark-committer-multipart.md)\. 
+ Spark's built\-in Parquet support is used\. Built\-in Parquet support is used in the following circumstances:
  + `spark.sql.hive.convertMetastoreParquet` set to `true`\. This is the default setting\.
  + When jobs write to Parquet data sources or tables—for example, the target table is created with the `USING parquet` clause\. 
  + When jobs write to non\-partitioned Hive metastore Parquet tables\. Spark's built\-in Parquet support does not support partitioned Hive tables, which is a known limitation\. For more information, see [Hive metastore Parquet table conversion](https://spark.apache.org/docs/latest/sql-data-sources-parquet.html#hive-metastore-parquet-table-conversion) in the Apache Spark SQL, DataFrames and Datasets Guide\.
+ Spark job operations that write to a default partition location—for example, `${table_location}/k1=v1/k2=v2/`—use the committer\. The committer is not used if a job operation writes to a custom partition location—for example, if a custom partition location is set using the `ALTER TABLE SQL` command\.
+ The following values for Spark must be used:
  + The `spark.sql.parquet.fs.optimized.committer.optimization-enabled` property must be set to `true`\. This is the default setting with Amazon EMR 5\.20\.0 and later\. With Amazon EMR 5\.19\.0, the default value is `false`\. For information about configuring this value, see [Enable the EMRFS S3\-optimized committer for Amazon EMR 5\.19\.0](emr-spark-committer-enable.md)\.
  + `spark.sql.hive.convertMetastoreParquet` must be set to `true` if writing to non\-partitioned Hive metastore tables\. This is the default setting\.
  + `spark.sql.parquet.output.committer.class` must be set to `com.amazon.emr.committer.EmrOptimizedSparkSqlParquetOutputCommitter`\. This is the default setting\.
  + `spark.sql.sources.commitProtocolClass` must be set to `org.apache.spark.sql.execution.datasources.SQLEmrOptimizedCommitProtocol` or `org.apache.spark.sql.execution.datasources.SQLHadoopMapReduceCommitProtocol`\. `org.apache.spark.sql.execution.datasources.SQLEmrOptimizedCommitProtocol` is the default setting for the EMR 5\.x series version 5\.30\.0 and higher, and for the EMR 6\.x series version 6\.2\.0 and higher\. `org.apache.spark.sql.execution.datasources.SQLHadoopMapReduceCommitProtocol` is the default setting for previous EMR versions\.
  + If Spark jobs overwrite partitioned Parquet datasets with dynamic partition columns, then the `partitionOverwriteMode` write option and `spark.sql.sources.partitionOverwriteMode` must be set to `static`\. This is the default setting\.
**Note**  
The `partitionOverwriteMode` write option was introduced in Spark 2\.4\.0\. For Spark version 2\.3\.2, included with Amazon EMR release 5\.19\.0, set the `spark.sql.sources.partitionOverwriteMode` property\. 

## When the EMRFS S3\-optimized committer is not used<a name="emr-spark-committer-reqs-anti"></a>

Generally, the EMRFS S3\-optimized committer is not used in the following situations\.


****  

| Situation | Why the committer is not used | 
| --- | --- | 
| When you write to HDFS | The committer only supports writing to Amazon S3 using EMRFS\. | 
| When you use the S3A file system | The committer only supports EMRFS\. | 
| When you use MapReduce or Spark's RDD API | The committer only supports using SparkSQL, DataFrame, or Dataset APIs\. | 

The following Scala examples demonstrate some additional situations that prevent the EMRFS S3\-optimized committer from being used in whole \(the first example\) and in part \(the second example\)\.

**Example –Dynamic partition overwrite mode**  
The following Scala example instructs Spark to use a different commit algorithm, which prevents use of the EMRFS S3\-optimized committer altogether\. The code sets the `partitionOverwriteMode` property to `dynamic` to overwrite only those partitions to which you're writing data\. Then, dynamic partition columns are specified by `partitionBy`, and the write mode is set to `overwrite`\.   
You must configure all three settings to avoid using the EMRFS S3\-optimized committer\. When you do so, Spark executes a different commit algorithm that uses Spark's staging directory, which is a temporary directory created under the output location that starts with `.spark-staging`\. The algorithm sequentially renames partition directories, which can negatively impact performance\.  

```
val dataset = spark.range(0, 10)
  .withColumn("dt", expr("date_sub(current_date(), id)"))

dataset.write.mode("overwrite")
  .option("partitionOverwriteMode", "dynamic")
  .partitionBy("dt")
  .parquet("s3://EXAMPLE-DOC-BUCKET/output")
```
The algorithm in Spark 2\.4\.0 follows these steps:  

1. Task attempts write their output to partition directories under Spark's staging directory—for example, `${outputLocation}/spark-staging-${jobID}/k1=v1/k2=v2/`\.

1. For each partition written, the task attempt keeps track of relative partition paths—for example, `k1=v1/k2=v2`\.

1. When a task completes successfully, it provides the driver with all relative partition paths that it tracked\.

1. After all tasks complete, the job commit phase collects all the partition directories that successful task attempts wrote under Spark's staging directory\. Spark sequentially renames each of these directories to its final output location using directory tree rename operations\.

1. The staging directory is deleted before the job commit phase completes\.

**Example –Custom partition location**  
In this example, the Scala code inserts into two partitions\. One partition has a custom partition location\. The other partition uses the default partition location\. The EMRFS S3\-optimized committer is only used for writing task output to the partition that uses the default partition location\.  

```
val table = "dataset"
val location = "s3://bucket/table"
                            
spark.sql(s"""
  CREATE TABLE $table (id bigint, dt date) 
  USING PARQUET PARTITIONED BY (dt) 
  LOCATION '$location'
""")
                            
// Add a partition using a custom location
val customPartitionLocation = "s3://bucket/custom"
spark.sql(s"""
  ALTER TABLE $table ADD PARTITION (dt='2019-01-28') 
  LOCATION '$customPartitionLocation'
""")
                            
// Add another partition using default location
spark.sql(s"ALTER TABLE $table ADD PARTITION (dt='2019-01-29')")
                            
def asDate(text: String) = lit(text).cast("date")
                            
spark.range(0, 10)
  .withColumn("dt",
    when($"id" > 4, asDate("2019-01-28")).otherwise(asDate("2019-01-29")))
  .write.insertInto(table)
```
The Scala code creates the following Amazon S3 objects:  

```
custom/part-00001-035a2a9c-4a09-4917-8819-e77134342402.c000.snappy.parquet
custom_$folder$
table/_SUCCESS
table/dt=2019-01-29/part-00000-035a2a9c-4a09-4917-8819-e77134342402.c000.snappy.parquet
table/dt=2019-01-29_$folder$
table_$folder$
```
When writing to partitions at custom locations, Spark uses a commit algorithm similar to the previous example, which is outlined below\. As with the earlier example, the algorithm results in sequential renames, which may negatively impact performance\. steps:  

1. When writing output to a partition at a custom location, tasks write to a file under Spark's staging directory, which is created under the final output location\. The name of the file includes a random UUID to protect against file collisions\. The task attempt keeps track of each file along with the final desired output path\.

1. When a task completes successfully, it provides the driver with the files and their final desired output paths\.

1. After all tasks complete, the job commit phase sequentially renames all files that were written for partitions at custom locations to their final output paths\.

1. The staging directory is deleted before the job commit phase completes\.