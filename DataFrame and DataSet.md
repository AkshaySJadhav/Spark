## Datasets and DataFrames

A Dataset is a distributed collection of data. Dataset is a new interface added in Spark 1.6 that provides the benefits of RDDs (strong typing, ability to use 
powerful lambda functions) with the benefits of Spark SQL’s optimized execution engine. A Dataset can be constructed from JVM objects and then manipulated using 
functional transformations (map, flatMap, filter, etc.). The Dataset API is available in Scala and Java. Python does not have the support for the Dataset API. But 
due to Python’s dynamic nature, many of the benefits of the Dataset API are already available (i.e. you can access the field of a row by name naturally 
row.columnName). The case for R is similar.

A DataFrame is a Dataset organized into named columns. It is conceptually equivalent to a table in a relational database or a data frame in R/Python, but with 
richer optimizations under the hood. DataFrames can be constructed from a wide array of sources such as: structured data files, tables in Hive, external databases, 
or existing RDDs. The DataFrame API is available in Scala, Java, Python, and R. In Scala and Java, a DataFrame is represented by a Dataset of Rows. In the Scala 
API, DataFrame is simply a type alias of Dataset[Row]. While, in Java API, users need to use Dataset<Row> to represent a DataFrame.

Spark 1.3 introduced two new data abstraction APIs – DataFrame and DataSet. The DataFrame APIs organizes the data into named columns like a table in relational 
database. It enables programmers to define schema on a distributed collection of data. Each row in a DataFrame is of object type row. Like an SQL table, each 
column must have same number of rows in a DataFrame. In short, DataFrame is lazily evaluated plan which specifies the operations needs to be performed on the 
distributed collection of the data. DataFrame is also an immutable collection.

Throughout this document, we will often refer to Scala/Java Datasets of Rows as DataFrames.


- DataFrame 

is defined well with a google search for "DataFrame definition":

A data frame is a table, or two-dimensional array-like structure, in which each column contains measurements on one variable, and each row contains one case. So, a 
DataFrame has additional metadata due to its tabular format, which allows Spark to run certain optimizations on the finalized query.

An RDD, on the other hand, is merely a Resilient Distributed Dataset that is more of a blackbox of data that cannot be optimized as the operations that can be 
performed against it, are not as constrained. However, you can go from a DataFrame to an RDD via its rdd method, and you can go from an RDD to a DataFrame 
(if the RDD is in a tabular format) via the toDF method. In general it is recommended to use a DataFrame where possible due to the built in query optimization.
