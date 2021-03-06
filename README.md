# MongoDB Zabbix monitorign plugin
说明：php神马的可以直接装在zabbix server上，我整个过程都做在zabbix server上了

Overview
========

The *MongoDB Plugin* can be used to monitor standalone, replicated as well as clustered MongoDB instances. The plugin monitors availability, resource utilization, health, performance and other important metrics of a MongoDB environment. Coupled with the Zabbix OS level monitoring, the MongoDB plugin provides great peace of mind knowing that MongoDB is being monitored 24x7 and sufficient data would be available for sizing, scalability, troubleshooting and support.


Setup and Configuration
=======================
一、先安装php等，这个应该都能看懂吧。

The MongoDB plugin uses the MongoDB PHP driver which needs to be installed on the Zabbix server.  For this, install and setup the following packages:

* **php5-dev (or php5-devel)** = Files for PHP5 module development
* **php5-pear** = PEAR - PHP Extension and Application Repository
* **gcc** = GNU C Compiler
* **make** = make utility

To install the above on a Zabbix appliance, login into the appliance as root (default password = zabbix) and run the following commands:

* yast -i php5-devel
* yast -i gcc
* yast -i php5-pear
* yast -i make

Now install the php MongoDB driver using the instructions at [http://us2.php.net/manual/en/mongo.installation.php](http://us2.php.net/manual/en/mongo.installation.php)

In the case of the Zabbix appliance, run the the following pecl command:

* pecl install mongo

After successful installation of the MongoDB driver, you need to "enable" it within php5. Edit the two files **/etc/php5/cli/php.ini and /etc/php5/apache2/php.ini** and add a line to the "Dynamic Extensions" sections as shown below.

```
;;;;;;;;;;;;;;;;;;;;;;
; Dynamic Extensions ;
;;;;;;;;;;;;;;;;;;;;;;

extension=mongo.so
```


Ensure that the php MongoDB driver is setup and configured properly by testing out one of the sample php programs for MongoDB driver ([http://us2.php.net/manual/en/mongo.tutorial.php](http://us2.php.net/manual/en/mongo.tutorial.php)).

Download the MongoDB Plugin shell script and php file from [https://github.com/nightw/mikoomi-zabbix-mongodb-monitoring/find/master](https://github.com/nightw/mikoomi-zabbix-mongodb-monitoring/find/master) and copy them into `externalscripts` (e.g. `/etc/zabbix/externalscripts`) directory on the MongoDB node you want to monitor. **Make sure that the php script and shell script are made executable.**

Next open up a browser and download the MongoDB Zabbix template.
Now login to the Zabbix frontend (if it still has the default user and password, then it should be Admin/zabbix).

二、现在导入mikoomi的模版

Navigate as follows: 

* Configuration >> Templates
* Click on the "Import Template" button on the top right-hand corner
* In the "Import file" dialog box, browse/search/enter the filename of the Zabbix template that was downloaded
* Upload the template

***Now you are ready to start monitoring your MongoDB servers !***



Monitoring a MongoDB Environment (single server, replicaset or cluster)
=======================================================================
 
Follow these steps to start monitoring a MongoDB server

* Setting up Zabbix server's side
  * Make sure the host running the MongoDB is added to Zabbix Hosts previously (see host addition [here](https://www.zabbix.com/documentation/2.2/manual/quickstart/host))
  * Login to the Zabbix front-end and navigate to **_Configuration >> Hosts_**
  * Click on host which is running the MongoDB button on the left
  * Click on **Templates** in the top menu bar
  * Use the **Select** button on the right side
  * Select the **Mikoomi Templates** group in the upper right corner
  * Check **Template_MongoDB**
  * Click on **Select** button
  * Click on **Add** button
  * Click **Save**
* Setting up the MongoDB server node
  * Add something like this (look out especially for the **ZABBIX_HOSTNAME** variable, which must meet the name of the node in the Zabbix server which we attached the template to in the previous steps) the following to the Zabbix user's crontab:

三、这里有点坑，我这有5台mongo，我全做zabbix server上，过程如下：

在计划任务里面添加每分钟运行

mikoomi-mongodb-plugin.sh -H zabbix server的ip -P 10051 -z mongo的主机，在zabbix里面添加时候的Host name -h 当然是你的mongodb的ip -p 27017

你有多少台mongo就加多少计划任务，计划任务添加完，去/tmp下面看mikoomi-mongodb-plugin.php_xxxx.data和mikoomi-mongodb-plugin.php_xxxx.log，如果日志没有报错就大功告成了！

下面是mikoomi的命令帮助

mikoomi-mongodb-plugin.php Version 0.4

Usage : mikoomi-mongodb-plugin.php [-D] [-h <mongoDB Server Host>] [-p <mongoDB Port>] [--ssl] [-u <username>] [-x <password>] [-H
<Zabbix Server ip/hostname>] [-P <Zabbix Server Port>] -z <Zabbix_Name>

where

   -D    = Run in detail/debug mode
   
   -h    = Hostname or IP address of server running MongoDB
   
   -p    = Port number on which to connect to the mongod or mongos process
   
   -z    = Name (hostname) of MongoDB instance or cluster in the Zabbix UI
   
   -u    = User name for database authentication
   
   -x    = Password for database authentication
   
   -H    = Zabbix server IP or hostname
   
   -P    = Zabbix server Port or hostname
   
   --ssl = Use SSL when connecting to MongoDB
   
```
ZABBIX_HOSTNAME=$(hostname -f)
* * * * * /etc/zabbix/externalscripts/mikoomi-mongodb-plugin.sh -z $ZABBIX_HOSTNAME
```

**Note that in a sharded and/or replicated MongoDB environment, you need to monitor only ONE of the mongos process**. However that process needs to be aware of the entire Mongo environment (or cluster) - i.e. all the shards and all the replicas within each replicaset. 

Now data should be collected by the template at intervals of 60 seconds.

If something is wrong (data does not show up in the Zabbix server, etc.) then you should look at the output at /tmp/mikoomi-mongodb-plugin.php_*.log file)


Monitored Metrics
=================

The MongoDB plugin monitors the following metrics or items during each cycle:

* Asserts: Total Msg Asserts
* Asserts: Total Regular Asserts
* Asserts: Total Assert Rollovers
* Asserts: Total User Asserts
* Asserts: Total Warning Asserts
* Background Flushing: Background Flush Average Time (ms)
* Background Flushing: Number of Flushes in Last 1 Minute
* Background Flushing: Last Background Flush Time (ms)
* Background Flushing: Total Background Flush Time (ms) in Last 1 Minute
* Cursors: Client Cursor Size
* Cursors: Cursor Time Outs
* Cursors: Open Curors
* Database Connections: Connections Available
* Database Connections: Current Connections
* Databases and Collections: List of All Database Stats
* Databases and Collections: List of Database Average Object Size
* Databases and Collections: List of Database Collection Count
* Databases and Collections: Database Count
* Databases and Collections: List of Database Data Size
* Databases and Collections: List of Database File Size
* Databases and Collections: List of Database Index Count
* Databases and Collections: List of Database Index Size
* Databases and Collections: Total Collections Across All Databases
* Databases and Collections: Total Indexes Across All Collections
* Databases and Collections: Total Objects Across All Databases
* Databases and Collections: Total Size of All Databases (MB)
* Databases and Collections: List of Database Extent Count
* Databases and Collections: List of Database Object Count
* Databases and Collections: List of Database Storage Size
* Global Locking: Current Reader Queue Length
* Global Locking: Current Total Queue Length
* Global Locking: Current Writer Queue Length
* Global Locking: Total Lock Time (microseconds) in Last 1 Minute
* Index Effectiveness: Total Btree Accesses in Last 1 Minute
* Index Effectiveness: Total Btree Hits in Last 1 Minute
* Index Effectiveness: Total Btree Misses in Last 1 Minute
* Index Effectiveness: Btree Miss Ratio
* Index Effectiveness: Total Btree Resets
* Journaling: Commits in last 1 Minute
* Journaling: Commits in Writelocks in last 1 Minute
* Journaling: Datafile Write Time (ms) in last 1 Minute
* Journaling: Datafile Writes (MB) in last 1 Minute
* Journaling: Early Commits in last 1 Minute
* Journaling: Journal Write Time (ms) in last 1 Minute
* Journaling: Journal Writes (MB) in last 1 Minute
* Journaling: Log Buffer Prep Time (ms) in last 1 Minute
* Memory: Heap Memory Size (MB)
* Memory: Page Faults/minute
* Memory: Memory Addressing (32/64 bit)
* Memory: Resident Memory Size (MB)
* Memory: Virtual Memory Size (MB)
* Miscellaneous:  MongoDB Plugin Checksum
* Miscellaneous: MongoDB Plugin Data Collection Time (seconds)
* Miscellaneous: MongoDB Plugin Version
* Miscellaneous: MongoDB Version
* Miscellaneous: MongoDB Uptime (seconds)
* Network Activity: Network Inbound Traffic (MB)
* Network Activity: Network Outbound Traffic (MB)
* Network Activity: Network Requests
* Performance: Is there any writeback operations queued
* OpCounters: Total Commands in Last 1 Minute
* OpCounters: Total Delete Ops in Last 1 Minute
* OpCounters: Total Getmore Ops in Last 1 Minute
* OpCounters: Total Insert Ops in Last 1 Minute
* OpCounters: Total Query Ops in Last 1 Minute
* OpCounters: Total Update Ops in Last 1 Minute
* Replication: Is Mongo Server Part of a ReplicatSet
* Replication: Entries in oplog.rs Collection
* Replication: Count of ReplicaSet Members Needing Attention
* Replication: List of ReplicaSet Members in Attention State
* Replication: ReplicaSet Host Names
* Replication: ReplicaSet Member Count
* Replication: ReplicaSet Name
* Sharding: Is Mongo Server a Cluster Router (mongos process)
* Sharding: List of Sharded Databases and Collections
* Sharding: Total Number of Chunks
* Sharding: Total Number of Shards
* Sharding: List of Shards in Cluster
* Sharding: List of Sharded Collections in Cluster
* Sharding: Total Number of Sharded Collections

Pre-canned Triggers
===================

Triggers in Zabbix are events of interest that happen with respect to the monitored metrics. For example, the plugin keeps track of the total number of collections. If this count changes, it flags this event by firing a trigger. You can choose to ignore this trigger or you can choose to take an action - e.g. send an email or run a shell script.

The MongoDB plugin comes with the following built-in triggers:

* One or more databases have been created
* One or more databases have been destroyed
* No Data Received in 5 minutes
* One or more replication members need attention
* One or more members have been removed from the ReplicaSet
* One or more members have been added to the ReplicaSet
* One or more new shard chunks have been created
* One or more shards have been added to the cluster
* One or more shards have been removed from the cluster
* One or more new sharded collections have been created
* One or more collections have been dropped
* One or more collections have been added
* One or more indexes have been dropped
* One or more indexes have been added

Pre-canned Graphs
=================

While Zabbix allows creating graphing one or more monitored metric, the plugin comes with the following pre-canned graphs to get you productive immediately:

**MongoDB Cumulative Database Size in MB**

**MongoDB Memory Footprint:** This graph plots the following metrics

* Virtual Memory Size (MB)
* Heap Memory Size (MB)
* Resident Memory Size (MB)

**MongoDB Database Operations:** This graph plots the following metrics

* Total Query Ops in Last 1 Minute
* Total Update Ops in Last 1 Minute
* Total Insert Ops in Last 1 Minute
* Total Delete Ops in Last 1 Minute
* Total Command Ops in Last 1 Minute
* Total Get More Ops in Last 1 Minute

**MongoDB Journaling:** This graph plots the following metrics

* Commits in last 1 Minute
* Datafile Writes (MB) in last 1 Minute
* Datafile Write Time (ms) in last 1 Minute
* Journaling: Journal Writes (MB) in last 1 Minute
* Journaling: Journal Write Time (ms) in last 1 Minute
