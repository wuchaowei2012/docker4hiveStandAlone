
export PYSPARK_PYTHON=python3
export PYSPARK_DRIVER_PYTHON=ipython
export SPARK_HOME=/root/Fred_wu/spark-2.4.5-bin-hadoop2.6
export HADOOP_HOME=/root/Fred_wu/hadoop-2.7.7
export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64


# 0 启动 docker 
## Running standalone Hive Metastore (with Mysql)

export HIVE_METASTORE_JDBC_URL=jdbc:postgresql://postgres.docker.starburstdata.com/hivemetastore
export HIVE_METASTORE_DRIVER="org.postgresql.Driver"
export HIVE_METASTORE_USER=hivemetastore
export HIVE_METASTORE_PASSWORD=hivemetastore

cd /root/Fred_wu/docker-images-master/hive-metastore-unsecured && docker-compose up metastore mysql

## Running standalone Hive Metastore (with Postgres)

export HIVE_METASTORE_JDBC_URL=jdbc:postgresql://postgres.docker.starburstdata.com/hivemetastore
export HIVE_METASTORE_DRIVER="org.postgresql.Driver"
export HIVE_METASTORE_USER=hivemetastore
export HIVE_METASTORE_PASSWORD=hivemetastore
cd /root/Fred_wu/docker-images-master/hive-metastore-unsecured && docker-compose up metastore postgres

# 1 设置到 container 端口转发
启动 metastore 服务

cd /root/Fred_wu/docker-images-master/hive-metastore-unsecured && docker-compose up metastore

也可以使用 hive container 进行操作。 (过于复杂)

# 2 开始 spark thrift服务

$SPARK_HOME/sbin/start-thriftserver.sh –driver-java-options "Dhive.metastore.uris=thrift://localhost:9083"

$SPARK_HOME/sbin/start-thriftserver.sh --executor-cores 4 --total-executor-cores 4 --master local[4]


# 3 开启 hive 支持
/root/Fred_wu/pyspark4externaldb/pyspark_start.sh

from pyspark.sql import SparkSession

spark = SparkSession \
    .builder \
    .appName("Python Spark SQL Hive integration example") \
    .config("spark.master", "local"). \
    config("hive.metastore.uris","thrift://10.200.12.134:9083"). \
    config("spark.sql.warehouse.dir",  "/root/Fred_wu/warehouse"). \
    enableHiveSupport() \
    .getOrCreate()

spark.sql("show databases").show()


# 4 Stopping the Hive Metastore and clean data
docker-compose down  (小心使用)
docker-compose down --volumes  (小心使用)


# 5 hive cli
While Metastore is started you can start standalone Hive CLI with:

cd /root/Fred_wu/docker-images-master/hive-metastore-unsecured && docker-compose exec metastore hive



# 建立表
create table test_data(a bigint) 
date DATE,
PARTITIONED BY (date)
stored as parquet; 
