## Apache Spark 

The reason is that Hadoop framework is based on a simple programming model (MapReduce),Spark was introduced by Apache Software Foundation for speeding up the Hadoop computational computing software process. As against a common belief, Spark is not a modified version of Hadoop and is not, really, dependent on Hadoop because it has its own cluster management. Hadoop is just one of the ways to implement Spark.

Spark uses Hadoop in two ways – one is storage and second is processing. Since Spark has its own cluster management computation, it uses Hadoop for storage purpose only. The main feature of Spark is its in-memory cluster computing that increases the processing speed of an application.

Spark is designed to cover a wide range of workloads such as batch applications, iterative algorithms, interactive queries and streaming. There are three ways of Spark deployment as explained below.

========
1. Standalone :- Spark Standalone deployment means Spark occupies the place on top of HDFS(Hadoop Distributed File System) and space is allocated for HDFS, explicitly. Here, Spark and MapReduce will run side by side to cover all spark jobs on cluster.

2. Hadoop Yarn :- Hadoop Yarn deployment means, simply, spark runs on Yarn without any pre-installation or root access required. It helps to integrate Spark into Hadoop ecosystem or Hadoop stack. It allows other components to run on top of stack.

3. Spark in MapReduce (SIMR) :- Spark in MapReduce is used to launch spark job in addition to standalone deployment. With SIMR, user can start Spark and uses its shell without any administrative access.


#### Here are some JARGONS from Apache Spark :-

Job:- A piece of code which reads some input  from HDFS or local, performs some computation on the data and writes some output data.

Stages:-Jobs are divided into stages. Stages are classified as a Map or reduce stages(Its easier to understand if you have worked on Hadoop and want to correlate). Stages are divided based on computational boundaries, all computations(operators) cannot be Updated in a single Stage. It happens over many stages.

Tasks:- Each stage has some tasks, one task per partition. One task is executed on one partition of data on one executor(machine).

DAG - DAG stands for Directed Acyclic Graph, in the present context its a DAG of operators.

Executor - The process responsible for executing a task.

Driver - The program/process responsible for running the Job over the Spark Engine

Master - The machine on which the Driver program runs

Slave - The machine on which the Executor program runs



Referance : https://www.edureka.co/blog/spark-tutorial/?utm_campaign=social-media-edureka-november-sd&utm_medium=crosspost&utm_source=quora
