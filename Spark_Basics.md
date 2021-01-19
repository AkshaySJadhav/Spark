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

Executor - A process launched for an application on a worker node, that runs tasks and keeps data in memory or disk storage across them. Each application has its own executors.

Driver - The process running the main() function of the application and creating the SparkContext

Deploy Mode : Distinguishes where the driver process runs. In "cluster" mode, the framework launches the driver inside of the cluster. In "client" mode, the submitter launches the driver outside of the cluster.

Master - The machine on which the Driver program runs

Slave - The machine on which the Executor program runs

### Spark Executor, Drivers and LINEAGE.

1. DRIVER

Driver is a Java process. This is the process where the main() method of our Scala, Java, Python program runs. It executes the user code and creates a SparkSession or SparkContext and the SparkSession is responsible to create DataFrame, DataSet, RDD, execute SQL, perform Transformation & Action, etc.

 Responsibility of DRIVER :-

- The main() method of our program runs in the Driver process. It creates SparkSession or SparkContext.
- Conversion of the user code into Task (transformation and action). It looks at the user code and determines are the possible Tasks, i.e. the number of tasks to be performed is decided by the Driver. But how does it determines the Tasks?
– With the help of Lineage. (Discussed later in the blog)
- Helps to create the Lineage, Logical Plan and Physical Plan.
- Wondering what they are? Check my blog by clicking here.
- Once the Physical Plan is generated, the Driver schedules the execution of the tasks by coordinating with the Cluster Manager.
- Coordinates with all the Executors for the execution of Tasks. It looks at the current set of Executors and schedules our tasks.
- Keeps track of the data (in the form of metadata) which was cached (persisted) in Executor’s (worker’s) memory.

2. EXECUTOR

Executor resides in the Worker node. Executors are launched at the start of a Spark Application in coordination with the Cluster Manager. They are dynamically launched and removed by the Driver as per required.

Responsibility of EXECUTOR :-

- To run an individual Task and return the result to the Driver.
- It can cache (persist) the data in the Worker node.

Thinking how these Driver and Executor Processes are launched after submitting a job (spark-submit)?
Well, then let’s talk about the Cluster Manager.

Spark is dependent on the Cluster Manager to launch the Executors and also the Driver (in Cluster mode).We can use any of the Cluster Manager (as mentioned above) with Spark i.e. Spark can be run with any of the Cluster Manager.

```spark-submit –master <Spark master URL> –executor-memory 2g –executor-cores 4 WordCount-assembly-1.0.jar```

- Let’s say a user submits a job using “spark-submit”.
- “spark-submit” will in-turn launch the Driver which will execute the main() method of our code.
- Driver contacts the cluster manager and requests for resources to launch the Executors.
- The cluster manager launches the Executors on behalf of the Driver.
- Once the Executors are launched, they establish a direct connection with the Driver.
- The driver determines the total number of Tasks by checking the Lineage.
- The driver creates the Logical and Physical Plan.
- Once the Physical Plan is generated, Spark allocates the Tasks to the Executors.
- Task runs on Executor and each Task upon completion returns the result to the Driver.
- Finally, when all Task is completed, the main() method running in the Driver exits, i.e. main() method invokes sparkContext.stop().
- Finally, Spark releases all the resources from the Cluster Manager.

3. LINEAGE

When a new RDD is derived from an existing RDD using transformation, that new RDD contains a pointer to the parent RDD and Spark keeps track of all the dependencies between these RDDs using a component called the Lineage. In case of data loss, this lineage is used to rebuild the data. DataFrame, DataSet, SQL are internally converted to RDDs for computation as RDDs are the lowest level of abstraction in Spark. So, all the transformations that are involved internally in a DataFrame, DataSet, SQL can be seen by converting them to RDD.
 

Spark provides a script named “spark-submit” which helps us to connect with a different kind of Cluster Manager and it controls the number of resources the application is going to get i.e. it decides the number of Executors to be launched, how much CPU and memory should be allocated for each Executor, etc.



### Cluster Mode Overview

Spark applications run as independent sets of processes on a cluster, coordinated by the SparkContext object in your main program (called the driver program).

Specifically, to run on a cluster, the SparkContext can connect to several types of cluster managers (either Spark’s own standalone cluster manager, Mesos or YARN), which allocate resources across applications. Once connected, Spark acquires executors on nodes in the cluster, which are processes that run computations and store data for your application. Next, it sends your application code (defined by JAR or Python files passed to SparkContext) to the executors. Finally, SparkContext sends tasks to the executors to run.


##### Cluster Manager Types

There are several useful things to note about this architecture:

1. Each application gets its own executor processes, which stay up for the duration of the whole application and run tasks in multiple threads. This has the benefit of isolating applications from each other, on both the scheduling side (each driver schedules its own tasks) and executor side (tasks from different applications run in different JVMs). However, it also means that data cannot be shared across different Spark applications (instances of SparkContext) without writing it to an external storage system.
2. Spark is agnostic to the underlying cluster manager. As long as it can acquire executor processes, and these communicate with each other, it is relatively easy to run it even on a cluster manager that also supports other applications (e.g. Mesos/YARN).
3. The driver program must listen for and accept incoming connections from its executors throughout its lifetime (e.g., see spark.driver.port in the network config section). As such, the driver program must be network addressable from the worker nodes.
4. Because the driver schedules tasks on the cluster, it should be run close to the worker nodes, preferably on the same local area network. If you’d like to send requests to the cluster remotely, it’s better to open an RPC to the driver and have it submit operations from nearby than to run a driver far away from the worker nodes.

<p align="center">
<img src="Hadoop-v1.0-vs-Hadoop-v2.0.png" width=450 height=300 align="center"></p>

- Standalone – a simple cluster manager included with Spark that makes it easy to set up a cluster.
- Apache Mesos – a general cluster manager that can also run Hadoop MapReduce and service applications.
- Hadoop YARN – the resource manager in Hadoop 2.
- Kubernetes – an open-source system for automating deployment, scaling, and management of containerized applications.
- A third-party project (not supported by the Spark project) exists to add support for Nomad as a cluster manager.

Referance : 

 1. https://www.edureka.co/blog/spark-tutorial/?utm_campaign=social-media-edureka-november-sd&utm_medium=crosspost&utm_source=quora

2. https://spark.apache.org/docs/latest/cluster-overview.html
3. https://spark.apache.org/docs/latest/
