---
layout: ../../layouts/MarkdownPostLayout.astro
title: '用spark实现的数据分析'
pubDate: 2023-12-30
description: '最近做的作业'
author: 'liyishui'
image:
    url: 'https://pic.imgdb.cn/item/659128b9c458853aef90fedb.jpg'
    alt: '最近看的一部电影《末路狂花》.'
tags: ["Spark"]
---

### 用户分析
```
// 导入SparkSession和相关的类
import org.apache.spark.sql.SparkSession
import org.apache.spark.sql.functions._
import org.apache.spark.sql.types._


// 创建SparkSession
val spark = SparkSession.builder().appName("YourAppName").getOrCreate()

val myManualSchema = StructType(Array(
	StructField("Date",StringType),
	StructField("UserID",StringType),
	StructField("Item",StringType),
	StructField("location",StringType),
	StructField("FinalEnd",StringType)
	))

val df = spark.read.format("csv").schema(myManualSchema).option("delimiter","\t").load("file:///home/liyishui/UserlogsHot.log")

val click_count = df.groupBy("Date","UserID","Item").count()

click_count.show()

import java.io.PrintWriter
val click_count_json = click_count.toJSON
val json_writer = new PrintWriter("/home/liyishui/click.json")
click_count_json.collect().foreach(json_writer.println)
json_writer.close()

import org.apache.spark.sql.expressions.Window
val windows = Window.partitionBy("UserID","Item").orderBy(col("count").desc)

val result = click_count.withColumn("rank",row_number().over(windows)).filter(col("rank")<=3).select("Date","UserID","Item","count").toJSON.collect()

result.foreach(println)
```

### NBA分析
```
// 导入SparkSession和相关的类
import org.apache.spark.sql.SparkSession
import org.apache.spark.sql.functions._
import org.apache.spark.sql.types._


// 创建SparkSession
val spark = SparkSession.builder().appName("YourAppName").getOrCreate()

val df1= spark.read.format("csv").option("encoding","gbk").option("head","true").load("file:///home/liyishui/NBA14-16.csv").toDF("球队","赛季","投篮","投篮命中","投篮出手","三分","三分命中","三分出手","罚球","罚球命中","罚球出手","篮板","前场","后场","助攻","抢断","盖帽","失误","犯规","得分","失分","胜","负","公式","联盟")

val eightFore= df1.filter(col("赛季")==="15-16").orderBy(col("得分").desc).limit(8)

//eightFore.show()

val score1415 = df1.filter(col("赛季")==="14-15").select("球队","胜")

val score1516 = df1.filter(col("赛季")==="15-16").select(col("球队"),col("胜").alias("胜_1"))

val total = score1415.join(score1516,score1415.col("球队")===score1516.col("球队")).withColumn("RankChange",col("胜")-col("胜_1"))

total.orderBy(col("RankChange").desc).show()

val season1516Data = df1.filter(col("赛季") === "15-16")

val topFieldGoalTeam = season1516Data.orderBy(col("投篮命中").desc).limit(3)

val topThreePointTeam = season1516Data.orderBy(col("三分命中").desc).limit(3)

val topDefenseTeam = season1516Data.orderBy(col("失分").asc).limit(3)

topFieldGoalTeam.show()
topThreePointTeam.show()
topDefenseTeam.show()
```

### 泰坦尼克号生还情况分析
```
// 导入SparkSession和相关的类
import org.apache.spark.sql.SparkSession
import org.apache.spark.sql.functions._
import org.apache.spark.sql.types._
import spark.implicits._

import org.apache.spark.sql.{SparkSession, DataFrame}
import org.apache.spark.sql.types.{IntegerType, DoubleType}

/*
val spark = SparkSession.builder().appName("YourAppName").getOrCreate()

val dfclass = spark.read.option("header","true").csv("file:///home/liyishui/class.csv")
*/

val spark = SparkSession.builder().appName("YourAppName").getOrCreate()

val df = spark.read.option("header","true").csv("file:///home/liyishui/titanic.csv")

df.withColumn("Pclass",df("Pclass").cast(IntegerType))
      .withColumn("Survived",df("Survived").cast(IntegerType))
      .withColumn("Age",df("Age").cast(DoubleType))
      .withColumn("SibSp",df("SibSp").cast(IntegerType))
      .withColumn("Parch",df("Parch").cast(IntegerType))
      .withColumn("Fare",df("Fare").cast(DoubleType))
      
val df1=df.drop("PassengerId").drop("Name").drop("Ticket").drop("Cabin")

val columns=df1.columns
val missing_cnt=columns.map(x=>df1.select(col(x)).where(col(x).isNull).count)
val result_cnt=sc.parallelize(missing_cnt.zip(columns)).toDF("missing_cnt","column_name")
result_cnt.show()

def meanAge(dataFrame: DataFrame): Double = {
dataFrame
.select("Age")
.na.drop()
.agg(round(mean("Age"), 0))
.first()
.getDouble(0)
}

val df2= df1
.na.fill(Map(
"Age" -> meanAge(df1),
"Embarked" -> "S"))

df2.show()

val survived_count=df2.groupBy("Survived").count()
survived_count.show()
survived_count.coalesce(1).write.option("header", "true").csv("/home/liyishui/survived_count.csv")

val survived_embark=df2.groupBy("Embarked","Survived").count()
survived_embark.show()
survived_embark.coalesce(1).write.option("header", "true").csv("/home/liyishui/survived_embark.csv")


val survived_sex_count=df2.groupBy("Sex","Survived").count()
val survived_sex_percent=survived_sex_count.withColumn("percent",format_number(col("count").divide(sum("count").over()).multiply(100),5));
survived_sex_percent.show()
survived_sex_percent.coalesce(1).write.option("header", "true").csv("/home/liyishui/survived_sex_percent.csv")


val survived_df = df2.filter(col("Survived")===1)
val pclass_survived_count=survived_df.groupBy("Pclass").count()
val pclass_survived_percent=pclass_survived_count.withColumn("percent",format_number(col("count").divide(sum("count").over()).multiply(100),5));
pclass_survived_percent.show()
pclass_survived_percent.coalesce(1).write.option("header", "true").csv("/home/liyishui/pclass_survived_percent.csv")

val df4=df2.withColumn("Parch_label",when(df2("Parch")>0,1).otherwise(0))
val parch_survived_count=df4.groupBy("Parch_label","Survived").count()
parch_survived_count.show()
parch_survived_count.coalesce(1).write.option("header", "true").csv("/home/liyishui/parch_survived_count.csv")

val df3=survived_df.withColumn("Age_label",when(df2("Age")<=18,"minor").when(df2("Age")>18 && df2("Age")<=35,"young").when(df2("Age")>35 && df2("Age")<=55,"middle").otherwise("older"))
val age_survived=df3.groupBy("Age_label","Survived").count()
age_survived.show()
age_survived.coalesce(1).write.option("header", "true").csv("/home/liyishui/age_survived.csv")

val sef = Seq("Pclass", "Fare")
val df5 = df2.select(sef.head, sef.tail: _*)
df5.show(5)
df5.coalesce(1).write.option("header", "true").csv("/home/liyishui/pclass_fare.csv")

```