**1. <a href="http://www.oracle.com/technetwork/java/javase/downloads/index.html" target="_blank" rel="noopener">Download and Install Java 8</a>**

```text
abdhesh@abdhesh-latitude:~/Documents/Applications$ wget https://download.oracle.com/otn-pub/java/jdk/8u151-b12/e758a0de34e24606bca991d704f6dcbf/jdk-8u151-linux-x64.tar.gz
```


Extract tar file:

```text
abdhesh@abdhesh-latitude:~/Documents/Applications$ tar -xf jdk-8u151-linux-x64.tar.gz 
abdhesh@abdhesh-latitude:~/Documents/Applications$ ls
jdk-8u151-linux-x64  jdk-8u151-linux-x64.tar.gz
```

Set environment path variable for Java

```text
abdhesh@abdhesh-latitude:~/Documents/Applications$ sudo vim ~/.bashrc
```

The Above command will open a file, and you need to add below lines at the end of the file.

```text
export JAVA=/home/abdhesh/Documents/Applications/jdk-8u151-linux-x64
export PATH=$JAVA/bin:$PATH
```

Save and exit. Now reload a **.bashrc** file on same terminal&#8217;s session

```text
abdhesh@abdhesh-latitude:~/Documents/Applications$ source ~/.bashrc 
```

Run java version command:

```text
abdhesh@abdhesh-latitude:~/Documents/Applications$ java -version
java version "1.8.0_151"
Java(TM) SE Runtime Environment (build 1.8.0_151-b12)
Java HotSpot(TM) 64-Bit Server VM (build 25.151-b12, mixed mode)
```

**2. <a href="https://www.scala-lang.org/download/" target="_blank" rel="noopener">Download and Install Scala</a>**

```text
abdhesh@abdhesh-latitude:~/Documents/Applications$ wget https://downloads.lightbend.com/scala/2.12.4/scala-2.12.4.tgz
```

Extract tar file:

```text
abdhesh@abdhesh-latitude:~/Documents/Applications$ tar -xf scala-2.12.4.tgz 
abdhesh@abdhesh-latitude:~/Documents/Applications$ ls
scala-2.12.4  scala-2.12.4.tgz
```

Set environment path variable for scala

```
abdhesh@abdhesh-latitude:~/Documents/Applications$ sudo vim ~/.bashrc
export SCALA=/home/abdhesh/Documents/Applications/scala-2.12.4
export PATH=$JAVA/bin:$SCALA/bin:$PATH
```

Save and exit. Now reload a **.bashrc** file on same terminal&#8217;s session

```
abdhesh@abdhesh-latitude:~/Documents/Applications$ source ~/.bashrc 
```

Run scala version command:

```
abdhesh@abdhesh-latitude:~/Documents/Applications$ scala -version
Scala code runner version 2.12.4 -- Copyright 2002-2017, LAMP/EPFL and Lightbend, Inc.
```

**3. <a href="https://spark.apache.org/downloads.html" target="_blank" rel="noopener">Download and Install Apache Spark</a>**

```
abdhesh@abdhesh-latitude:~/Documents/Applications$ wget https://apache.mirror.anlx.net/spark/spark-2.2.1/spark-2.2.1-bin-hadoop2.7.tgz
```

Extract tar file:

```
abdhesh@abdhesh-latitude:~/Documents/Applications$ tar -xf spark-2.2.1-bin-hadoop2.7.tgz 
abdhesh@abdhesh-latitude:~/Documents/Applications$ ls
spark-2.2.1-bin-hadoop2.7  spark-2.2.1-bin-hadoop2.7.tgz
```

Set environment path variable for spark

```
abdhesh@abdhesh-latitude:~/Documents/Applications$ sudo vim ~/.bashrc
```

The above command will open a file, and you need to add below lines at the end of the file.

```
export SPARK=/home/abdhesh/Documents/Applications/spark-2.2.1-bin-hadoop2.7
export PATH=$JAVA/bin:$SCALA/bin:$SPARK/bin:$PATH
```

Now reload **.bashrc** file on same terminal&#8217;s session

```
abdhesh@abdhesh-latitude:~/Documents/Applications$ source ~/.bashrc 
```

Run Spark shell:

Here is link of <a href="https://github.com/apache/spark/blob/master/examples/src/main/resources/people.json" target="_blank" rel="noopener">people.json file</a>

```
abdhesh@abdhesh-latitude:~/Documents/Applications$ spark-shell 
Using Spark's default log4j profile: org/apache/spark/log4j-defaults.properties
Setting default log level to "WARN".
To adjust logging level use sc.setLogLevel(newLevel). For SparkR, use setLogLevel(newLevel).
17/12/28 01:02:16 WARN NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
17/12/28 01:02:17 WARN Utils: Your hostname, abdhesh-latitude resolves to a loopback address: 127.0.1.1; using 192.168.0.16 instead (on interface wlp2s0)
17/12/28 01:02:17 WARN Utils: Set SPARK_LOCAL_IP if you need to bind to another address
Spark context Web UI available at http://192.168.0.16:4040
Spark context available as 'sc' (master = local[*], app id = local-1514422939241).
Spark session available as 'spark'.
Welcome to
      ____              __
     / __/__  ___ _____/ /__
    _\ \/ _ \/ _ `/ __/  '_/
   /___/ .__/\_,_/_/ /_/\_\   version 2.2.1
      /_/
         
Using Scala version 2.11.8 (Java HotSpot(TM) 64-Bit Server VM, Java 1.8.0_151)
Type in expressions to have them evaluated.
Type :help for more information.

scala> 
scala> val df  = spark.read.json("spark-2.2.1-bin-hadoop2.7/data/people.json")
df: org.apache.spark.sql.DataFrame = [age: bigint, name: string]

scala&gt; df.filter("age &gt;= 19").select("name","age").show()
+------+---+
|  name|age|
+------+---+
|  Andy| 30|
|Justin| 19|
+------+---+


scala> //using Sql

scala> df.createOrReplaceTempView("people")

scala> spark.sql("SELECT * FROM people WHERE age &gt;=19").show()
+---+------+
|age|  name|
+---+------+
| 30|  Andy|
| 19|Justin|
+---+------+


scala> 
```

Stay tuned for next blog post 🙂