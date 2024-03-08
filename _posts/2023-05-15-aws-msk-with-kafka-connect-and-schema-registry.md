---
layout: post
title:  "AWS MSK with kafka-connect and schema registry"
categories: kafa AWS
tags: AWS kafka MSK 
---

In this blog I will explain how to configure kafka-connect to work with MSK and schema registry.

The MSK uses IAM based authentication . In the future blogs I will also talk about working with other authentication methods.

If you try to do without extra configurations we will get the following error :

```
Error while getting broker list.  
connect  | java.util.concurrent.ExecutionException: org.apache.kafka.common.errors.TimeoutException: Timed out waiting for a node assignment. Call: listNodes  
connect  |      at java.base/java.util.concurrent.CompletableFuture.reportGet(CompletableFuture.java:395)  
connect  |      at java.base/java.util.concurrent.CompletableFuture.get(CompletableFuture.java:1999)  
connect  |      at org.apache.kafka.common.internals.KafkaFutureImpl.get(KafkaFutureImpl.java:165)  
connect  |      at io.confluent.admin.utils.ClusterStatus.isKafkaReady(ClusterStatus.java:147)  
connect  |      at io.confluent.admin.utils.cli.KafkaReadyCommand.main(KafkaReadyCommand.java:149)  
connect  | Caused by: org.apache.kafka.common.errors.TimeoutException: Timed out waiting for a node assignment. Call: listNodes  
connect  | Expected 1 brokers but found only 0. Trying to query Kafka for metadata again ...  
connect  | Expected 1 brokers but found only 0. Brokers found [].  
connect  | Using log4j config /etc/cp-base-new/log4j.properties
```

Though the error says timeout . I checked everything and there is no connectivity issues.

After reading couple of github issues I came to know that timeout doesn’t necessary mean it is network issue and could be many other reasons for that.

Installing the IAM Authentication Library
=========================================

The first step towards resolving this issue involves installing the necessary AWS MSK IAM authentication library. This library can be downloaded from [this link](https://github.com/aws/aws-msk-iam-auth/releases/download/v1.1.4/aws-msk-iam-auth-1.1.4-all.jar). After installation, it’s crucial to define proper properties for Kafka-Connect to perform IAM-based authentication.

Building a Docker Image for Kafka-Connect
=========================================

Next, we’ll build a Docker image on top of the Confluent Kafka Connect base image to connect with AWS MSK. This new image comes pre-packaged with essential libraries stored in the right directories and permissions appropriately set.

```
# Use a specific version for reproducibility  
FROM confluentinc/cp-kafka-connect-base:7.4.0  
  
# Define environment variable for better readability  
ENV JAR_URL=https://github.com/aws/aws-msk-iam-auth/releases/download/v1.1.4/aws-msk-iam-auth-1.1.4-all.jar  
ENV KAFKA_PATH=/usr/share/java/kafka  
ENV CP_BASE_PATH=/usr/share/java/cp-base-new  
  
# Use a single layer for adding the JAR files and setting permissions  
RUN curl -L -o ${KAFKA_PATH}/aws-msk-iam-auth-1.1.4-all.jar ${JAR_URL} && \
    cp ${KAFKA_PATH}/aws-msk-iam-auth-1.1.4-all.jar ${CP_BASE_PATH}/aws-msk-iam-auth-1.1.4-all.jar && \
    chmod 666 ${KAFKA_PATH}/aws-msk-iam-auth-1.1.4-all.jar ${CP_BASE_PATH}/aws-msk-iam-auth-1.1.4-all.jar && \
    chown appuser:appuser ${KAFKA_PATH}/aws-msk-iam-auth-1.1.4-all.jar ${CP_BASE_PATH}/aws-msk-iam-auth-1.1.4-all.jar   
  
USER appuser  
  
EXPOSE 8083  

```

I ran kafka-connect as a service using docker compose below

```
version: "3.5"  
services:  
  connect:  
    image: connect:latest  
    hostname: localhost:connect  
    container_name: connect  
    ports:  
       - 8083:8083  
    volumes:  
      - "./jars:/jars2"  
      - "./data:/topic_data"  
    environment:  
      CONNECT_BOOTSTRAP_SERVERS: '<MSK_CLUSTER_BOOTSTRAP_SERVER>'  
      CONNECT_REST_PORT: '8083'  
      CONNECT_REST_ADVERTISED_HOST_NAME: connect  
      CONNECT_GROUP_ID: compose-connect-group  
      CONNECT_CONFIG_STORAGE_REPLICATION_FACTOR: 1  
      CONNECT_OFFSET_FLUSH_INTERVAL_MS: 10000  
      CONNECT_OFFSET_STORAGE_TOPIC: docker-connect-offsets  
      CONNECT_OFFSET_STORAGE_REPLICATION_FACTOR: 1  
      CONNECT_STATUS_STORAGE_TOPIC: docker-connect-status  
      CONNECT_STATUS_STORAGE_REPLICATION_FACTOR: 1  
      CONNECT_CONFIG_STORAGE_TOPIC: docker-connect-storage  
      CONNECT_KEY_CONVERTER: org.apache.kafka.connect.json.JsonConverter  
      CONNECT_VALUE_CONVERTER: org.apache.kafka.connect.json.JsonConverter  
      CONNECT_PLUGIN_PATH: '/jars2,/jars'  
      KAFKA_ADVERTISED_LISTENERS: SASL_SSL://<MSK_CLUSTER_BOOTSTRAP_SERVER>  
      CONNECT_REQUEST_TIMEOUT_MS: 120000  
      TRACE: true  
      CONNECT_SECURITY_PROTOCOL: "SASL_SSL"  
      CONNECT_SASL_MECHANISM: "AWS_MSK_IAM"  
      CONNECT_SASL_JAAS_CONFIG: "software.amazon.msk.auth.iam.IAMLoginModule required;"  
      CONNECT_SASL_CLIENT_CALLBACK_HANDLER_CLASS: "software.amazon.msk.auth.iam.IAMClientCallbackHandler"  

```

The above file contains all the environment variables necessary to get kafka-connect up and running. The variables related to MSK authentication are

```
 CONNECT_SECURITY_PROTOCOL: "SASL_SSL"  
      CONNECT_SASL_MECHANISM: "AWS_MSK_IAM"  
      CONNECT_SASL_JAAS_CONFIG: "software.amazon.msk.auth.iam.IAMLoginModule required;"  
      CONNECT_SASL_CLIENT_CALLBACK_HANDLER_CLASS: "software.amazon.msk.auth.iam.IAMClientCallbackHandler"
```

**My Debugging Process :**  
Even though it is just few lines I had to do a lot of research for googling and debugging for this. I would like to discuss how I got there

I tried multiple ways but kept getting timeout error I wasn’t not sure what’s wrong . I added the jar file and I was using below environment variables.

```
SECURITY_PROTOCOL  
SASL_MECHANISM  
SASL_JASS_CONFIG  
SASL_CLIENT_CLIENT_HANDLER_CLASS
```

I didn’t find any documentation to use the right variables. so I got into the container using bash and when to the directory mentioned in CMD of the dockerfile I found in the dockerhub for cp-connect-base

The location is “ /etc/confluent/docker/run”

Below is the files in the folder /etc/confluent/docker/run

I did cat on run file and the output is this

```
#!/usr/bin/env bash  
#  
# Copyright 2016 Confluent Inc.  
#  
# Licensed under the Apache License, Version 2.0 (the "License");  
# you may not use this file except in compliance with the License.  
# You may obtain a copy of the License at  
#  
# http://www.apache.org/licenses/LICENSE-2.0  
#  
# Unless required by applicable law or agreed to in writing, software  
# distributed under the License is distributed on an "AS IS" BASIS,  
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.  
# See the License for the specific language governing permissions and  
# limitations under the License.  
  
. /etc/confluent/docker/bash-config  
  
. /etc/confluent/docker/mesos-setup.sh  
. /etc/confluent/docker/apply-mesos-overrides  
  
echo "===> User"  
id  
  
echo "===> Configuring ..."  
/etc/confluent/docker/configure  
  
echo "===> Running preflight checks ... "  
/etc/confluent/docker/ensure  
  
echo "===> Launching ... "  
exec /etc/confluent/docker/launch
```

From the file it’s obvious what’s happening

I did cat on all and found scripts “bash-config” and “configure” are important.

bash-config

```
#  
# Copyright 2018 Confluent Inc.  
#  
# Licensed under the Apache License, Version 2.0 (the "License");  
# you may not use this file except in compliance with the License.  
# You may obtain a copy of the License at  
#  
# http://www.apache.org/licenses/LICENSE-2.0  
#  
# Unless required by applicable law or agreed to in writing, software  
# distributed under the License is distributed on an "AS IS" BASIS,  
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.  
# See the License for the specific language governing permissions and  
# limitations under the License.  
  
set -o nounset \
    -o errexit  
  
# Trace may expose passwords/credentials by printing them to stdout, so turn on with care.  
if [ "${TRACE:-}" == "true" ]; then  
  set -o verbose \
      -o xtrace  
fi
```

This script runs in verbose mode when TRACE is set to true. I used that to understand each step that’s happening even the command runs.

configure

```
#!/usr/bin/env bash  
#  
# Copyright 2016 Confluent Inc.  
#  
# Licensed under the Apache License, Version 2.0 (the "License");  
# you may not use this file except in compliance with the License.  
# You may obtain a copy of the License at  
#  
# http://www.apache.org/licenses/LICENSE-2.0  
#  
# Unless required by applicable law or agreed to in writing, software  
# distributed under the License is distributed on an "AS IS" BASIS,  
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.  
# See the License for the specific language governing permissions and  
# limitations under the License.  
  
. /etc/confluent/docker/bash-config  
  
dub ensure CONNECT_BOOTSTRAP_SERVERS  
dub ensure CONNECT_GROUP_ID  
dub ensure CONNECT_CONFIG_STORAGE_TOPIC  
dub ensure CONNECT_OFFSET_STORAGE_TOPIC  
dub ensure CONNECT_STATUS_STORAGE_TOPIC  
dub ensure CONNECT_KEY_CONVERTER  
dub ensure CONNECT_VALUE_CONVERTER  
# This is required to avoid config bugs. You should set this to a value that is  
# resolvable by all containers.  
dub ensure CONNECT_REST_ADVERTISED_HOST_NAME  
  
# Default to 8083, which matches the mesos-overrides. This is here in case we extend the containers to remove the mesos overrides.  
if [ -z "$CONNECT_REST_PORT" ]; then  
  export CONNECT_REST_PORT=8083  
fi  
  
# Fix for https://issues.apache.org/jira/browse/KAFKA-3988  
if [[ ${CONNECT_INTERNAL_KEY_CONVERTER-} == "org.apache.kafka.connect.json.JsonConverter" ]] || [[ ${CONNECT_INTERNAL_VALUE_CONVERTER-} == "org.apache.kafka.connect.json.JsonConverter" ]]  
then  
  export CONNECT_INTERNAL_KEY_CONVERTER_SCHEMAS_ENABLE=false  
  export CONNECT_INTERNAL_VALUE_CONVERTER_SCHEMAS_ENABLE=false  
fi  
  
if [[ $CONNECT_KEY_CONVERTER == "io.confluent.connect.avro.AvroConverter" ]]  
then  
  dub ensure CONNECT_KEY_CONVERTER_SCHEMA_REGISTRY_URL  
fi  
  
if [[ $CONNECT_VALUE_CONVERTER == "io.confluent.connect.avro.AvroConverter" ]]  
then  
  dub ensure CONNECT_VALUE_CONVERTER_SCHEMA_REGISTRY_URL  
fi  
  
dub path /etc/"${COMPONENT}"/ writable  
  
dub template "/etc/confluent/docker/${COMPONENT}.properties.template" "/etc/${COMPONENT}/${COMPONENT}.properties"  
  
# The connect-distributed script expects the log4j config at /etc/kafka/connect-log4j.properties.  
dub template "/etc/confluent/docker/log4j.properties.template" "/etc/kafka/connect-log4j.properties"
```

This script ensure that it fails if all the necessary env. variables are not set.  
It also talks creates properties file . This information was useful for me to know which env. variables are turned into properties and what are not .

```
dub template "/etc/confluent/docker/${COMPONENT}.properties.template" "/etc/${COMPONENT}/${COMPONENT}.properties"  
  
# The connect-distributed script expects the log4j config at /etc/kafka/connect-log4j.properties.  
dub template "/etc/confluent/docker/log4j.properties.template" "/etc/kafka/connect-log4j.properties"
```

> Infact after studying the connect.properties.template I came to know that I have to add CONNECT for the properties to be picked by kafka-connect.

In this way I was able to debug and find the correct values to pass and the changes to be made on docker image to connect to MSK

similarly I figured out for schema-registry. Below is the Dockerfile and docker-compose.yaml file

```
# Inject AWS IAM Auth to Confluent image  
FROM confluentinc/cp-schema-registry:7.2.2  
  
USER root  
ADD --chown=appuser:appuser https://github.com/aws/aws-msk-iam-auth/releases/download/v1.1.4/aws-msk-iam-auth-1.1.4-all.jar /usr/share/java/cp-base-new/  
ADD --chown=appuser:appuser https://github.com/aws/aws-msk-iam-auth/releases/download/v1.1.4/aws-msk-iam-auth-1.1.4-all.jar /usr/share/java/schema-registry/  
  
ENV SCHEMA_REGISTRY_KAFKASTORE_SECURITY_PROTOCOL="SASL_SSL"  
ENV SCHEMA_REGISTRY_KAFKASTORE_SASL_MECHANISM="AWS_MSK_IAM"  
ENV SCHEMA_REGISTRY_KAFKASTORE_SASL_JAAS_CONFIG="software.amazon.msk.auth.iam.IAMLoginModule required;"  
ENV SCHEMA_REGISTRY_KAFKASTORE_SASL_CLIENT_CALLBACK_HANDLER_CLASS="software.amazon.msk.auth.iam.IAMClientCallbackHandler"  
  
USER appuser  
  
EXPOSE 8081
```

docker-compose.yaml

```
version: "3.5"  
services:  
  schema-registry:  
    image: sr:latest  
    hostname: localhost:schema-registry  
    container_name: schema-registry  
    ports:  
      - "0.0.0.0:8081:8081"  
    volumes:  
      - "./data:/topic_data"  
    environment:  
      SCHEMA_REGISTRY_HOST_NAME: schema-registry  
      SCHEMA_REGISTRY_KAFKASTORE_BOOTSTRAP_SERVERS: "<BOOTSTRAP_SERVER>"  
      SCHEMA_REGISTRY_LISTENERS: http://0.0.0.0:8081  
      SR_HOSTNAME: schema-registry  
      SR_LISTENERS: http://schema-registry:8081  
      SR_DEBUG: 'true'  
      SR_KAFKASTORE_TOPIC_SERVERS: SASL_SSL://<BOOTSTRAP_SERVER>  
      SCHEMA_REGISTRY_LOG4J_LOGGERS: "org.apache.kafka=ERROR,io.confluent.rest.exceptions=FATAL"  
      SCHEMA_REGISTRY_LOG4J_ROOT_LOGLEVEL: DEBUG
```

Overall, I was presented with a great opportunity to work on this new technology . I learnt a lot about working with kafka — MSK , kafk-connect and schema registry . How all work together.