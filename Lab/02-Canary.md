#
## 1 Service và 2 Deployment cùng label
**Chỉnh sử file `nginx_deployment_stable.yaml` với nội dung sau:**
```sh
apiVersion: apps/v1
kind: Deployment
metadata:
  name: sample-deployment-stable
spec:
  replicas: 6
  strategy:
    type: Recreate
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx-container
          image: nginx:1.12
          ports:
            - containerPort: 80
```
**Chỉnh sửa file `nginx_deployment_canary.yaml` với nội dung sau:**
```sh
apiVersion: apps/v1
kind: Deployment
metadata:
  name: sample-deployment-canary
spec:
  replicas: 3
  strategy:
    type: Recreate
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx-container
          image: nginx:1.13
          ports:
            - containerPort: 80
```
**Chỉnh sửa file `nginx_service_canary.yaml` với nội dung sau:**
```sh
apiVersion: v1
kind: Service
metadata:
  name: mywebservice
spec:
  selector:
    app: nginx
  ports:
  - port: 80
    targetPort: 80
  type: NodePort
```
**Thực hiện deploy app sử dụng cả 2 version image**
```sh
kubectl apply -f nginx_deployment_canary.yaml
kubectl apply -f nginx_deployment_stable.yaml
```
**Kiểm tra service**
```sh
[root@server01 ~]# kubectl get svc
NAME           TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
kubernetes     ClusterIP   10.96.0.1       <none>        443/TCP        7d2h
mywebservice   NodePort    10.102.19.26    <none>        80:32543/TCP   5h7m
test-nginx     NodePort    10.110.236.21   <none>        80:31566/TCP   6d23h
```
***Chỉ có 1 service `mywebservice` gồm 9 Replica***
```sh
[root@server01 ~]# kubectl describe service mywebservice
Name:                     mywebservice
Namespace:                default
Labels:                   <none>
Annotations:              kubectl.kubernetes.io/last-applied-configuration:
                            {"apiVersion":"v1","kind":"Service","metadata":{"annotations":{},"name":"mywebservice","names                                                                                      pace":"default"},"spec":{"ports":[{"port":80,...
Selector:                 app=nginx
Type:                     NodePort
IP:                       10.102.19.26
Port:                     <unset>  80/TCP
TargetPort:               80/TCP
NodePort:                 <unset>  32543/TCP
Endpoints:                10.244.1.18:80,10.244.1.19:80,10.244.1.20:80 + 6 more...
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>
[root@server01 ~]# kubectl describe service mywebservice
Name:                     mywebservice
Namespace:                default
Labels:                   <none>
Annotations:              kubectl.kubernetes.io/last-applied-configuration:
                            {"apiVersion":"v1","kind":"Service","metadata":{"annotations":{},"name":"mywebservice","namespace":"default"},"spec":{"ports":[{"port":80,...
Selector:                 app=nginx
Type:                     NodePort
IP:                       10.102.19.26
Port:                     <unset>  80/TCP
TargetPort:               80/TCP
NodePort:                 <unset>  32543/TCP
Endpoints:                10.244.1.18:80,10.244.1.19:80,10.244.1.20:80 + 6 more...
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>
```
