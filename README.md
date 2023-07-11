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

