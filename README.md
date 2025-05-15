# Apache Spark Connect with S3 Tables (Managed Iceberg)

This README provides step-by-step instructions on how to set up and use Apache Spark Connect with S3 Tables using Apache Iceberg.

## Prerequisites

- Java 11 or higher installed
- AWS account with appropriate permissions
- AWS credentials configured in your environment

## Setup Instructions

### 1. Download and Extract Apache Spark

```bash
# Download Spark
wget https://www.apache.org/dyn/closer.lua/spark/spark-3.5.5/spark-3.5.5-bin-hadoop3.tgz

# Extract the archive
tar -xvf spark-3.5.5-bin-hadoop3.tgz

# Navigate to the Spark directory
cd spark-3.5.5-bin-hadoop3
```

### 2. Start the Spark Connect Server

Set up the required environment variables and start the Spark Connect server:

```bash
# Set Java home (modify this to match your Java installation path)
export JAVA_HOME='/opt/homebrew/opt/openjdk@11'

# Set network binding options for local development
export SPARK_SUBMIT_OPTS="-Dspark.driver.bindAddress=127.0.0.1 -Dspark.driver.host=127.0.0.1"

# Start the Connect server with required packages and configurations
./sbin/start-connect-server.sh \
  --packages "org.apache.spark:spark-connect_2.12:3.5.5,org.apache.iceberg:iceberg-spark-runtime-3.5_2.12:1.6.1,software.amazon.s3tables:s3-tables-catalog-for-iceberg:0.1.3,software.amazon.awssdk:bundle:2.29.38,org.apache.hadoop:hadoop-aws:3.3.4,com.github.ben-manes.caffeine:caffeine:3.1.8,org.apache.commons:commons-configuration2:2.11.0"
```

### 3. Connect Using PySpark Client

Open a new terminal window and navigate to your Spark directory:

```bash
cd /Users/soumilshah/Desktop/project/

# Set Java home
export JAVA_HOME='/opt/homebrew/opt/openjdk@11'

# Start PySpark with remote connection to the Spark Connect server
./bin/pyspark --remote "sc://localhost"
```

### 4. Verify the Connection with a Simple DataFrame

Run the following in the PySpark shell to verify your connection:

```python
from pyspark.sql import SparkSession
spark = SparkSession.builder.appName("SimpleDF").getOrCreate()

# Create a simple DataFrame
columns = ["id", "name"]
data = [(1, "Sarah"), (2, "Maria")]
df = spark.createDataFrame(data).toDF(*columns)
df.show()
```

### 5. Configure S3 Tables with Iceberg

Configure your Spark session to work with S3 Tables and Iceberg:

```python
from pyspark.sql import SparkSession

spark = SparkSession.builder \
    .appName("iceberg_s3tables") \
    .config("spark.sql.extensions", "org.apache.iceberg.spark.extensions.IcebergSparkSessionExtensions") \
    .config("spark.sql.catalog.s3tablesbucket", "org.apache.iceberg.spark.SparkCatalog") \
    .config("spark.sql.catalog.s3tablesbucket.catalog-impl", "software.amazon.s3tables.iceberg.S3TablesCatalog") \
    .config("spark.sql.catalog.s3tablesbucket.warehouse", "arn:aws:s3tables:us-east-1:867098943567:bucket/soumilshah-dev") \
    .config("spark.sql.catalog.s3tablesbucket.client.region", "us-east-1") \
    .getOrCreate()
```

### 6. Query S3 Tables

Explore your S3 Tables:

```python
# List all namespaces in the catalog
spark.sql("SHOW NAMESPACES IN s3tablesbucket").show()

# List all tables in the s3tablescatalog namespace
spark.sql("SHOW TABLES IN s3tablesbucket.s3tablescatalog").show()

# Query the customers table
spark.sql("SELECT * FROM s3tablesbucket.s3tablescatalog.customers").show()
```

### 7. Stop the Spark Connect Server

When you're done, stop the Spark Connect server:

```bash
./sbin/stop-connect-server.sh
```

## Common Issues and Troubleshooting

- If you encounter connection issues, ensure the server is running and check the server logs
- AWS permissions problems can cause failures when accessing S3 Tables
- Verify your AWS credentials are properly configured
- Check for version compatibility between Spark, Iceberg, and the S3 Tables catalog

## Additional Resources

- [Apache Spark Documentation](https://spark.apache.org/docs/latest/)
- [Apache Iceberg Documentation](https://iceberg.apache.org/)
- [AWS S3 Tables Documentation](https://docs.aws.amazon.com/s3/latest/userguide/s3-tables.html)
