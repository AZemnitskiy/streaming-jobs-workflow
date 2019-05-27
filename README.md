# Config-driven Release Management Workflow of Kafka Topics, Jobs & Schemas

This repository has a goal of establishing config-driven release workflow for key resource definitions in Kafka-centric ecosystem:
 - topics in Kafka
 - schema definitions in Schema Registry
 - jobs in Kafka Connect
 
# Manual 
### Creating New Topic
Topics requires a valid schema definition file to be created first. Let's create a simple Avro schema first:

**schemas/mytopic/mytopic.v1.yml**
````
{
  "type": "record",
  "namespace": "com.example",
  "name": "mytopic",
  "doc": "Avro Schema for my topic",
  "fields": [{
    "name": "test_field",
    "type": "string",
    "doc": "Test field"
  }]
}
````  
Next, create a YAML file populating required topic fields:

**topics/mytopic.yml**
````
topic: "mytopic"                      # required, topic name
replication-factor: 1                 # required, replicatio factor
partitions: 1                         # required, number of topic partitions
schema:  "mytopic"                    # required, schema definition is required before topic could be created
# previous-names:                     # optional, specify when renaming an existing topic
# - "oldtopic"
````  
Finally, run deployment script to apply changes. 
This pushes new schema to the Schema Registry and creates a topic in Kafka. 
```` 
python deploy.py
````
###  Modifying Existing Topic
Changes in the existing topic definition would produce an "alter" statement on the existing Kafka topic during deployment. 
In the example below we are changing number of partitions in an topic for an existing topic: 

**topics/mytopic.yml**
````
topic: "mytopic"                    
replication-factor: 1                
partitions: 10                        
schema: "mytopic"

````  
Run deployment script to apply changes.
 ```` 
python deploy.py
 ````

# API Endpoints & Variables
Required API endpoints and any extra variables that need to be substituted 
in your YAML configs should be set inside __utils/vars.yml__.

### API Docs

**Kafka Manager**

https://github.com/yahoo/kafka-manager/blob/master/conf/routes  

**Kafka REST Proxy**

https://docs.confluent.io/current/kafka-rest/index.html

**Kafka Connect - REST API**

https://docs.confluent.io/current/connect/references/restapi.html

# Directory Structure
````
infra
  single-node-kafka-dev
    docker-compose.yml
schemas
  customer
    customer.v1.avsc
    customer.v2.avsc
topics
  customer.yml
jobs
  etl
    etl.customer.in_file.yml
    etl.customer.out_file.yml
    etl.customer.out_azblob.yml
  streaming
    streaming.customer.yml  
utils
  deploy
    deploy_all.py
    deploy_jobs.py
    deploy_topics.py
    deploy_schemas.py
  vars.yml
````
### Schemas
This folder describes current state of message schemas in the Schema Repository.
Any operations made inside this folder (e.g. file create/update/delete) will be deployed as corresponding 
changes to the target Schema Registry node. 

#### Naming Convensions
The following file naming convension is followed (required for deployments):

**&lt;subject name&gt;.&lt;version number&gt;**

Deviations from the the naming convension above would throw validation error during deployment.

#### Avro Format
We are using Apache Avro format to define message schemas. 

https://avro.apache.org/docs/1.3.3/spec.html


### Jobs
Repository of Kafka Connect job definitions.
Any operations made inside this folder (e.g. file create/update/delete) will be deployed as corresponding 
changes to the target Kafka Connect node. 
 
#### Naming Convensions
The following naming convension is suggested (not strict):

**&lt;message type&gt;.&lt;dataset name&gt;.&lt;context&gt;**

Valid message type values are left up to the organization/team to define. Typical types include:

  * __logging__ - for logging data (slf4j, syslog, etc)
  * __queuing__ - for classical queuing use cases.
  * __tracking__ - for tracking events such as user clicks, page views, ad views, etc.
  * __etl/db__ - for ETL and CDC use cases such as database feeds.
  * __streaming__ - for intermediate topics created by stream processing pipelines.
  * __push__ - for data that’s being pushed from offline (batch computation) environments into online environments.
  * __user__ - for user-specific data such as scratch and test topics.