+++
title = "Writing Unit Tests for Spark Apps in Scala"
type = "docs"
bookToC = false
+++

Often, something you’d like to test when you’re writing self-contained Spark applications, is whether your given work on a DataFrame or Dataset will return what you want it to after multiple joins and manipulations to the input data.

This is not different from traditional unit testing, with the only exception that you'd like to test and introspect not only the functionality of the code [but the data itself.](https://www.youtube.com/watch?v=yACtdj1_IxE) 

There’s two ways to be defensive about creating correct data inputs and outputs in Scala Spark. The first by writing and reading from Datasets, which are strongly-typed collections of objects. This works well if you know exactly the data structures you’d like to write.

If you’re more in experimental mode, another way to check your data is to write unit tests against Spark code that you can run both locally and as part of CI/CD when you merge your Spark jobs into prod. 

## Scala Unit Tests

First, a word about unit tests. In Scala, with the Scalatest suite, you can use either traditional [TDD unit tests with FunSuite](https://www.scalatest.org/user_guide/selecting_a_style), or FlatSpec, which is [more behavior-driven](https://www.scalatest.org/scaladoc/3.2.10/org/scalatest/flatspec/AnyFlatSpec.html). Flatspec gives you acess to matchers, which are a scala-based DSL of [custom assertions.](https://www.scalatest.org/user_guide/using_matchers). Scalatest leans towards FlatSpec as the default testing capability in any given Scala project, but you can use either style. 

I've seen a mix of different styles for Spark, but most of them follow FunSuite, including [Spark Test Base](https://github.com/holdenk/spark-testing-base), [Spark Fast Tests](https://github.com/MrPowers/spark-fast-tests/tree/20b1b5f4574a63c8c8007b0f77a94b11e7156b08), and this [Spark unit testing example library](https://github.com/tmalaska/SparkUnitTestingExamples) from a previous [Spark Summit.](https://www.youtube.com/watch?v=4U9Me6shpno)

I've also chosen to follow FunSuite for this example because I'm more familiar with traditional unit testing, and because you can implement much of the functionality, including FlatSpec's matchers, in FunSuite directly. 

Here's an example of a simple test you can set up against a DataFrame:

```scala
import org.apache.spark.sql.catalyst.expressions.AttributeSet.empty.intersect
import org.apache.spark.sql.{DataFrame, Dataset, Encoders, Row, SQLContext, SQLImplicits, SparkSession}
import org.apache.spark.{SparkConf, SparkContext}
import org.scalatest.{BeforeAndAfterAll, BeforeAndAfterEach, FunSuite}

final class YourTestpec extends FunSuite with BeforeAndAfterEach with BeforeAndAfterAll  {self =>
  @transient var ss: SparkSession = null
  @transient var sc: SparkContext = null

  private object testImplicits extends SQLImplicits {
    protected override def _sqlContext: SQLContext = self.ss.sqlContext
  }
  import testImplicits._

  override def beforeAll(): Unit = {
    val sparkConfig = new SparkConf()
    sparkConfig.set("spark.broadcast.compress", "false")
    sparkConfig.set("spark.shuffle.compress", "false")
    sparkConfig.set("spark.shuffle.spill.compress", "false")
    sparkConfig.set("spark.master", "local")

    ss = SparkSession.builder().config(sparkConfig).getOrCreate()
  }

  override def afterAll(): Unit = {
    ss.stop()
  }


test("simple dataframe assert") {

	val df = spark.createDataFrame(Seq((1,"a string","another string",12344567L).toDF("first val","stringval","stringval2","longnum")

    assert(df.count == 1)

 }
```

Note that here I'm setting up a Spark session and context beforehand so that when I run `sbt test`, I'm actually running it locally on my machine against the version of Spark that comes bundled with my project. This makes testing quicker than having to ship your project to wherever it runs against your data remotely. 

Another fantastic alternative is using the [Spark Test Base](https://github.com/holdenk/spark-testing-base), which has methods for both DataFrames and Datasets and even sets up a SparkContext for you: 

```scala
import org.apache.spark.sql.catalyst.expressions.AttributeSet.empty.intersect
import org.apache.spark.sql.{DataFrame, Dataset, Encoders, Row, SQLContext, SQLImplicits, SparkSession}
import org.apache.spark.{SparkConf, SparkContext}
import org.scalatest.{BeforeAndAfterAll, BeforeAndAfterEach, FunSuite}
import com.holdenkarau.spark.testing.{DatasetGeneratorRDDGenerator, SharedSparkContext
}

final class YourTestpec extends FunSuite with DataFrameSuiteBase  with SharedSparkContext with DatasetGenerator{

    override def beforeAll(): Unit = {
    val sparkConfig = new SparkConf()
    sparkConfig.set("spark.broadcast.compress", "false")
    sparkConfig.set("spark.shuffle.compress", "false")
    sparkConfig.set("spark.shuffle.spill.compress", "false")
    sparkConfig.set("spark.master", "local")

    ss = SparkSession.builder().config(sparkConfig).getOrCreate()
  }

  override def afterAll(): Unit = {
    ss.stop()
  }


test("simple dataframe assert") {

	val df = spark.createDataFrame(Seq((1,"a string","another string",12344567L).toDF("first val","stringval","stringval2","longnum")

    val df2 = spark.createDataFrame(Seq((1,"a string","another string",12344567L).toDF("first val","stringval","stringval2","longnum")


    assertDataFrameEquals(df, df2) 

 }
```


Both Spark Test Base and Fast Tests work well for most of what you'd like to test in Spark, such as checking column equality, schemas, totals, and values, and asserting DataFrame equality, which is what I was looking for. 

## Heard you like Arrays of Arrays

Sometimes, however, you need to test more complex data structures. For example, often for machine learning, particularly for text processing, you need to create nesting where you might have a DataFrame or Dataset, where the output looks something like: 


| user      | text_feature_1                 | text_feature_2           |
|-----------|--------------------------------|--------------------------|
| 123456789 | Array("text","text","text")    | Map("val"->2, "val2"->3) |
| 21234234  | Array("text1","text1","text2") | Map("val"->4, "val2"->5) |


How do you test for equality between what you expect and what you output here if you're looking to test the entire method generating this DataFrame? 

The main problem here is in the way Scala does object comparison. Scala by default, in its object comparison methods such as `equals`, `sameElements`, and even `deep`, checks for referential equality, whether these objects are exactly the same object. deepEquals only works on [arrays](https://docs.oracle.com/javase/8/docs/api/java/util/Arrays.html#deepEquals-java.lang.Object:A-java.lang.Object:A-). 

That means that if your columns are creating complex objects like Maps, they'll never be equal, since, when you create two DataFrames, even if they're equivalent, they're made up of Row objects, which are made up of Maps and Arrays that are each unique new instantiated objects. 

In this case, you need a way of traversing the data structure. 

Here's what I came up with for my test: 


```scala
test("test my join") {
 
    // Create our test data
    val test = spark.DataFrame = ss.createDataFrame(Seq(
      (12345678901L,Array("text","text","text"),Map("text"->1 , "text"->2 , "text" ->2)))
      .toDF("user_id",
        "my_array",
        "my_map")
 
val expected: DataFrame = spark.createDataFrame(Seq(
      (12345678901L,Array("tag","tag","tag"),Map("tag"->1 , "tag"->2 , "tag" ->2)))
      .toDF("user_id",
        "my_array",
        "my_map")
 
// Create our test data
    val expectedArr = test.collect()
    val testArr = expected.collect()
 
// zip into a collection that compares across tuples of elements
    (expectedArr zip testArr).foreach{
      case (a,b) => assert(dsEqual(a,b))
    }
 
  }
 
 
  def dsEqual(a:MyCaseClassRecord, b:MyCaseClassRecord): Boolean ={
    a.user_id == b.user_id &&
      sameArray(a.my_array, b.my_array) &&
      sameMap(a.my_map,b.my_map)
  }
 
// compare Arrays with nesting
  def sameArray(a:Array[String], b:Array[String]) : Boolean ={
    a == b || a.sameElements(b)
  }
 
// compare Maps with nesting
  def sameMap(a:Map[String,Int], b:Map[String,Int]) : Boolean ={
    a == b || a.sameElements(b)
  }

```


