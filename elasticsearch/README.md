# Introduction
This file shows you how to install witix on Google Cloud Plataform (GCP), using Container Engine, Kubernetes and Docker

## Before
* Install [gcloud sdk](https://cloud.google.com/sdk/docs/quickstart-linux). 
* Install [Kubernetes](http://kubernetes.io/docs/getting-started-guides/binary_release/)
* Install [Docker](https://docs.docker.com/engine/installation/)

## Baby Steps
1. Go to the Google Cloud Platform Console.
1. Create or select a project (PROJECT ID).
1. Click Continue to enable the API and any related services. This step can take several minutes.
1. Enable billing for your project. (trial 300$ for 3 months)

The next steps we will reffer to google project as *cit-labs-witix* 

# Build
Witix images are avaliable on (docker hub)[https://hub.docker.com/r/ciandtsoftware], but if you want to build your own imagens, take a look at [build imagens link!](../../witix-docker/README.md)

# Deploy on Google Container Engine

## Setup
Creating kubernete cluster on google container

```shell
gcloud config set compute/zone us-central1-b
gcloud init
gcloud container clusters create witix-elasticsearch-cluster
```

If cluster has already exists, you can use command bellow:
```shell
gcloud container clusters get-credentials <CLUSTER-NAME>
```

## Deploy

```
kubectl create -f service-account.yaml
kubectl create -f es-discovery-svc.yaml
kubectl create -f es-master-rc.yaml
```

Wait until `es-master` is provisioned, and

```
kubectl create -f es-svc.yaml
kubectl create -f es-client-rc.yaml
```

Wait until `es-client` is provisioned, and

```
kubectl create -f es-data-rc.yaml
```

Wait until `es-data` is provisioned.

Now, I leave up to you how to validate the cluster, but a first step is to wait for containers to be in the `Running` state and check Elasticsearch master logs:

```console
$ kubectl get svc,rc,pods
NAME                      CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
elasticsearch             10.100.202.107                 9200/TCP   10m
elasticsearch-discovery   10.100.99.125    <none>        9300/TCP   10m
kubernetes                10.100.0.1       <none>        443/TCP    21h
NAME                      DESIRED          CURRENT       AGE
es-client                 1                1             5m
es-data                   1                1             4m
es-master                 1                1             7m
NAME                      READY            STATUS        RESTARTS   AGE
es-client-5b1oc           1/1              Running       0          5m
es-data-0s6eg             1/1              Running       0          4m
es-master-tile7           1/1              Running       0          7m
```
### Scale

Scaling each type of node to handle your cluster is as easy as:

```
kubectl scale --replicas 2 rc/es-data
```

You can use this commands to help you during troubleshooting 
```
$ kubectl logs -f es-master-tile7
$ kubectl exec es-master-tile7 -- printenv
# show hostIP for label component=elasticsearch
$ kubectl get pods -l component=elasticsearch -o json | grep hostIP
```

### Access the service
*Don't forget* that services in Kubernetes are only acessible from containers in the cluster. For different behavior you should [configure the creation of an external load-balancer](http://kubernetes.io/v1.1/docs/user-guide/services.html#type-loadbalancer). While it's supported within this example service descriptor, its usage is out of scope of this document, for now.

```console
$ kubectl get svc elasticsearch
NAME            CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
elasticsearch   10.100.202.107  <???????????>  9200/TCP   13m
```

```
curl http://<EXTERNAL-IP:9200
```

You should see something similar to the following:

```json
{
  "name" : "Hitman",
  "cluster_name" : "myesdb",
  "version" : {
    "number" : "2.3.2",
    "build_hash" : "b9e4a6acad4008027e4038f6abed7f7dba346f94",
    "build_timestamp" : "2016-04-21T16:03:47Z",
    "build_snapshot" : false,
    "lucene_version" : "5.5.0"
  },
  "tagline" : "You Know, for Search"
}
```


# Clean Up
```shell
kubectl delete -f  es-data-rc.yaml
kubectl delete -f  es-client-rc.yaml
kubectl delete -f  es-master-rc.yaml
kubectl delete -f  es-discovery-svc.yaml
kubectl delete -f  es-svc.yaml
kubectl delete -f  service-account.yaml

gcloud container clusters delete witix-elasticsearch-cluster
```

If you want you can delete project using this command:
```shell
gsutil ls
gsutil rm -r gs://artifacts.<PROJECT_ID>.appspot.com/
```

# References: 
https://cloud.google.com/container-engine/docs/quickstart#run_a_container_image
http://kubernetes.io/docs/hellonode/
https://github.com/kubernetes/kubernetes/tree/master/examples/elasticsearch/production_cluster