# Getting Started with DSE On Docker

![](https://github.com/roberd13/Getting-Started-With-DSE-and-Docker/blob/master/red-bluepill.png)

*You take the ***blue pill***, the story ends. You wake up in your bed and believe whatever you want to believe about your normal ideas on how to run DSE and your apps. You take the ***red pill***, you stay in Wonderland, I show you how deep the rabbit hole goes and you set sail on the Container ship of the future called  DSE on Docker*

## ***Welcome to the real world!!!***

*Containers are the Future, [Google boast that they run everything in containers](https://www.theregister.co.uk/2014/05/23/google_containerization_two_billion/). So why not you?  Containers are scalable and fast to deploy, you can have a single node or multinode cluster up and running in minutes with just a few commands that is in a consistent, environment,  plus you can share them with your teammates and be the hero.*

*There are multiple ways to get DSE on Docker.  You can pull DSE from *[Docker Store](https://store.docker.com/images/datastax)*, use docker compose or even roll your own from our public *[Github Repo](https://github.com/datastax/docker-images).**

*Let's not waste any time and get you started on your mastery by creating some  DSE, OpsCenter and Studio Containers.*

***This blog assumes you to have***

* *A basic understanding of Docker images and containers.*

* *Docker installed on your local system, see [Docker Installation Instructions](https://docs.docker.com/engine/installation/).*

* *When *[building](https://github.com/datastax/docker-images#building)* custom images from the DataStax github repository, a *[DataStax Academy account](https://academy.datastax.com/).**

**_The Basics_**

*Here you will see examples on how to start a simple standalone container for the specific workloads by adding flags to the end of the `docker run` command.  No flag for transactional, -g for graph, -k for Analytics, -s for Search workloads.  You can use any combination of those flags for the workload you need.*

***Notice the -e flag which is for environmental variable it has `DS_LICENSE=accept` just after it.  This is required or the container will not start.***

***Now for some fun.***

# **_Lets create a DataStax Enterprise container_**

### **_Create a DSE database container_**

```
docker run -e DS_LICENSE=accept --name my-dse -d store/datastax/dse-server:5.1.5
```

### **_Create a DSE container with Graph enabled_**

```
docker run -e DS_LICENSE=accept --name my-dse -d store/datastax/dse-server:5.1.5 -g
```

### **_Create a DSE container with Analytics (Spark) enabled_**

```
docker run -e DS_LICENSE=accept --name my-dse -d store/datastax/dse-server:5.1.5 -k
```

### **_Create a DSE container with Search enabled_**

```
docker run -e DS_LICENSE=accept --name my-dse -d store/datastax/dse-server:5.1.5 -s
```

### **_Congratulations You have now graduated to Advanced training!_**

**_It’s time to build on the basics you learned and use some of the features that have been added to the DSE Docker container_**

### *DataStax provided Docker images include a start up script that swaps DSE configuration files found in the Volume **/conf** with the configuration file in the default location on the container.*

*To use this feature:*

1. *Create a directory on your local host.*

2. *Add the configuration files you want to replace. Use the following links for full list of configuration files:*

    * *[DSE](https://github.com/datastax/docker-images/blob/master/server/5.1/files/overwritable-conf-files)*

    * *[OPSCENTER](https://github.com/datastax/docker-images/blob/master/opscenter/6.1/files/overwritable-conf-files)*

    * *[STUDIO](https://github.com/datastax/docker-images/blob/master/studio/2.0/files/overwritable-conf-files)*

3. *The file name must match a corresponding configuration file in the image and include all the required values, for example **cassandra.yaml**, **dse.yaml**, **opscenterd.conf**.*

4. *Mount the local directory to the exposed Volume **/conf**.*

5. *Start the container. For example to start a transactional node:*

*`docker run -e DS_LICENSE=accept --name my-dse -d  -v /dse/conf:/conf store/datastax/dse-server:5.1.5`*

*The DSE images also expose the following volumes.*

* *For DataStax Enterprise Transactional, Search, Graph, and Analytics workloads:*

    * *`/var/lib/cassandra`: Data from Cassandra*

    * *`/var/lib/spark`: Data from DSE Analytics w/ Spark*

    * *`/var/lib/dsefs`: Data from DSEFS*

    * *`/var/log/cassandra`: Logs from Cassandra*

    * *`/var/log/spark`: Logs from Spark*

    * *`/conf`: Directory to add custom config files for the container to pickup.*

* *For OpsCenter: `/var/lib/opscenter`*

* *For Studio: `/var/lib/datastax-studio`*

*To persist data, pre-create directories on the local host and map the directory to the corresponding volume using the docker run -v flag.*

*NOTE: If the volumes are not mounted from the local host, all data is lost when the container is removed.*

*To mount a volume, use the following syntax:*

*`docker run -v <local_directory>:<container_volume>`*

*Example Mount the host directory `/dse/conf` to the DSE volume `/conf` to manage configuration files.*

```
docker run -e DS_LICENSE=accept --name my-dse -d  -v /dse/conf:/conf store/datastax/dse-server:5.1.5
```

*See Docker docs > [Use volumes](https://docs.docker.com/engine/tutorials/dockervolumes/#mount-a-host-directory-as-a-data-volume) for more information.*

# **_Now you have a container running with advanced features, it’s time to use your container._**

# **_Using interactive commands_**

*If the container is running in the background (using the **-d**), use the following command to run DSE commands interactively.*

*`docker exec -it <container_name> <command_name>`*

*For example*

*`nodetool status` command:*

```
docker exec -it my-dse nodetool status
```

*To load data we can use a cqlsh prompt:*

```
docker exec -it my-dse cqlsh
```

*If you want to get a bash prompt where you can do anything just like having a ssh connection to an oldschool node*

*`docker exec -it <container_name> bash`*

*To exit the shell without stopping the container use* **ctl P ctl Q** 

# *Now that you have mastered DSE let’s **_Create an OpsCenter container_***

*Here we will learn how to create an Opscenter container and a connected DataStax Enterprise server container on the same Docker host.*

*To create and connect the containers:*

*First create an OpsCenter container.*

```
docker run -e DS_LICENSE=accept -d -p 8888:8888 --name my-opscenter
```

*See [OpsCenter Docker run options](https://github.com/datastax/docker-images#OpsCenter-Docker-run-options) for additional options that persist data or manage configuration.*

*To Create a [DataStax Enterprise (DSE) server](https://store.docker.com/images/datastax) container that is linked to the OpsCenter container.*

```
docker run -e DS_LICENSE=accept --link my-opscenter:opscenter --name my-dse -d store/datastax/dse-server:5.1.5
```

*Get the DSE container IP address:*

*`docker exec -it my-dse nodetool status`*

1. *Open a browser and go to **http://DOCKER_HOST_IP:8888**.*

2. *Click **Manage existing cluster**.*

3. *In **host name**, enter the DSE IP address.*

4. *Click **Install agents manually**. Note that the agent is already installed on the DSE image; no installation is required.*

*OpsCenter is ready to use with DSE. See the [OpsCenter User Guide](http://docs.datastax.com/en/opscenter/6.1/) for detailed usage and configuration instructions.*

**_Wait, there's more_**

**_ Now dive deeper and create a Studio container_**

```
docker run -e DS_LICENSE=accept --link my-dse --name my-studio -p 9091:9091 -d datastax/dse-studio
```

1. *Open your browser and point to **http://DOCKER_HOST_IP:9091**, create the new connection using my-dse as the hostname.*

*Check *[Studio documentation](http://docs.datastax.com/en/dse/5.1/dse-dev/datastax_enterprise/studio/stdToc.html)* for further instructions.*

# **_If that was just a little too much fun for you or you don’t need any customization, you can use Docker Compose for automated provisioning_**

*Bootstrapping a multi-node cluster with OpsCenter and Studio can be automated with **[Docker Compose](https://docs.docker.com/compose/)**. To get sample **compose.yml** files visit the following links.*

* *[DSE](https://github.com/datastax/docker-images/blob/master/docker-compose.yml)*

* *[OpsCenter](https://github.com/datastax/docker-images/blob/master/docker-compose.opscenter.yml)*

* *[Studio](https://github.com/datastax/docker-images/blob/master/docker-compose.studio.yml)*

**_Wow this is so simple_**

*3-Node Setup*

```
docker-compose  -f docker-compose.yml up -d --scale node=2
```

*3-Node Setup with OpsCenter*

```
docker-compose -f docker-compose.yml -f docker-compose.opscenter.yml up -d --scale node=2
```

*3-Node Setup with OpsCenter and Studio*

```
docker-compose -f docker-compose.yml -f docker-compose.opscenter.yml -f docker-compose.studio.yml up -d --scale node=2
```

*Single Node Setup with Studio*

```
docker-compose -f docker-compose.yml -f docker-compose.studio.yml up -d --scale node=0
```

**_In Closing_**

*I hope you have enjoyed learning the basics of creating a DSE container or cluster using Docker.  If you would like to learn more and see more advanced configuration options please visit *[Installing DSE on Docker](http://docs.datastax.com/en/dse/5.1/dse-admin/datastax_enterprise/install/dockerReadme.html)* and our public *[Github Repo](https://github.com/datastax/docker-images/blob/master/README.md)*.*

*I will leave you with this*

## "You are the ONE, use your training for good and awaken the rest of your colleagues to the Real World of DSE on Docker."

