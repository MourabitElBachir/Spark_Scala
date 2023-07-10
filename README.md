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
