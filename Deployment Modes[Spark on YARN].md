Deployment Modes :
================

  In YARN, each application instance has an ApplicationMaster process, which is the first container started for that application.The application is responsible for requesting resources from the ResourceManager. Once the resources are allocated, the application instructs NodeManagers to start containers on its behalf. ApplicationMasters eliminate the need for an active client: the process starting the application can terminate, and coordination continues from a process managed by YARN running on the cluster.

#### 1. Cluster Deployment Mode :


In cluster mode, the Spark driver runs in the ApplicationMaster on a cluster host. A single process in a YARN container is responsible for both driving the application and requesting resources from YARN. The client that launches the application does not need to run for the lifetime of the application.

Cluster mode is not well suited to using Spark interactively. Spark applications that require user input, such as spark-shell and pyspark, require the Spark driver to run inside the client process that initiates the Spark application.


<img src="spark-yarn-cluster.png" height=500 width=700>

#### 2. Client Deployment Mode :

In client mode, the Spark driver runs on the host where the job is submitted. The ApplicationMaster is responsible only for requesting executor containers from YARN. After the containers start, the client communicates with the containers to schedule work.

<img src="spark-yarn-client.png" height=500 width=700>


Below is the summary of both the modes :

<img src="ModeSummary.png">


When user submits a job in client mode then Driver program will spone on the same node that process will consume memory from the same host. Where as in cluster mode driver program will run on any of the available node on cluster. Spark is smart and reources allocation will happen in such a way that driver program get automatically sponned on any available node.
 
