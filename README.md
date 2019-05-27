# Streaming Jobs Workflow

Reference workflow that uses config-based directory structure to:  
 - control all topics operations (create/update/delete) are via configuration changes
 - all message schema operations are performed via configuration changes
 - Kafka Connect jobs are configured 

# Project Structure
````
infra
  single-node-kafka-dev
    docker-compose.yml
schemas
  customer
    customer.v1.avro
    customer.v2.avro
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
# Jobs
Repository of Kafka Connect jobs. 

### Naming Convension
The following naming convension is suggested:

** <message type>.<dataset name>.<data name> **

Valid message type values are left up to the organization/team to define. Typical types include:

  * __logging__ - for logging data (slf4j, syslog, etc)
  * __queuing__ - for classical queuing use cases.
  * __tracking__ - for tracking events such as user clicks, page views, ad views, etc.
  * __etl/db__ - for ETL and CDC use cases such as database feeds.
  * __streaming__ - for intermediate topics created by stream processing pipelines.
  * __push__ - for data that’s being pushed from offline (batch computation) environments into online environments.
  * __user__ - for user-specific data such as scratch and test topics.