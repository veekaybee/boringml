+++
title = "Sample data in PySpark"
type = "docs"
bookToC = false
+++

Here's how to create a small fake dataset for testing in PySpark. More on [sc.parallelize](https://spark.apache.org/docs/2.1.1/programming-guide.html#parallelized-collections). 

```python
from pyspark.sql.session import SparkSession
rdd = sc.parallelize([(0,None), (0,1), (0,2), (1,2), (1,10), (1,20), (3,18), (3,18), (3,18)])
df=rdd.toDF(['id','score'])
df.show()
```
```
+---+-----+
| id|score|
+---+-----+
|  0| null|
|  0|    1|
|  0|    2|
|  1|    2|
|  1|   10|
|  1|   20|
|  3|   18|
|  3|   18|
|  3|   18|
+---+-----+
```

```
df.printSchema()
root
 |-- id: long (nullable = true)
 |-- score: long (nullable = true)

```

None is a special keyword in Python that will let you create nullable fields. 
If you want to simulate NaN fields, you can do `float('nan')` for the value.
Note that if you don't specify each field as float, you get a null result for the values that are not typed. 

```python
from pyspark.sql.session import SparkSession
import numpy as np
rdd = sc.parallelize([(0,np.nan), (0,float(1)), (0,float(2)), (1,float(2)), (1,float(10)), (1,float(20)), (3,float(18)), (3,float(18)), (3,18)])
df=rdd.toDF(['id','score'])
df.show()
```

```
+---+-----+
| id|score|
+---+-----+
|  0|  NaN|
|  0|  1.0|
|  0|  2.0|
|  1|  2.0|
|  1| 10.0|
|  1| 20.0|
|  3| 18.0|
|  3| 18.0|
|  3| null|
+---+-----+
```
