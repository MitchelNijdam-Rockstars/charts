# Data Platform Blueprint 1 with Kafka-Spark-Solr

Enterprise applications increasingly rely on large amounts of data, that needs be distributed, processed, and stored.
Open source and commercial supported software stacks are available to implement a data platform, that can offer
common data management services, accelerating the development and deployment of data hungry business applications.

This Helm chart enables the fully automated Kubernetes deployment of such multi-stack data platform, covering the following software components:

-   Apache Kafka – Data distribution bus with buffering capabilities
-   Apache Spark – In-memory data analytics
-   Solr – Data persistence and search

These containerized stateful software stacks are deployed in multi-node cluster configurations, which is defined by the
Helm chart blueprint for this data platform deployment, covering:

-   Pod placement rules – Affinity rules to ensure placement diversity to prevent single point of failures and optimize load distribution
-   Pod resource sizing rules – Optimized Pod and JVM sizing settings for optimal performance and efficient resource usage
-   Default settings to ensure Pod access security

In addition to the Pod resource optimizations, this blueprint is validated and tested to provide Kubernetes node count and sizing recommendations [(see Kubernetes Cluster Requirements)](#kubernetes-cluster-requirements) to facilitate cloud platform capacity planning. The goal is optimize the number of required Kubernetes nodes in order to optimize server resource usage and, at the same time, ensuring runtime and resource diversity.

The first release of this blueprint defines a small size data platform deployment, deployed on 3 Kubernetes application nodes with physical diverse underlying server infrastructure.

Use cases for this small size data platform setup include: data and application evaluation, development, and functional testing.

## TL;DR

```console
$ helm repo add bitnami https://charts.bitnami.com/bitnami
$ helm install my-release bitnami/dataplatform-bp1
```

## Introduction

This chart bootstraps Data Platform Blueprint-1 deployment on a [Kubernetes](http://kubernetes.io) cluster using the [Helm](https://helm.sh) package manager.

The "Small" size data platform in default configuration deploys the following:
1. Zookeeper with 3 nodes to be used for both Kafka and Solr
2. Kafka with 3 nodes using the zookeeper deployed above
3. Solr with 2 nodes using the zookeeper deployed above
4. Spark with 1 Master and 2 worker nodes

Bitnami charts can be used with [Kubeapps](https://kubeapps.com/) for deployment and management of Helm Charts in clusters. This Helm chart has been tested on top of [Bitnami Kubernetes Production Runtime](https://kubeprod.io/) (BKPR). Deploy BKPR to get automated TLS certificates, logging and monitoring for your applications.

## Prerequisites

- Kubernetes 1.12+
- Helm 3.1.0
- PV provisioner support in the underlying infrastructure

## Kubernetes Cluster requirements

Below are the minimum Kubernetes Cluster requirements for "Small" size data platform:

| Data Platform Size | Kubernetes Cluster Size                                                      | Usage                                                                       |
|:-------------------|:-----------------------------------------------------------------------------|:----------------------------------------------------------------------------|
| Small              | 1 Master Node (2 CPU, 4Gi Memory) <br /> 3 Worker Nodes (4 CPU, 32Gi Memory) | Data and application evaluation, development, and functional testing <br /> |

## Installing the Chart

To install the chart with the release name `my-release`:

```console
$ helm repo add bitnami https://charts.bitnami.com/bitnami
$ helm install my-release bitnami/dataplatform-bp1
```

These commands deploy Data Platform on the Kubernetes cluster in the default configuration. The [Parameters](#parameters) section lists recommended configurations of the parameters to bring up an optimal and resilient data platform. Please refer the individual charts for the remaining set of configurable parameters.

> **Tip**: List all releases using `helm list`

## Uninstalling the Chart

To uninstall/delete the `my-release` deployment:

```console
$ helm delete my-release
```

The command removes all the Kubernetes components associated with the chart and deletes the release.

## Parameters

The following tables lists the recommended configurations for each application used in the data platform. If you need to configure any other parameters apart from the ones mentioned below, you can refer to the corresponding chart and update the values.yaml accordingly.

### Global parameters

| Parameter                 | Description                                     | Default                                                 |
|:--------------------------|:------------------------------------------------|:--------------------------------------------------------|
| `global.imageRegistry`    | Global Docker image registry                    | `nil`                                                   |
| `global.imagePullSecrets` | Global Docker registry secret names as an array | `[]` (does not add image pull secrets to deployed pods) |
| `global.storageClass`     | Global storage class for dynamic provisioning   | `nil`                                                   |

### Zookeeper chart parameters

Parameters below are set as per the recommended values, they can be overwritten if required.

| Parameter                      | Description                                                                     | Default                                                                              |
|:-------------------------------|:--------------------------------------------------------------------------------|:-------------------------------------------------------------------------------------|
| `zookeeper.enabled`            | Switch to enable or disable the Zookeeper helm chart                            | `true`                                                                               |
| `zookeeper.replicaCount`       | Number of Zookeeper nodes                                                       | `3`                                                                                  |
| `zookeeper.heap`               | Zookeepers's Java Heap size                                                     | Zookeeper Java Heap size set for optimal resource usage                              |
| `zookeeper.resources.limits`   | The resources limits for Zookeeper containers                                   | `{}`                                                                                 |
| `zookeeper.resources.requests` | The requested resources for Zookeeper containers for a small kubernetes cluster | Zookeeper pods Resource requests for optimal resource usage size                     |
| `zookeeper.affinity`           | Affinity for pod assignment                                                     | Zookeeper pods Affinity rules for best possible resiliency (evaluated as a template) |

### Kafka chart parameters

Parameters below are set as per the recommended values, they can be overwritten if required.

| Parameter                   | Description                                                                 | Default                                                                              |
|:----------------------------|:----------------------------------------------------------------------------|:-------------------------------------------------------------------------------------|
| `kafka.enabled`             | Switch to enable or disable the Kafka helm chart                            | `true`                                                                               |
| `kafka.replicaCount`        | Number of Kafka nodes                                                       | `3`                                                                                  |
| `kafka.heapOpts`            | Kafka's Java Heap size                                                      | Kafka Java Heap size set for optimal resource usage                                  |
| `kafka.resources.limits`    | The resources limits for Kafka containers                                   | `{}`                                                                                 |
| `kafka.resources.requests`  | The requested resources for Kafka containers for a small kubernetes cluster | Kafka pods Resource requests set for optimal resource usage                          |
| `kafka.affinity`            | Affinity for pod assignment                                                 | Kafka pods affinity rules set for best possible resiliency (evaluated as a template) |
| `kafka.zookeeper.enabled`   | Switch to enable or disable the Zookeeper helm chart                        | `false` Common Zookeeper deployment used for kafka and solr                          |
| `kafka.externalZookeeper.servers` | Server or list of external Zookeeper servers to use                         | Zookeeper installed as a subchart to be used                                         |

### Solr chart parameters

Parameters below are set as per the recommended values, they can be overwritten if required.

| Parameter                        | Description                                         | Default                                                                             |
|:---------------------------------|:----------------------------------------------------|:------------------------------------------------------------------------------------|
| `solr.enabled`                   | Switch to enable or disable the Solr helm chart     | `true`                                                                              |
| `solr.replicaCount`              | Number of Solr nodes                                | `2`                                                                                 |
| `solr.authentication.enabled`    | Enable Solr authentication                          | `true`                                                                              |
| `solr.resources.limits`          | The resources limits for Solr containers            | `{}`                                                                                |
| `solr.resources.requests`        | The requested resources for Solr containers         | Solr pods resource requests set for optimal resource usage                          |
| `solr.javaMem`                   | Java memory options to pass to the Solr container   | Solr Java Heap size set for optimal resource usage                                  |
| `solr.heap`                      | Java Heap options to pass to the solr container     | `nil`                                                                               |
| `solr.affinity`                  | Affinity for Solr pods assignment                   | Solr pods Affinity rules set for best possible resiliency (evaluated as a template) |
| `solr.zookeeper.enabled`         | Enable Zookeeper deployment. Needed for Solr cloud. | `false` common zookeeper used between kafka and solr                                |
| `solr.externalZookeeper.servers` | Servers for an already existing Zookeeper.          | Zookeeper installed as a subchart to be used                                        |

### Spark chart parameters

Parameters below are set as per the recommended values, they can be overwritten if required.

| Parameter                   | Description                                      | Default                                                                                     |
|:----------------------------|:-------------------------------------------------|:--------------------------------------------------------------------------------------------|
| `spark.enabled`             | Switch to enable or disable the Spark helm chart | `true`                                                                                      |
| `spark.master.affinity`     | Spark master affinity for pod assignment         | Spark master pod Affinity rules set for best possible resiliency (evaluated as a template)  |
| `spark.master.resources`    | CPU/Memory resource requests/limits for Master   | Spark master pods resource requests set for optimal resource usage                          |
| `spark.worker.javaOptions`  | Set options for the JVM in the form `-Dx=y`      | No default                                                                                  |
| `spark.worker.replicaCount` | Set the number of workers                        | `2`                                                                                         |
| `spark.worker.affinity`     | Spark worker affinity for pod assignment         | Spark worker pods Affinity rules set for best possible resiliency (evaluated as a template) |
| `spark.worker.resources`    | CPU/Memory resource requests/limits for worker   | Spark worker pods resource requests set for optimal resource usage                          |

Specify each parameter using the `--set key=value[,key=value]` argument to `helm install`. For example,

```console
$ helm install my-release \
  --set kafka.replicaCount=3 \
  bitnami/dataplatform-bp1
```

The above command deploys the data platform with Kafka with 3 nodes (replicas).

In case you need to deploy the data platform skipping any component, you can specify the 'enabled' parameter using the `--set <component>.enabled=false` argument to `helm install`. For Example,

```console
$ helm install my-release \
  --set solr.enabled=false \
  bitnami/dataplatform-bp1
```

The above command deploys the data platform without Solr.

Alternatively, a YAML file that specifies the values for the above parameters can be provided while installing the chart. For example,

```console
$ helm install my-release -f values.yaml bitnami/dataplatform-bp1
```

> **Tip**: You can use the default [values.yaml](values.yaml)

## Configuration and installation details

### [Rolling VS Immutable tags](https://docs.bitnami.com/containers/how-to/understand-rolling-tags-containers/)

It is strongly recommended to use immutable tags in a production environment. This ensures your deployment does not change automatically if the same tag is updated with a different image.

Bitnami will release a new chart updating its containers if a new version of the main container, significant changes, or critical vulnerabilities exist.

## Troubleshooting

Find more information about how to deal with common errors related to Bitnami’s Helm charts in [this troubleshooting guide](https://docs.bitnami.com/general/how-to/troubleshoot-helm-chart-issues).

In order to render complete information about the deployment including all the sub-charts, please use --render-subchart-notes flag while installing the chart.
