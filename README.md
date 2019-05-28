# Config-driven Release Management Workflow of Kafka Topics, Jobs & Schemas

This repository and related scripts introduce config-driven release workflow for key resource definitions in Kafka ecosystem:
 - topics in Kafka
 - schema definitions in Kafka Schema Registry
 - jobs in Kafka Connect
 
# Manual 
### Creating New Topic
Topics needs a valid schema definition file to be created first. 
Let's create a simple Avro schema:

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
topic: "mytopic" 
replication-factor: 1
partitions: 1
schema:  "mytopic"
````  
Finally, run deployment script to apply changes. 
This pushes new schema to the Schema Registry and creates a topic in Kafka. 
```` 
python deploy.py
````
###  Modifying Existing Topic
Changes in the existing topic definition would produce an "alter" statement on the existing Kafka topic during deployment. 
In the example below we are changing number of partitions inside an existing topic: 

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
in your YAML configs should be set inside __utils/variables.yml__.

### API Docs

**Kafka Manager**

https://github.com/yahoo/kafka-manager/blob/master/conf/routes  

**Kafka REST Proxy**

https://docs.confluent.io/current/kafka-rest/index.html

**Kafka Connect - REST API**

https://docs.confluent.io/current/connect/references/restapi.html

# Directory Structure
````
topics
  customer.yml
schemas
  customer
    customer.v1.avsc
    customer.v2.avsc
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
  infra
    single-node-kafka-dev
      docker-compose.yml  
  variables.yml
````
### Topics

#### Naming Convention
The following file naming convention is suggested (though, not enforced):

**&lt;topic name&gt;.yml**

#### File Format
This is a YAML config file with the following fields:

````
# Topic name 
# [type: string, required: true]
topic: "customer"  

# Topic replication factor
# [type: int, required: true]            
replication-factor: 1 
         
# Number of topic partitions         
# [type: int, required: true]           
partitions: 2

# Associated schema name (without version). 
# Schema should already be defined in the repo.                 
# [type: string, required: true]               
schema: "customer"

# Populated this list only when renaming a topic.
# Otherwise old topic content would be purged.              
# [type: array of strings, required: false] 
previous-names:
- "oldtopic"
````  

### Schemas
This folder contains all available schemas definitions that will be  synced with a Schema Repository (serves as a source of schemas).
Any changes made inside this folder (e.g. file create/update/delete) will be deployed as an incremental
change to the target Schema Registry. 

#### Naming Convention
The following file naming convention is used (and enforced):

**&lt;subject name&gt;.v&lt;version number&gt;.avsc**

If file name is not compatible, this would throw validation error during deployment.

#### File Format
We are using Apache Avro format to define message schemas. 
Avro schemas are declared with plain JSON. For specific details refer to the format spec: 

https://avro.apache.org/docs/1.3.3/spec.html

### Jobs
Repository of Kafka Connect job definitions.
Any operations made inside this folder (e.g. file create/update/delete) will be deployed as corresponding 
changes to the target Kafka Connect service. 
 
#### Naming Convention
The following file naming convention is suggested (though, not enforced):

**&lt;message type&gt;.&lt;dataset name&gt;.&lt;context&gt;.yml**

Valid message type values are left up to the organization/team to define. Typical types include:

  * __logging__ - for logging data (slf4j, syslog, etc)
  * __queuing__ - for classical queuing use cases.
  * __tracking__ - for tracking events such as user clicks, page views, ad views, etc.
  * __etl/db__ - for ETL and CDC use cases such as database feeds.
  * __streaming__ - for intermediate topics created by stream processing pipelines.
  * __push__ - for data that’s being pushed from offline (batch computation) environments into online environments.
  * __user__ - for user-specific data such as scratch and test topics.
  
#### File Format
Format follows Kafka Connect definitions:

https://docs.confluent.io/current/connect/references/allconfigs.html