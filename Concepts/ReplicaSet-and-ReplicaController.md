# Difference between a replica set and replication controller

Replica set and replication controller - Both the terms have the word replica. Why do we need to replicate anything? Let's start with that. 

There are multiple ways your container can crash. Replication is used for the core purpose of Reliability, Load Balancing, and Scaling.

There are two main types of Replications in Kubernetes - Replica sets and Replication controller.

The replication controller makes sure that few pre-defined pods always exist. So in case of a pod crashes, the replication controller replaces it.
```sh
apiVersion: v1
kind: ReplicationController
metadata:
  name: example
spec:
  replicas: 3
  selector:
    app: example
  template:
    metadata:
      name: example
      labels:
        app: example
    spec:
      containers:
      - name: example
        image: example/rc
        ports:
        - containerPort: 80
```
Replica sets are comparatively more useful. In recent times replication sets have replaced replication controllers. What is so special about them? Let's have a look

Replica sets have a few more functionalities when compared to the replication controller. 
```
apiVersion: extensions/v1beta1
 kind: ReplicaSet
 metadata:
   name: example
 spec:
   replicas: 3
   selector:
     matchLabels:
       app: example
   template:
     metadata:
       labels:
         app: example
         environment: dev
     spec:
       containers:
       - name: example
         image: example/rs
         ports:
         - containerPort: 80
```
In this case, we are using match labels instead of labels which could easily be written like this:
```sh
...
spec:
   replicas: 3
   selector:
     matchExpressions:
      - {key: app, operator: In, values: [example, example, rs]}
      - {key: teir, operator: NotIn, values: [production]}
  template:
     metadata:
...
```
## Tài liệu tham khảo
- https://www.edureka.co/community/43891/difference-between-replica-set-and-replication-controller?fbclid=IwAR0ytDfg-TCBP1lA1nvgKd7d913pH1sTnF3N8yr7pQL93T-T-lMq2WV1hr8
