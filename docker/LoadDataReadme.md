# Steps to load data to iceberg

Run the container
```
docker exec -it spark-thrift bash
```
```
/usr/spark/bin/spark-shell --conf "spark.serializer=org.apache.spark.serializer.KryoSerializer" --conf "spark.sql.extensions=org.apache.iceberg.spark.extensions.IcebergSparkSessionExtensions" --conf "spark.sql.catalog.iceberg_catalog=org.apache.iceberg.spark.SparkCatalog" --conf "spark.sql.catalog.iceberg_catalog.type=hive" --conf "spark.sql.catalog.iceberg_catalog.uri=thrift://hive-metastore:9083" --conf "spark.hadoop.fs.s3a.aws.credentials.provider=org.apache.hadoop.fs.s3a.SimpleAWSCredentialsProvider" --conf "spark.hadoop.fs.s3a.impl=org.apache.hadoop.fs.s3a.S3AFileSystem" --conf "spark.hadoop.fs.defaultFS=s3a://pai-datavault-dev-datalake" --conf "spark.sql.session.timeZone=UTC" --conf "spark.hadoop.fs.s3a.access.key=minioadmin" --conf "spark.hadoop.fs.s3a.secret.key=minioadmin" --conf "spark.hadoop.fs.s3a.endpoint=http://minio:9000"
``` 
```
spark.sql("CREATE SCHEMA iceberg_catalog.vault_sdm LOCATION 's3a://vault-datalake/iceberg/vault_sdm/'")
spark.read.parquet("file:///data/EGS/asset_host_list").writeTo("iceberg_catalog.vault_sdm.asset_host_list").create()
spark.read.parquet("file:///data/EGS/host_vulnerability").writeTo("iceberg_catalog.vault_sdm.host_vulnerability").create()
spark.read.parquet("file:///data/EGS/knowledgebase").writeTo("iceberg_catalog.vault_sdm.knowledge_base").create()
spark.read.parquet("file:///data/EGS/host_summary").writeTo("iceberg_catalog.vault_sdm.host_summary").create()
spark.sql("SHOW TABLES FROM vault_sdm").show()

```

# profiles.yml file template
```
entity-inventory-datavault:
  outputs:
    dev:
      host: "{{ env_var('SPARK_THRIFT_HOST') }}"
      method: thrift
      port: 10000
      schema: "{{ env_var('DDM_SPARK_SCHEMA') }}"
      threads: 10
      type: spark
      auth: NOSASL
  target: dev
}
```
# Run the dbt Incremental materialization model
```
docker run -e SPARK_THRIFT_HOST=spark-thrift -e DDM_SPARK_SCHEMA=vault_test --network=docker_dbt_sample_network --mount type=bind,source=${dbt project path},target=/usr/app --mount type=bind,source=${dbt project path},target=/root/.dbt/ ghcr.io/dbt-labs/dbt-spark run --vars '{"schema_name": "vault_test","schema":"s3a://vault-datalake/iceberg/vault_test/"}'
```

# To access beeline
```
docker exec -it spark-thrift /usr/spark/bin/beeline -u "jdbc:hive2://spark-thrift:10000/;auth=noSasl" -n root
```