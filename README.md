## A simple, 3 node MongoDB cluster on Docker Swarm

Given a Docker Swarm, will start up 3 Mongo DB instances sharing a replica set.

Based on [@lucjuggery](https://twitter.com/lucjuggery)'s  https://medium.com/lucjuggery/mongodb-replica-set-on-swarm-mode-45d66bc9245 but updated for v3 Compose syntax.

### Usage
1. Build the `mongors` image:

```
$ docker-compose build

WARNING: Some services (rs) use the 'deploy' key, which will be ignored. Compose does not support 'deploy' configuration - use `docker stack deploy` to deploy to a swarm.
rs1 uses an image, skipping
rs2 uses an image, skipping
rs3 uses an image, skipping
Building rs
Step 1/5 : FROM mongo:3.2
 ---> f2b4f7eadfd2
Step 2/5 : COPY init.sh /tmp/init.sh
 ---> Using cache
 ---> 5a02bbaf1db5
Step 3/5 : RUN chmod +x /tmp/init.sh
 ---> Using cache
 ---> e143322f615c
Step 4/5 : LABEL maintainer="Eric Smalling <smalls@docker.com>"
 ---> Running in fddadee5ab63
Removing intermediate container fddadee5ab63
 ---> 325bfed426b0
Step 5/5 : CMD /tmp/init.sh
 ---> Running in 7e27035f1f8a
Removing intermediate container 7e27035f1f8a
 ---> d4962824f7cb

Successfully built d4962824f7cb
Successfully tagged ericsmalling/mongors:3.4

```

2. Deploy the stack:

```
$ docker stack deploy -c docker-compose.yml db

Ignoring unsupported options: build

Creating network db_mongo
Creating service db_rs2
Creating service db_rs3
Creating service db_rs
Creating service db_rs1
```

3. Check state of services:

```
$ docker service ls

ID                  NAME                MODE                REPLICAS            IMAGE                      PORTS
jnbd0m27pfni        db_rs               replicated          0/1                 ericsmalling/mongors:3.4   
58r9k9pp4el2        db_rs1              replicated          1/1                 mongo:3.4                  
lb6rnp9icrdt        db_rs2              replicated          1/1                 mongo:3.4                  
fb0xaw2ftcop        db_rs3              replicated          1/1                 mongo:3.4           
```

*The db_rs service will try to run several times waiting for the other services
to finish starting.  The last run should succeed, check with the following and the last (top-most) attempt should have a `Complete` state:*

```
$ docker service ps db_rs

ID                  NAME                IMAGE                      NODE                    DESIRED STATE       CURRENT STATE             ERROR                       PORTS
etwghr495vzn        db_rs.1             ericsmalling/mongors:3.4   linuxkit-025000000001   Shutdown            Complete 42 seconds ago                               
b0mfdypyqkph         \_ db_rs.1         ericsmalling/mongors:3.4   linuxkit-025000000001   Shutdown            Failed 49 seconds ago     "task: non-zero exit (1)"   
```

Additional verification (substitute the correct container name/hash in your environment):

```
$ docker exec -it db_rs1.1.iwokdgch5bud3tihaqd6484li mongo

MongoDB shell version v3.4.16
connecting to: mongodb://127.0.0.1:27017
MongoDB server version: 3.4.16
Welcome to the MongoDB shell.
For interactive help, type "help".
For more comprehensive documentation, see
	http://docs.mongodb.org/
Questions? Try the support group
	http://groups.google.com/group/mongodb-user
Server has startup warnings:
2018-07-20T00:27:38.874+0000 I STORAGE  [initandlisten]
2018-07-20T00:27:38.874+0000 I STORAGE  [initandlisten] ** WARNING: Using the XFS filesystem is strongly recommended with the WiredTiger storage engine
2018-07-20T00:27:38.874+0000 I STORAGE  [initandlisten] **          See http://dochub.mongodb.org/core/prodnotes-filesystem
2018-07-20T00:27:38.904+0000 I CONTROL  [initandlisten]
2018-07-20T00:27:38.904+0000 I CONTROL  [initandlisten] ** WARNING: Access control is not enabled for the database.
2018-07-20T00:27:38.904+0000 I CONTROL  [initandlisten] **          Read and write access to data and configuration is unrestricted.
2018-07-20T00:27:38.904+0000 I CONTROL  [initandlisten]

rs0:PRIMARY> rs.status()
{
	"set" : "rs0",
	"date" : ISODate("2018-07-20T00:44:17.877Z"),
	"myState" : 1,
	"term" : NumberLong(1),
	"syncingTo" : "",
	"syncSourceHost" : "",
	"syncSourceId" : -1,
	"heartbeatIntervalMillis" : NumberLong(2000),
	"optimes" : {
		"lastCommittedOpTime" : {
			"ts" : Timestamp(1532047457, 1),
			"t" : NumberLong(1)
		},
		"appliedOpTime" : {
			"ts" : Timestamp(1532047457, 1),
			"t" : NumberLong(1)
		},
		"durableOpTime" : {
			"ts" : Timestamp(1532047457, 1),
			"t" : NumberLong(1)
		}
	},
	"members" : [
		{
			"_id" : 0,
			"name" : "rs1:27017",
			"health" : 1,
			"state" : 1,
			"stateStr" : "PRIMARY",
			"uptime" : 999,
			"optime" : {
				"ts" : Timestamp(1532047457, 1),
				"t" : NumberLong(1)
			},
			"optimeDate" : ISODate("2018-07-20T00:44:17Z"),
			"syncingTo" : "",
			"syncSourceHost" : "",
			"syncSourceId" : -1,
			"infoMessage" : "",
			"electionTime" : Timestamp(1532046476, 1),
			"electionDate" : ISODate("2018-07-20T00:27:56Z"),
			"configVersion" : 1,
			"self" : true,
			"lastHeartbeatMessage" : ""
		},
		{
			"_id" : 1,
			"name" : "rs2:27017",
			"health" : 1,
			"state" : 2,
			"stateStr" : "SECONDARY",
			"uptime" : 992,
			"optime" : {
				"ts" : Timestamp(1532047447, 1),
				"t" : NumberLong(1)
			},
			"optimeDurable" : {
				"ts" : Timestamp(1532047447, 1),
				"t" : NumberLong(1)
			},
			"optimeDate" : ISODate("2018-07-20T00:44:07Z"),
			"optimeDurableDate" : ISODate("2018-07-20T00:44:07Z"),
			"lastHeartbeat" : ISODate("2018-07-20T00:44:16.645Z"),
			"lastHeartbeatRecv" : ISODate("2018-07-20T00:44:16.957Z"),
			"pingMs" : NumberLong(0),
			"lastHeartbeatMessage" : "",
			"syncingTo" : "rs3:27017",
			"syncSourceHost" : "rs3:27017",
			"syncSourceId" : 2,
			"infoMessage" : "",
			"configVersion" : 1
		},
		{
			"_id" : 2,
			"name" : "rs3:27017",
			"health" : 1,
			"state" : 2,
			"stateStr" : "SECONDARY",
			"uptime" : 992,
			"optime" : {
				"ts" : Timestamp(1532047447, 1),
				"t" : NumberLong(1)
			},
			"optimeDurable" : {
				"ts" : Timestamp(1532047447, 1),
				"t" : NumberLong(1)
			},
			"optimeDate" : ISODate("2018-07-20T00:44:07Z"),
			"optimeDurableDate" : ISODate("2018-07-20T00:44:07Z"),
			"lastHeartbeat" : ISODate("2018-07-20T00:44:16.645Z"),
			"lastHeartbeatRecv" : ISODate("2018-07-20T00:44:16.956Z"),
			"pingMs" : NumberLong(0),
			"lastHeartbeatMessage" : "",
			"syncingTo" : "rs1:27017",
			"syncSourceHost" : "rs1:27017",
			"syncSourceId" : 0,
			"infoMessage" : "",
			"configVersion" : 1
		}
	],
	"ok" : 1
}
```

4. Shut it all down:

```
$ docker stack rm db

Removing service db_rs
Removing service db_rs1
Removing service db_rs2
Removing service db_rs3
Removing network db_mongo
```
