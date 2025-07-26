In this blog post, I have explained why we should not use **inferSchema = true**. It means we are telling spark to infer
schema automatically.

The schema means here are the **column types**. A column can be of type **String, Double, Long, etc**.

If the schema is not specified using **schema** function and **inferSchema** option is enabled, this function goes
through the **input once to determine the input schema.** It means it **takes some time to infer a schema**.

If the schema is not specified using **schema** function and **inferSchema** option is disabled, then **it determines the
columns as string types**, and it reads only the first line to determine the names and the number of fields.

In the below examples, I have explained how much time it takes to infer a schema and with the same action.

*time* function calculates how much time a block of code takes to execute.

```scala
def time[A](name: String)(body: => A) = {
  val start = System.currentTimeMillis()
  body
  val end = System.currentTimeMillis()
  println(s"$name Took ${end - start} millis")
}
```

**src/main/resources/engineer.csv**

```
name,department,years_of_experience,dob
Abdhesh,Software Engineer,8,1990-07-20
Shikha,Fullstak Developer,9,1989-07-02
```

**1- Using inferSchema = true :**

```scala
case class Developer(
                      name: String,
                      department: String,
                      years_of_experience: Int,
                      dob: Timestamp
                    )

time("inferSchema = true") {
  val developerDF = spark.read
    .option("inferSchema", "true")
    .option("header", "true")
    .option("timestampFormat", "yyyy-MM-dd")
    .csv("src/main/resources/engineer.csv")
  import spark.implicits._
  val developerDS = developerDF.as[Developer]
  developerDS.collect().toList.foreach(println)
}
```  

Output:

```
Developer(Abdhesh,Software Engineer,8,1990-07-20 00:00:00.0)
Developer(Shikha,Fullstak Developer,9,1989-07-02 00:00:00.0)
inferSchema = true Took 18040 millis
```

**2- Using explicit schema: (inferSchema = false)**

```scala
time("inferSchema = false") {
  val schema = StructType(
    List(
      StructField("name", StringType, false),
      StructField("department", StringType, false),
      StructField("years_of_experience", IntegerType, false),
      StructField("dob", TimestampType, false)
    )
  )
  val developerDF = spark.read
    .option("header", "true")
    .schema(schema)
    .csv("src/main/resources/engineer.csv")
  import spark.implicits._
  val developerDS = developerDF.as[Developer]
  developerDS.collect().toList.foreach(println)
}
```  

Output:

```
Developer(Abdhesh,Software Engineer,8,1990-07-20 00:00:00.0)
Developer(Shikha,Full Stack Developer,9,1989-07-02 00:00:00.0)
inferSchema = false Took 718 millis
```

If you do not want to define schema explicit, then you can derive schema from an encoder.

```scala
time("inferSchema = false, derive schema from an encoder") {
  implicit val encoderDeveloper: Encoder[Developer] = Encoders.product[Developer]
  val developerDF = spark.read
    .option("header", "true")
    .schema(encoderDeveloper.schema)
    .csv("src/main/resources/engineer.csv")
  val developerDS = developerDF.as[Developer]
  developerDS.collect().toList.foreach(println)
}
```  

Output:

```
Developer(Abdhesh,Software Engineer,8,1990-07-20 00:00:00.0)
Developer(Shikha,Full Stack Developer,9,1989-07-02 00:00:00.0)
inferSchema = false, derive schema from an encoder Took 388 millis
```

**3- Infer schema dynamically**

Dynamically, we can infer the schema from the first row of the CSV(after the header row) and set while reading full CSV.
It is the best trick to get schema dynamically if you do not know the schema of CSV.

```scala
time("Infer schema from first row") {
  val developerDF1RowSchema = spark.read
    .option("header", "true")
    .option("inferSchema", "true")
    .option("timestampFormat", "yyyy-MM-dd")
    .csv("src/main/resources/engineer.csv")
    .head()
    .schema

  val developerDF = spark.read
    .option("header", "true")
    .option("timestampFormat", "yyyy-MM-dd")
    .schema(developerDF1RowSchema)
    .csv("src/main/resources/engineer.csv")

  import spark.implicits._
  val developerDS = developerDF.as[Developer]
  developerDS.collect().toList.foreach(println)
}
```

Output:

```
Developer(Abdhesh,Software Engineer,8,1990-07-20 00:00:00.0)
Developer(Shikha,Full Stack Developer,9,1989-07-02 00:00:00.0)
Infer schema from first row Took 3570 millis
```

Now If you compare between approach 1st and 2nd, the processing time is dropped **~97%**. I just have two records in my
CSV file. Think about, if you have a huge CSV file, then you could get better performance by defining schema explicitly.
So **you should never ever use inferSchema = true.** If you want, you can get a code from
my <a href="https://github.com/abdheshkumar/spark-practices/blob/master/src/main/scala/InferSchema.scala" target="_blank" aria-label="undefined (opens in a new tab)" rel="noreferrer noopener">
Github</a> repository.

Happy coding 🙂