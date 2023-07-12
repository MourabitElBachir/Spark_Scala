# Setting Up Apache Spark with Scala in Jupyter Notebook Using Spylon

This README provides a step-by-step guide to set up Apache Spark with Scala in a Jupyter Notebook using Spylon.

## Prerequisites

- Java 8 or later
- Python 3.5 or later
- Jupyter 4.2 or later
- Apache Spark 2.0.0 or later

## Installation

1. **Install Apache Spark**

    If you don't have Apache Spark installed, you can install it using pip:

    ```bash
    pip install pyspark
    ```

2. **Install Spylon-kernel**

    Spylon-kernel is a Jupyter kernel for Scala with Spark magic. You can install it with pip:

    ```bash
    pip install spylon-kernel
    ```

3. **Add the Spylon Kernel to Jupyter**

    Once the Spylon-kernel is installed, you need to install the kernel into Jupyter:

    ```bash
    python -m spylon_kernel install
    ```

4. **Restart Jupyter**

    Restart Jupyter to recognize the new kernel.

## Usage

When starting a new notebook, select the Spylon kernel (Scala) as your programming environment.

Initialize Spark using the `%%init_spark` magic command:

```scala
%%init_spark
launcher.master = "local[*]"
launcher.conf.spark.app.name = "My Spark App"
launcher.conf.spark.executor.memory = "4g"
```
In this configuration, Spark is running in local mode using all available cores (indicated by the asterisk in "local[*]"). The Spark application name is set to "My Spark App", and the executor memory is set to 4 GB.

You have to execute the %%init_spark command at the beginning of the notebook before executing any other cell that uses Spark.

After initializing Spark, you can use it just like you would in a regular Scala application. For example:

```scala
val sparkDF = spark.read.json("examples/src/main/resources/people.json")
sparkDF.show()
```
<b>Note:</b> The pip install ... commands are to be run in the terminal. If you're installing these packages system-wide or in a Python environment, you may want to run pip install ... commands directly in the terminal without the preceding exclamation mark.


### Cache en Spark 

```scala
import org.apache.spark.sql.functions._
import spark.implicits._

// Define a large DataFrame
val df = spark.range(0, 10000000).toDF("id")

// Apply some transformation
val transformedDf = df.withColumn("new_column", expr("id * 5"))

// Cache the DataFrame
transformedDf.cache()

// Start the timer
val startTimeWithCache = System.nanoTime

// Perform an action to trigger the computations
transformedDf.count()

// End the timer
val endTimeWithCache = System.nanoTime

val timeWithCache = (endTimeWithCache - startTimeWithCache) / 1e9d
println(s"Time with cache: $timeWithCache seconds")

transformedDf.unpersist()

val startTimeWithoutCache = System.nanoTime

transformedDf.count()

val endTimeWithoutCache = System.nanoTime

val timeWithoutCache = (endTimeWithoutCache - startTimeWithoutCache) / 1e9d
println(s"Time without cache: $timeWithoutCache seconds")
```
```scala
data.persist(StorageLevel.MEMORY_ONLY) // Store DataFrame partitions in memory only === data.cache()

data.persist(StorageLevel.MEMORY_AND_DISK) // Store DataFrame partitions in memory and spill to disk if necessary

data.persist(StorageLevel.MEMORY_ONLY_SER) // Store DataFrame partitions in memory after serializing them

data.persist(StorageLevel.MEMORY_AND_DISK_SER) // Store DataFrame partitions in memory after serializing them, spill to disk if necessary

data.persist(StorageLevel.DISK_ONLY) // Store DataFrame partitions on disk only

data.cache() // Equivalent to MEMORY_ONLY storage level, cache DataFrame partitions in memory
```

## Create a project with modules using build.sbt 

### Structure du Projet
```ruby
mon-projet-spark
|-- build.sbt
|-- project
|   |-- build.properties
|-- module1
|   |-- src
|   |   |-- main
|   |   |   |-- scala
|   |   |   |   |-- Module1.scala
|-- module2
|   |-- src
|   |   |-- main
|   |   |   |-- scala
|   |   |   |   |-- Module2.scala

```

### Configuration du Projet

Le fichier `build.sbt` racine devrait ressembler à ceci:

```sbt
lazy val root = (project in file("."))
  .aggregate(module1, module2)
  .settings(
    name := "mon-projet-spark",
    version := "0.1",
    scalaVersion := "2.12.10"
  )

lazy val module1 = (project in file("module1"))
  .settings(
    name := "module1",
    version := "0.1",
    scalaVersion := "2.12.10",
    libraryDependencies += "org.apache.spark" %% "spark-core" % "3.0.1",
    libraryDependencies += "org.apache.spark" %% "spark-sql" % "3.0.1"
  )

lazy val module2 = (project in file("module2"))
  .settings(
    name := "module2",
    version := "0.1",
    scalaVersion := "2.12.10",
    libraryDependencies += "org.apache.spark" %% "spark-core" % "3.0.1",
    libraryDependencies += "org.apache.spark" %% "spark-sql" % "3.0.1"
  )
```

Et le fichier project/build.properties :

```sbt
sbt.version = 1.4.1
```

### Compilation et Empaquetage du Projet
Pour compiler le projet entier, naviguez vers le répertoire racine et exécutez:
```sbt
sbt compile
```
Pour empaqueter le projet en un fichier JAR, exécutez :
```sbt
sbt package
```
Cela va créer un fichier JAR pour chaque module dans le répertoire target/scala-2.12 de chaque module.

### Creation automatique des sources :

```sbt
lazy val createDirectories = taskKey[Unit]("Create src and test directories")

createDirectories := {
  val s: TaskStreams = streams.value
  val baseDirs = Seq((Compile, baseDirectory.value / "src", baseDirectory.value / "test"),
    (Test, baseDirectory.value / "src" / "test", baseDirectory.value / "test" / "test"))
  baseDirs.foreach {
    case (conf, srcDir, testDir) => {
      Seq("main", "test").foreach(d => {
        if (!Files.exists(srcDir.toPath.resolve(d))) {
          s.log.info(s"Creating source directory: ${srcDir / d}")
          IO.createDirectory(srcDir / d)
        }
        if (!Files.exists(testDir.toPath.resolve(d))) {
          s.log.info(s"Creating test directory: ${testDir / d}")
          IO.createDirectory(testDir / d)
        }
      })
    }
  }
}
// Call the task when SBT loads
onLoad in Global := ((s: State) => "createDirectories" :: s.get(onLoad in Global).getOrElse(identity[State] _)(s))
```
## Installer Spark & Hadoop localement sur Windows

1. Download Hadoop : https://github.com/steveloughran/winutils/

2. Choisir Hadoop 2.7.1 :
![image](https://github.com/MourabitElBachir/Spark_Scala/assets/32568108/f2fba0ef-5e4a-4a75-b262-b6d1d80ed98a)

3. Download Spark à partir des archives : https://archive.apache.org/dist/spark/spark-3.0.1/

Il faut choisir : spark-3.0.1-bin-hadoop2.7.tgz

![image](https://github.com/MourabitElBachir/Spark_Scala/assets/32568108/81cc0c47-e9d1-4547-a60d-91a3d630f647)

4. Déplacer les dossier décompresser vers C:/
   
6. Mettre les chemins dans le Path :
   ![image](https://github.com/MourabitElBachir/Spark_Scala/assets/32568108/861fb57c-71a7-4e05-8080-713782d02b68)
   ![image](https://github.com/MourabitElBachir/Spark_Scala/assets/32568108/b6c9cdcf-fa17-4c2a-9d42-4add0c3d42f8)
   ![image](https://github.com/MourabitElBachir/Spark_Scala/assets/32568108/0ee717e0-4991-4440-838b-4424f08fbf49)

## Spark Streaming :

1- Code scala/Spark :

```
import org.apache.spark._
import org.apache.spark.streaming._
import org.apache.spark.streaming.flume._

object StreamingFlumeLogAggregator {
  def main(args: Array[String]) {
    val conf = new SparkConf().setAppName("StreamingFlumeLogAggregator")
    val ssc = new StreamingContext(conf, Seconds(1))

    val flumeStream = FlumeUtils.createStream(ssc, "localhost", 9092)

    val lines = flumeStream.map(e => new String(e.event.getBody.array()))

    val pattern = """(?x)^(\S+) \S+ (\S+) \[([\w:/]+\s[+\-]\d{4})\] "(\S+) (\S+)\s*(\S*)" (\d{3}) (\S+) "(\S+)" "(.+)"$""".r

    val urls = lines.flatMap {
      case pattern(_, _, _, _, url, _, _, _, _, _) => Some(url)
      case _ => None
    }

    val urlCounts = urls.map(url => (url, 1))
      .reduceByKeyAndWindow(_ + _, _ - _, Minutes(5), Seconds(1))

    val sortedResults = urlCounts.transform(rdd => rdd.sortBy(x => x._2, false))

    sortedResults.print()

    ssc.checkpoint("/home/maria_dev/checkpoint")
    ssc.start()
    ssc.awaitTermination()
  }
}
```







