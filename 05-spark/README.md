# Data Manipulation with Spark

## Introduction

In this workshop we will work with [Apache Spark](https://spark.apache.org/) and implement some basic operations using the Spark DataFrame API for Python. 

We assume that the **Analytics platform** described [here](../01-environment) is running and accessible. 

The same data as in the [Hive Workshop](../04-hive/README.md) will be used. 

##	 Accessing Spark

[Apache Spark](https://spark.apache.org/) is a fast, in-memory data processing engine with elegant and expressive development APIs in Scala, Java, and Python that allow data workers to efficiently execute machine learning algorithms that require fast iterative access to datasets. Spark on Apache Hadoop YARN enables deep integration with Hadoop and other YARN enabled workloads in the enterprise.

You can run batch application such as MapReduce types jobs or iterative algorithms that build upon each other. You can also run interactive queries and process streaming data with your application. Spark also provides a number of libraries which you can easily use to expand beyond the basic Spark capabilities such as Machine Learning algorithms, SQL, streaming, and graph processing. Spark runs on Hadoop clusters such as Hadoop YARN or Apache Mesos, or even in a Standalone Mode with its own scheduler.

There are various ways for accessing Spark

 * **PySpark** - accessing Hive from the commandline
 * **Apache Zeppelin** - a browser based GUI for working with various tools of the Big Data ecosystem
 * **Jupyter** - a browser based GUI for working with a Python and Spark

There is also the option to use **Thrift Server** to execute Spark SQL from any tool supporting SQL. But this is not covered in this workshop.

Before we can use the Spark environment, we need create a folder in HDFS, where the Spark logfiles will be stored 

```
docker exec -ti namenode hadoop fs -mkdir -p /var/log/spark/logs
```

This way can they can also be accessed by the Spark Historyserver.

### Using the Python API through PySpark

The [PySpark API](https://spark.apache.org/docs/latest/api/python/index.html) allows us to work with Spark through the command line. 

In our environment, PySpark is accessible inside the `spark-master` container. To start PySpark use the `pyspark` command. 

```
docker exec -ti spark-master pyspark
```

and you should end up on the **pyspark** command prompt `>>>` as shown below

```
bigdata@bigdata:~$ docker exec -ti spark-master pyspark
WARNING: HADOOP_PREFIX has been replaced by HADOOP_HOME. Using value of HADOOP_PREFIX.
Python 3.7.3 (default, May 19 2019, 19:22:57)
[GCC 6.3.0 20170516] on linux
Type "help", "copyright", "credits" or "license" for more information.
Ivy Default Cache set to: /root/.ivy2/cache
The jars for the packages stored in: /root/.ivy2/jars
:: loading settings :: url = jar:file:/spark/jars/ivy-2.4.0.jar!/org/apache/ivy/core/settings/ivysettings.xml
org.apache.commons#commons-lang3 added as a dependency
:: resolving dependencies :: org.apache.spark#spark-submit-parent-7f79251a-791b-4afd-9ea6-cb401e73ed44;1.0
	confs: [default]
	found org.apache.commons#commons-lang3;3.5 in central
:: resolution report :: resolve 128ms :: artifacts dl 2ms
	:: modules in use:
	org.apache.commons#commons-lang3;3.5 from central in [default]
	---------------------------------------------------------------------
	|                  |            modules            ||   artifacts   |
	|       conf       | number| search|dwnlded|evicted|| number|dwnlded|
	---------------------------------------------------------------------
	|      default     |   1   |   0   |   0   |   0   ||   1   |   0   |
	---------------------------------------------------------------------
:: retrieving :: org.apache.spark#spark-submit-parent-7f79251a-791b-4afd-9ea6-cb401e73ed44
	confs: [default]
	0 artifacts copied, 1 already retrieved (0kB/6ms)
2019-05-20 18:48:15,353 WARN util.NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
Setting default log level to "WARN".
To adjust logging level use sc.setLogLevel(newLevel). For SparkR, use setLogLevel(newLevel).
Welcome to
      ____              __
     / __/__  ___ _____/ /__
    _\ \/ _ \/ _ `/ __/  '_/
   /__ / .__/\_,_/_/ /_/\_\   version 2.4.3
      /_/

Using Python version 3.7.3 (default, May 19 2019 19:22:57)
SparkSession available as 'spark'.
>>>
```

You have an active `SparkSession` available as the `spark` variable. Enter any valid command, just to test we can ask Spark for the version which is installed. 

```
>>> spark.version
'2.4.3'
```

You can use `pyspark` for this workshop. But there are also two other, browser-based tools which are much more comfortable to use and which also allow to store the different steps as a notebook. 

### Using Jupyter

In a browser window navigate to <http://analyticsplatform:38888>. 
Enter `abc123` into the **Password or token** field and click **Log in**. 

You should be forwarded to the **Jupyter** homepage. Click on the **Python 3** icon in the **Notebook** section to create a new notebook using the **Python 3** kernel.

![Alt Image Text](./images/jupyter-create-notebook.png "Jupyter Create Notebook")
  
You will be forwarded to an empty notebook with a first empty cell. Here you can enter your commands. We first have to create Spark Session. Add the following code to the first cell

```
import os
import pyspark
from pyspark.sql import SparkSession

conf = pyspark.SparkConf()

spark = SparkSession.builder.appName('abc').config(conf=conf).getOrCreate()
sc = spark.sparkContext
```

and execute it by entering **Ctrl** + **Enter**. If you check the code you can see that we connect to the Spark Master and get a session on the "spark cluster", available through the `spark` variable. The Spark Context is available as variable `sc`.

First execute `spark.version` in another shell to show the Spark version in place. And then excute a python command `print ("hello")` just to see that you are executing python. 

![Alt Image Text](./images/jupyter-execute-cell.png "Jupyter Execute cell")

You can use Jupyter to perform the workshop. 

## Working with Apache Spark Data Frames

First let's upload the data needed for this workshop, using the techniques we have learned in the [HDFS Workshop](../02-hdfs/README.md).

### Upload Raw Data

In this workshop we are working with truck data. The files are also available in the `data-tranfer` folder of the Analytics Platform: [/01-environment/docker/data-transfer/flightdata](../01-environment/docker/data-transfer/truckdata).

In HDFS under the folder `/user/hue` create a new folder `truckdata` and upload the two files into that folder. 

Here are the commands to perform when using the **Hadoop Filesystem Command** on the commmand line

```
docker exec -ti namenode hadoop fs -mkdir -p /user/hue/truckdata/

docker exec -ti namenode hadoop fs -copyFromLocal /data-transfer/truckdata/geolocation.csv /user/hue/truckdata/
docker exec -ti namenode hadoop fs -copyFromLocal /data-transfer/truckdata/trucks.csv /user/hue/truckdata/

docker exec -ti namenode hadoop fs -ls /user/hue/truckdata/
```

Now with the raw data in HDFS, let's use Spark and the DataFrames API to work with the data. 

### Working with Trucks and Geolocation data using Spark DataFrames

Let’s now start working with the Trucks data, which we have uploaded with the file `trucks.csv`.

#### Working with Truck Data

Now let’s start using some code. First let’s import the spark python API. 

```
from pyspark.sql.types import *
```

Next let’s import the trucks data into a DataFrame and show the first 5 rows. We use header=true to use the header line for naming the columns and specify to infer the schema.  

```
trucksRawDF = spark.read.csv("hdfs://namenode:9000/user/hue/truckdata/trucks.csv", 
    	sep=",", inferSchema="true", header="true")
trucksRawDF.show(5)
```

Let’s display the schema, which has been derived from the data:

```	
trucksRawDF.printSchema()
```

The result should be a rather large schema only shown here partially. You can see that both string as well as integer datatypes have been used and that the names of the columns are derived from the header row of the CSV file. 

```
root
	|-- driverid: string (nullable = true)
	|-- truckid: string (nullable = true)
	|-- model: string (nullable = true)
	|-- jun13_miles: integer (nullable = true)
	|-- jun13_gas: integer (nullable = true)
	|-- may13_miles: integer (nullable = true)
	|-- may13_gas: integer (nullable = true)
	|-- apr13_miles: integer (nullable = true)
	|-- apr13_gas: integer (nullable = true)
	...
	...
```
	
Next let’s ask for the total number of rows in the dataset. Should return 100. 

```
trucksRawDF.count()
```
	
You can also transform data easily into another format, just by writing the DataFrame out to a new file. Let’s create a JSON representation of the data. We will write it to a refined folder. 

```
trucksRawDF.write.json("hdfs://namenode:9000/user/hue/truckdata/truck.json")
```
	
Should you want to execute it a 2nd time, then you first have to delete the folder truck-json, otherwise the 2nd execution will throw an error.

```
hadoop fs -rm -R hdfs://namenode:9000/user/hue/truckdata/truck.json
```
	
By now we have imported the truck data and made it available as the `trucksRawDF` Data Frame. We will use that data frame again later. 

Additionally we have also stored the data to a file in json format. 

Now let’s do the same for the geolocation data. 

#### Working with the geolocation data

Again use some markdown to show that we are now working with geolocation data

```
%md ## Import Geolocation data
```
	
Then read the data from the `geolocation.csv` file into another DataFrame.

```
geolocationRawDF = spark.read.csv("hdfs://namenode:9000/user/hue/truckdata/geolocation.csv", 
	    sep=",", inferSchema="true", header="true")
geolocationRawDF.show(5)
```

And show the schema, which has been inferred.

```
geolocationRawDF.printSchema()
```

Let's also print the number of rows in the DataFrame. This should return 8000.

```
geolocationRawDF.count()
```

Now we hold the geolocation data in another dataframe in memory. Let's see how we can further make use of these dataframes.

### Working with the data using Spark SQL

Now we also have the geolocation available in the DataFrame. Let’s work with it using SQL. 

Add some markdown to start the new section

```
## Let's use some SQL to work with the data
```

To use the data in a SQL statement, we can register the DataFrame as a temporary view. 

```
trucksRawDF.createOrReplaceTempView("trucks")
geolocationRawDF.createOrReplaceTempView("geolocation")
```

Temporary views in Spark SQL are session-scoped and will disappear if the session that creates it terminates. If you want to have a temporary view that is shared among all sessions and keep alive until the Spark application terminates, you can create a global temporary view. So instead of the above, you could also do

```
trucksRawDF.createGlobalTempView("trucks")
geolocationRawDF.createGlobalTempView("geolocation")
```

Global temporary view is tied to a system preserved database `global_temp`, and we must use the qualified name to refer it, e.g. SELECT * FROM `global_temp.geolocation`.


We can always ask for the table registered by executing the show tables SQL command.

```
spark.sql("show tables").show()

spark.sql("SELECT * FROM trucks").show()
```

Play with some different statements on `geolocation`. Let's start with just showing the data

```
spark.sql("SELECT * FROM geolocation").show()
```

```
unsafeDrivingDF = spark.sql("""
	                SELECT driverid, COUNT(*) occurance 
	                FROM geolocation WHERE event != 'normal'
	                GROUP BY driverid
	                ORDER BY COUNT(*) DESC
	                        """)
unsafeDrivingDF.show()
```

Using triple double-quotes allows us to specify the SQL over multiple lines, which allow a copy-paste from the version tested above.

Again register the result as a temporary view named `unsafe_driving`

```
unsafeDrivingDF.createOrReplaceTempView("unsafe_driving")
```

### Transform the data using Spark SQL

Now let’s use that technique to do some restructuring (transformation) of the data. 

Again let’s name that section of the notebook, using some markdown. 

```
## Restructure the Trucks data
```

We will use the same statement as with the Hive workshop to unpivot the data, so that the different values by month are no longer in one result line per driver but on separate result lines. 

```
truckMileageDF = spark.sql("""
SELECT truckid, driverid, rdate, miles, gas, miles / gas mpg 
FROM trucks 
LATERAL VIEW stack(54, 
				'jun13',jun13_miles,jun13_gas,
				'may13',may13_miles,may13_gas,
				'apr13',apr13_miles,apr13_gas,
				'mar13',mar13_miles,mar13_gas,
				'feb13',feb13_miles,feb13_gas,
				'jan13',jan13_miles,jan13_gas,
				'dec12',dec12_miles,dec12_gas,
				'nov12',nov12_miles,nov12_gas,
				'oct12',oct12_miles,oct12_gas,
				'sep12',sep12_miles,sep12_gas,
				'aug12',aug12_miles,aug12_gas,
				'jul12',jul12_miles,jul12_gas,
				'jun12',jun12_miles,jun12_gas,
				'may12',may12_miles,may12_gas,
				'apr12',apr12_miles,apr12_gas,
				'mar12',mar12_miles,mar12_gas,
				'feb12',feb12_miles,feb12_gas,
				'jan12',jan12_miles,jan12_gas,
				'dec11',dec11_miles,dec11_gas,
				'nov11',nov11_miles,nov11_gas,
				'oct11',oct11_miles,oct11_gas,
				'sep11',sep11_miles,sep11_gas,
				'aug11',aug11_miles,aug11_gas,
				'jul11',jul11_miles,jul11_gas,
				'jun11',jun11_miles,jun11_gas,
				'may11',may11_miles,may11_gas,
				'apr11',apr11_miles,apr11_gas,
				'mar11',mar11_miles,mar11_gas,
				'feb11',feb11_miles,feb11_gas,
				'jan11',jan11_miles,jan11_gas,
				'dec10',dec10_miles,dec10_gas,
				'nov10',nov10_miles,nov10_gas,
				'oct10',oct10_miles,oct10_gas,
				'sep10',sep10_miles,sep10_gas,
				'aug10',aug10_miles,aug10_gas,
				'jul10',jul10_miles,jul10_gas,
				'jun10',jun10_miles,jun10_gas,
				'may10',may10_miles,may10_gas,
				'apr10',apr10_miles,apr10_gas,
				'mar10',mar10_miles,mar10_gas,
				'feb10',feb10_miles,feb10_gas,
				'jan10',jan10_miles,jan10_gas,
				'dec09',dec09_miles,dec09_gas,
				'nov09',nov09_miles,nov09_gas,
				'oct09',oct09_miles,oct09_gas,
				'sep09',sep09_miles,sep09_gas,
				'aug09',aug09_miles,aug09_gas,
				'jul09',jul09_miles,jul09_gas,
				'jun09',jun09_miles,jun09_gas,
				'may09',may09_miles,may09_gas,
				'apr09',apr09_miles,apr09_gas,
				'mar09',mar09_miles,mar09_gas,
				'feb09',feb09_miles,feb09_gas,
				'jan09',jan09_miles,jan09_gas ) dummyalias AS rdate, miles, gas
	""")
```

With such a large SQL statement, the tripple-quotes are really helpful!

Let’s print the schema of the new DataFrame.

```
truckMileageDF.printSchema()
```

We can also work on the DataFrame in a fluent-API style, for example to only show data of a given driver. This is just the programmatic version as an alternative for using SQL.

```
truckMileageDF.filter(truckMileageDF.driverid=="A3").show(10)
```

We have successfully transformed the truck data and made it available as a DataFrame in memory. If we want to persist it so that it is available for other consumers as well, we can write it to HDFS. 

### Write the result data to HDFS

Again let’s name that section of the notebook, using some markdown. 

```
## Persist result data in HDFS
```

Let’s write the `truckMileageDF` DataFrame to HDFS. We could again use CSV or JSON to do that. 

But there are more efficient serialisation formats, such as **Parquet**, **ORC** or **Avro**. These have a lot of advantages if the file is later processed further.  

#### Using JSON

First we will see how we can save the result as a **JSON** formatted file. 

```
truckMileageDF.write.json('hdfs://namenode:9000/user/hue/truckdata/truckmileage-json')
```

and then check that the file has been written using the `hadoop fs` command

```sh
hadoop fs -ls -h hdfs://namenode:9000/user/hue/truckdata/truckmileage-parquet
```	

#### Using Parquet

Next let's save the result to a **Parquet** formatted file. We use a similar statement as above when we wrote JSON, just using the `parquet()` method instead.

```
truckMileageDF.write.parquet('hdfs://namenode:9000/user/hue/truckdata/truckmileage-parquet')
```

Let's see it has worked with another `hadoop fs` command

```
hadoop fs -ls -h hdfs://namenode:9000/user/hue/truckdata/truckmileage-parquet
```	
	
#### Using Apache ORC

To use an **ORC** formatted file, we use a similar statement as above but instead of the `parquet()` method we use the generic `format()` together with the `save()` method.

```
truckMileageDF.write.format("orc").save('hdfs://namenode:9000/user/hue/truckdata/truckmileage-orc')
```

Again let's see it it has worked with the `hadoop fs` command

```
hadoop fs -ls -h hdfs://namenode:9000/user/hue/truckdata/truckmileage-orc
```	

#### Using Apache Avro

We can also tryout the **Avro** format in a similar way

```
truckMileageDF.write.format("avro").save('hdfs://namenode:9000/user/hue/truckdata/truckmileage-avro')
```

Before executing, you also have to add the dependency `org.apache.spark:spark-avro_2.11:2.4.3` to the Spark interpreter.

Unfortunately that **does not currently** work due to [SPARK-26675](https://issues.apache.org/jira/browse/SPARK-26675?page=com.atlassian.jira.plugin.system.issuetabpanels%3Aall-tabpanel).

### Simple Analytics on the data using Spark SQL

Now let’s do some simple analytics on the data, both using SQL and the fluent-API. 

Again start with a new section my using some markdown. 

```
## Apply some analytics on the data
```

Now register the truck mileage DataFrame as a table 

```
truckMileageDF.createOrReplaceTempView("truck_mileage")
```	

Calculate and display the average value of mpg by truck. Let’s first do it with SQL

```
spark.sql("""SELECT truckid, avg(mpg) avgmpg
	FROM truck_mileage
	GROUP BY truckid""").show()
```

We can do the same in a programmatic way using the fluent API

```
avgMpgByTruck = truckMileageDF.groupBy("truckId").agg({"mpg":"avg"})
avgMpgByTruck.show()
```
