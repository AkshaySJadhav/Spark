
Deployment Modes :
========

In YARN, each application instance has an ApplicationMaster process, which is the first container started for that application.The application is responsible for requesting resources from the ResourceManager. Once the resources are allocated, the application instructs NodeManagers to start containers on its behalf. ApplicationMasters eliminate the need for an active client: the process starting the application can terminate, and coordination continues from a process managed by YARN running on the cluster.
