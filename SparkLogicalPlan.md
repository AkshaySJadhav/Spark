### Understanding Spark’s Logical and Physical Plan in layman’s term 


First, let’s see what Apache Spark is. The official definition of Apache Spark says that “Apache Spark™ is a unified analytics engine for large-scale data processing.” It is an in-memory computation processing engine where the data is kept in random access memory (RAM) instead of some slow disk drives and is processed in parallel.

Now, let’s look at the high-level steps which are involved while we work with respect to Apache Spark:

- We begin by writing the code. This code can be DataFrame, DataSet or a SQL and then we submit it.
- If the code is valid, Spark will convert it into a Logical Plan.
    Further, Spark will pass the Logical Plan to a Catalyst Optimizer.
    In the next step, the Physical Plan is generated (after it has passed through the Catalyst Optimizer), this is where the majority of our optimizations are going to happen.
    Once the above steps are complete, Spark executes/processes the Physical Plan and does all the computation to get the output. 

These are the 5 steps at the high-level which Spark follows. Now let’s break down each step into detail. The DRIVER (Master Node) is responsible for the generation of the Logical and Physical Plan.

#### Query Plan

#### Logical Plan:

Let’s say we have a code (DataFrame, DataSet, SQL). Now the first step will be the generation of the Logical Plan. Logical Plan is divided into three parts:

1. Unresolved Logical Plan OR Parsed Logical Plan
2. Resolved Logical Plan OR Analyzed Logical Plan OR Logical Plan
3. Optimized Logical Plan 

Now the question is what is a Logical Plan?

Logical Plan is an abstract of all transformation steps that need to be performed and it does not refer anything about the Driver (Master Node) or Executor (Worker Node). The SparkContext is responsible for generating and storing it. This helps us to convert the user expression into the most optimized version.

Let’s suppose you have written the code as –

val file = spark.sparkContext.textFile(“…”)

SparkContext is going to convert it into its own internal representation which Spark can understand. But, the first step which happens is the generation of Unresolved Logical Plan.

It may so happen that our code is valid and the syntax is also correct, but the column name or the table name may be inaccurate or may not even exist. That’s why we call it an Unresolved Logical Plan.

Basically, what happens here is that, when we give a wrong column name, our Unresolved Logical Plan is still created. This is the first step where Spark has created a blank Logical Plan where there are no checks for the column name, table name, etc.

Further, Spark is going to use a component called “Catalog” which is a repository where all the information about Spark table, DataFrame, DataSet will be present. The data from meta-store is pulled into the Catalog which is an internal storage component of Spark. There is yet another component named “Analyzer” which helps us to resolve/verify the semantics, column name, table name by cross-checking from the Catalog. And that’s why I say that DataFrame/DataSet follows Semi-lazy evaluation because, at the time of the creation of the Logical Plan, it starts performing analysis without an Action.

For example:
dataFrame.select(“age”) //Column “age” may not even exist.

dataFrame.select(max(“name”)) //Checks if column “name” is a number

If the Analyzer is not able to resolve them (column name, table name, etc), it can reject our Unresolved Logical Plan. But if it gets resolved then it creates Resolved Logical Plan. 


SQL to Resolved Logical Plan :

After, the Resolved Logical plan is generated it is then passed on to a “Catalyst Optimizer” which will apply its own rule and will try to optimize the plan. Basically, Catalyst Optimizer performs logical optimization. 
For example, 

1. It checks for all the tasks which can be performed and computed together in one Stage. 
2. In a multi-join query, it decides the order of execution of query for better performance.
3. Tries to optimize the query by evaluating the filter clause before any project. This, in turn, generates an Optimized Logical Plan.


####  Physical Plan:

We can also create our own customized Catalyst Optimizer according to our business use case by defining/applying specific rules to it to perform custom optimization. Now coming to Physical Plan, it is an internal optimization for Spark. Once our Optimized Logical Plan is created then further, Physical Plan is generated.

What does this Physical Plan do?

It simply specifies how our Logical Plan is going to be executed on the cluster. It generates different kinds of execution strategies and then keeps comparing them in the “Cost Model”. For example, it estimates the execution time and resources to be taken by each strategy. Finally, whichever plan/strategy is going to be the best optimal one is selected as the “Best Physical Plan” or “Selected Physical Plan”. 

What actually happens in Physical Plan?

Let’s say, we are performing a join query between two tables. In that join operation, we may have a fat table and a thin table with a different number of partitions scattered in different nodes across the cluster (same rack or different rack). Spark decides which partitions should be joined first (basically it decides the order of joining the partitions), the type of join, etc for better optimization.

Physical Plan is specific to Spark operation and for this, it will do a check-up of multiple physical plans and decide the best optimal physical plan. And finally, the Best Physical Plan runs in our cluster.

Once the Best Physical Plan is selected, it’s the time to generate the executable code (DAG of RDDs) for the query that is to be executed in a cluster in a distributed fashion. This process is called Codegen and that’s the job of Spark’s Tungsten Execution Engine.

Conclusion:

Logical Plan just depicts what I expect as output after applying a series of transformations like join, filter, where, groupBy, etc clause on a particular table.

Physical Plan is responsible for deciding the type of join, the sequence of the execution of filter, where, groupBy clause, etc.

This is how SPARK SQL works internally…!!!

References:
[1]. https://blog.knoldus.com/understanding-sparks-logical-and-physical-plan-in-laymans-term/
