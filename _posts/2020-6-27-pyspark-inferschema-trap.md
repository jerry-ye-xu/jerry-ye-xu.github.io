---
layout: post
title: PySpark data trap - inferschema
category: Practical
categories: ['PySpark','data wrangling']
tags: ['Python','PySpark', 'data wrangling']
---

---
<h2 class="no_toc">Table of Contents</h2>

* TOC
{:toc}

## PySpark

PySpark is the Python API wrapper for Apache Spark, a big data processing framework.

Previously I used Spark in an assignment and my team mates had slightly different results to mine when we were trying to solve the same question.

It turns out, it came down to the `inferschema` parameter when reading in a  Spark DataFrame.

## The Inferschema Trap

Essentially what happens when you set `inferschema=True` is that PySpark tries to infer the data type, and this can result in a discrepancy when you're filtering.

When a column is a string type, the `F.between()` function is inclusive.

However, when the data type is `timestamp` it is no longer inclusive.

## Code Example

First let's try and load the data into a

```{Python}
spark = SparkSession.builder.appName("delayed_flights").getOrCreate()

data_path = <path-to-data>

df = spark.read.format("csv") \
        .options(
            header="true",
            inferschema="true"
        ) \
        .load(data_path)


df.printSchema()

# Now let's filter by the dates...

flights_tiny_df = flights_tiny_df.filter(
    flights_tiny_df["flight_date"].between(
        "1996-01-01",
        "1996-12-31"
    )
)

flight_cols = ["tail_number", "flight_date"]
flight_dates = flights_tiny_df.select(flight_cols) \
    .groupBy("flight_date") \
    .count()

# Get the latest date
print(flight_dates.orderBy("flight_date", ascending=False) \
        .collect()[0]
    )
```
Filtering the dates gives
The `printSchema()` function returns
```
> root
> |-- flight_id: integer (nullable = true)
> |-- carrier_code: string (nullable = true)
> |-- flight_number: integer (nullable = true)
> |-- flight_date: timestamp (nullable = true)
> |-- origin: string (nullable = true)
> |-- destination: string (nullable = true)
> |-- tail_number: string (nullable = true)
> |-- scheduled_depature_time: string (nullable = true)
> |-- scheduled_arrival_time: string (nullable = true)
> |-- actual_departure_time: string (nullable = true)
> |-- actual_arrival_time: string (nullable = true)
> |-- distance: integer (nullable = true)
```
Filtering the dates gives
```
> Row(flight_date=datetime.datetime(1996, 12, 30, 0, 0), count=145)
```

Now if you change `inferschema=False` and run the exact same code again, you will get2
```
> root
> |-- flight_id: string (nullable = true)
> |-- carrier_code: string (nullable = true)
> |-- flight_number: string (nullable = true)
> |-- flight_date: string (nullable = true)
> |-- origin: string (nullable = true)
> |-- destination: string (nullable = true)
> |-- tail_number: string (nullable = true)
> |-- scheduled_depature_time: string (nullable = true)
> |-- scheduled_arrival_time: string (nullable = true)
> |-- actual_departure_time: string (nullable = true)
> |-- actual_arrival_time: string (nullable = true)
> |-- distance: string (nullable = true)
```
and the result
```{bash}
> Row(flight_date='1996-12-31', count=129)
```

This took me longer than necessary to debug because I did not suspect this to be an issue for a while.

Nevertheless I hope this means that you won't run into this issue in your own work!

You download the dataset [here](https://www.dropbox.com/s/a41ov7uqwlnnon8/ontimeperformance_flights_tiny.csv?dl=0) to reproduce the code.
