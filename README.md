# Kubernetes Logging with Fluent Bit v0.13

This repository is a modification of FluentBit v0.13 repo which specific instructions on 
using the elasticsearch ds in the output directory

There are NO modifications needed for this to work on VKE. Its been tested with VKE v1.10-67

NOTE: This has been tested with AWS Elasticseach in a public configuration for ease of use.

- Please use the standard ways to secure elasticsearch per AWS documentation.
- Two such options
- include using a proxy when elasticsearch is in a VPC deployment
- using Cognito to configure user/password for access into AWS elasticsearch in the public config.

[Fluent Bit](http://fluentbit.io) is a lightweight and extensible __Log Processor__ that comes with full support for Kubernetes:

- Read Kubernetes/Docker log files from the file system or through Systemd Journal
- Enrich logs with Kubernetes metadata
- Deliver logs to third party storage services like Elasticsearch, InfluxDB, HTTP, etc.

This repository contains a set of Yaml files to deploy Fluent Bit which consider namespace, RBAC, Service Account, etc.

## Getting started

[Fluent Bit](http://fluentbit.io) must be deployed as a DaemonSet, so on that way it will be available on every node of your Kubernetes cluster. To get started run the following commands to create the namespace, service account and role setup:

```
$ kubectl create namespace logging
$ kubectl create -f fluent-bit-service-account.yaml
$ kubectl create -f fluent-bit-role.yaml
$ kubectl create -f fluent-bit-role-binding.yaml
```

#### Fluent Bit to Elasticsearch

The next step is to create a ConfigMap that will be used by our Fluent Bit DaemonSet.
We will be using the standard configmap from the fluentbit v0.13 repo with NO modifications:

```
$ kubectl create -f ./output/elasticsearch/fluent-bit-configmap.yaml
```

Fluent Bit DaemonSet ready to be used with Elasticsearch on VMware Kubernetes Engine Cluster

Please ensure you replace the following values in the fluent-bit-ds.yaml file

```
- name: FLUENT_ELASTICSEARCH_HOST
  value: "<ENTER AWS ELASTICSEARCH URL HERE>"
- name: FLUENT_ELASTICSEARCH_PORT
  value: "<ENTER AWS ELASTICSEARCH PORT HERE - GENERALLY - 443>"
```

```
$ kubectl create -f ./output/elasticsearch/fluent-bit-ds.yaml
```

## Details

The default configuration of Fluent Bit makes sure of the following:

- Consume all containers logs from the running Node.
- The [Tail input plugin](http://fluentbit.io/documentation/0.12/input/tail.html) will not append more than __5MB__  into the engine until they are flushed to the Elasticsearch backend. This limit aims to provide a workaround for [backpressure](http://fluentbit.io/documentation/0.13/configuration/backpressure.html) scenarios.
- The Kubernetes filter will enrigh the logs with Kubernetes metadata, specifically _labels_ and _annotations_. The filter only goes to the API Server when it cannot find the cached info, otherwise it uses the cache.
- The default backend in the configuration is Elasticsearch set by the [Elasticsearch Ouput Plugin](http://fluentbit.io/documentation/0.13/output/elasticsearch.html). It uses the Logstash format to ingest the logs. If you need a different Index and Type, please refer to the plugin option and do your own adjustments.
- There is an option called __Retry_Limit__ set to False, that means if Fluent Bit cannot flush the records to Elasticsearch it will re-try indefinitely until it succeed.

## Get back to the authers!

Visit https://github.com/fluent/fluent-bit-kubernetes-logging

Your contribution on testing is highly appreciated, we aim to make logging cheaper for everybody so your feedback is fundamental, try to get back to us on:

- [Mailing List / Google Group](https://groups.google.com/forum/#!forum/fluent-bit)
- [Slack Channel #fluent-bit](http://slack.fluentd.org)
