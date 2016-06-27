## Pre-req
* Make elasticsearch config first. See [Readme!](../elasticsearch/README.md)


## Deploy

```
kubectl create -f kibana-svc.yaml
kubectl create -f kibana-rc.yaml
```


### Access the service

```
$ kubectl get svc witix-kibana-svc
NAME               CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
witix-kibana-svc   10.100.202.107  <???????????>  9200/TCP   13m
```

```
Open this url http://<EXTERNAL-IP:5601 on your browser
```

You should see something similar to the following:
<TODO - image>


# Clean Up
```shell
kubectl delete -f  kibana-svc.yaml
kubectl delete -f  kibana-rc.yaml
```