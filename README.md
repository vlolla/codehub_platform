# codehub-platform
## Scripts to Create Developer Platform for developement usage

This project contains docker-compose files that allow you to start an environment containing Kafka, Cassandra, postgres, prometheus, Grafana, MongoDb and Elastic Search.  
This environment can be used to build and test the streaming applications locally.

### Requirements

Docker installed and running.

### Starting The Streams Platform

This project contains several docker-compose files.  At startup you will need to pass two files.  The first file is the main docker-compose.  It contains all of the common configurations.  The second docker-compose file contains the project specific configurations which will override the common docker-compose file.  This is done with one of the following commands.

| Project | Startup Command |
| ----------- | ------------- |
| Domain1    | docker-compose -f ./docker-compose.yml -f ./docker-compose-domain1.yml up -d  | 
| Domain2    | docker-compose -f ./docker-compose.yml -f ./docker-compose-domain2.yml up -d |


If we need only Kafka - below command would load only confluent images

docker-compose -f ./docker-compose.yml -f ./docker-compose-domail1.yml up -d zookeeper  broker1  broker2  broker3  schema-registry  rest-proxy  kafka-topics-ui


When the streams-platform is started, the following containers are accessible from the user's machine.

| Container Name  | User Accessible Endpoint |
| --------------- | -------------- |
| cassandra       | localhost:9042 |
| broker1         | localhost:9092 |
| broker2         | localhost:9093 |
| broker3         | localhost:9094 |
| zookeeper       | localhost:2181 |
| schema-registry | localhost:8081 |
| prometheus      | http://localhost:9090 |
| grafana         | http://localhost:3000 username: admin password: admin |
| elasticsearch   | localhost:9200 |
| postgres        | localhost:5432 with username and password for test env |
| mongodb         | localhost:27017 with no userid and password

### Docker Network
When docker spins up the containers, it will create a network with the project name codehub32-platform_default.  
Any additional containers that are started that should interact with the codehub32-platform applications will have to connect to this network as well.


### Interacting With The Docker Environment

#### Connect To Cassandra

* Connect to the Cassandra container: `docker exec -it cassandra /bin/bash`
* Once connected to Cassandra container, you can create open a CQL session by running the command: `cqlsh`

#### List Topics In The Cluster

* Connect to the container named broker1 with: `docker exec -it broker1 /bin/bash`
* Once connected to broker1 run: `kafka-topics --list --zookeeper zookeeper:2181`

#### Print All Messages In A Topic

* Connect to the schema-registry container: 
`docker exec -it schema-registry /bin/bash` (we use the schema-registry containers because it comes with a avro consumer to deserialize avro messages)
* Once connected to schema-registry run: 
`kafka-avro-console-consumer --topic <<test.topic1>> --zookeeper zookeeper:2181 --property print.key=true --from-beginning`

### Docker Network

When docker spins up the containers, it will create a network with the project name streams-platform_default.  Any additional containers that are started that should interact with the streams-platform applications will have to connect to this network as well.


### Customizing For New Projects

To add a runtime for a new project, first add a configuration directory to docker/config to hold all of the project specific files.  You will also need to create a docker-compose file for this new project to override the common docker-compose.yml and point to these files.  This new docker-compose-\<Project>.yml file should be placed in the parent directory.  With this structure in place.

* Cassandra - Place a .cql file in the config directory.  This file will be executed at startup.  It should include all of the commands needed to create the Cassandra schema.  Then put an entry in docker-compose-\<Project>.yml to point to the config directory.
* Kafka - Place a .sh file in the config directory with the commands needed to create the Kafka topics.  Make sure to include the wait logic before creating the topics and `exec "$@"` at the end of the script.  With these commands, the script will verify that brokers 2 and 3 are ready for traffic, create the topics and then proceed with the startup of broker 1.  Finally, put an entry in docker-compose-\<Project>.yml to point to the topic creation script.
* Prometheus - Create a prometheus.yml configuration file in the config directory listing the applications to be scraped.  Then put an entry in docker-compose-\<Project>.yml to expose this file to the Prometheus container.
* Grafana - With the environment running and Prometheus configured to scrape the apps you wish to graph, create the dashboard using the web UI exposed by the Grafana container.  Export the dashboard as a .json file and place this export in the docker/config/\<Project>/dashboards directory.  Also, update docker-compose-\<Project>.yml so the Grafana container knows where the project specific dashboards are located.

