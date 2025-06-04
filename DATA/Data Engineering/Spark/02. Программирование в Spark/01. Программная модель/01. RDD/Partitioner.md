What is Partitioner?
- Partitioner is an abstract class in Apache Spark that defines the partitioning strategy for distributing data in an RDD based on keys.
- It is used to control how data is spread across partitions, which is critical for optimizing performance in operations like groupByKey, reduceByKey, or join.
- Main goal: Minimize data movement between nodes (shuffle) and ensure even load distribution.

For what is it used?
- Optimization of shuffle: Controls how keys are distributed across partitions, reducing the amount of data moved between nodes.
- Improving data locality: Ensures related data ends up in the same partition for efficient operations.
- Managing parallelism: Allows setting the number of partitions, affecting the degree of parallelism.

Subtypes of Partitioner
1. HashPartitioner:
   - Uses a hash function to distribute keys across partitions.
   - Keys with the same hash code end up in the same partition.
   - Default choice for operations like reduceByKey.
2. RangePartitioner:
   - Partitions keys based on their ranges (e.g., numeric or lexicographical).
   - Suitable for ordered data, ensuring a more even distribution.
   - Often used when keys can be sorted.

RDD methods where Partitioner is used
- partitionBy(partitioner):
  - Applied to an existing RDD to redistribute data according to a new partitioning scheme defined by Partitioner.
- groupByKey(partitioner):
  - Groups data by keys with the option to specify a custom Partitioner.
- reduceByKey(func, partitioner):
  - Performs aggregation by keys with a Partition to control partitions.
- join(otherRDD, partitioner):
  - Performs a join of two RDDs with a specified partitioning strategy.

Examples of code (Python)

Example 1: Using HashPartitioner
```python
from pyspark.sql import SparkSession

# Create SparkSession
spark = SparkSession.builder.appName("HashPartitionerExample").master("local").getOrCreate()
sc = spark.sparkContext

# Create RDD of key-value pairs
data = [(1, "a"), (2, "b"), (1, "c"), (3, "d")]
rdd = sc.parallelize(data)

# Define number of partitions and use HashPartitioner
num_partitions = 2
partitioned_rdd = rdd.partitionBy(num_partitions, partitioner=lambda k: hash(k) % num_partitions)

# Count elements in each partition
partitions = partitioned_rdd.glom().collect()
for i, partition in enumerate(partitions):
    print(f"Partition {i}: {partition}")

# Stop SparkSession
spark.stop()
```
Output: Data will be divided based on hash codes of keys. For example, all entries with key = 1 will end up in one partition.

Example 2: Using RangePartitioner
```python
from pyspark.sql import SparkSession
from pyspark.rdd import RangePartitioner

# Create SparkSession
spark = SparkSession.builder.appName("RangePartitionerExample").master("local").getOrCreate()
sc = spark.sparkContext

# Create RDD of key-value pairs
data = [(1, "a"), (2, "b"), (3, "c"), (4, "d"), (5, "e")]
rdd = sc.parallelize(data)

# Define number of partitions and use RangePartitioner
num_partitions = 2
partitioner = RangePartitioner(num_partitions, rdd)
partitioned_rdd = rdd.partitionBy(num_partitions, partitioner)

# Count elements in each partition
partitions = partitioned_rdd.glom().collect()
for i, partition in enumerate(partitions):
    print(f"Partition {i}: {partition}")

# Stop SparkSession
spark.stop()
```
Output: Keys will be distributed by ranges (e.g., 1-3 in one partition, 4-5 in another), ensuring ordered splitting.

Example 3: Custom Partitioner
```python
from pyspark.sql import SparkSession
from pyspark.rdd import Partitioner

# Custom Partitioner
class CustomPartitioner(Partitioner):
    def __init__(self, num_partitions):
        self.num_partitions = num_partitions

    def numPartitions(self):
        return self.num_partitions

    def getPartition(self, key):
        # Even keys to partition 0, odd keys to partition 1
        return 0 if key % 2 == 0 else 1

# Create SparkSession
spark = SparkSession.builder.appName("CustomPartitionerExample").master("local").getOrCreate()
sc = spark.sparkContext

# Create RDD
data = [(1, "a"), (2, "b"), (3, "c"), (4, "d")]
rdd = sc.parallelize(data)

# Apply custom partitioner
partitioned_rdd = rdd.partitionBy(2, CustomPartitioner(2))

# Count elements in each partition
partitions = partitioned_rdd.glom().collect()
for i, partition in enumerate(partitions):
    print(f"Partition {i}: {partition}")

# Stop SparkSession
spark.stop()
```
Output: Keys 2 and 4 (even) will go to partition 0, keys 1 and 3 (odd) to partition 1.

Conclusion
- Partitioner is critical for managing data distribution in Spark.
- HashPartitioner and RangePartitioner are standard implementations, but a custom Partitioner can be created for specific needs.
- Usage in methods like partitionBy or reduceByKey helps optimize performance.