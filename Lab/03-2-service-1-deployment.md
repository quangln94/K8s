# Deploy 2 service sử dụng cùng 1 deployment
**Chỉnh sử file `nginx_deployment_1.yaml` với nội dung sau:**
```sh
apiVersion: apps/v1
kind: Deployment
metadata:
  name: sample-deployment-1
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
        label: two
    spec:
      containers:
        - name: nginx-container
          image: nginx:1.12
          ports:
            - containerPort: 80
```
**Chỉnh sửa file `nginx_service_1.yaml` với nội dung sau:**
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
**Chỉnh sửa file `nginx_service_2.yaml` với nội dung sau:**
```sh
apiVersion: v1
kind: Service
metadata:
  name: mywebservice
spec:
  selector:
    app: nginx
    label: two
  ports:
  - port: 80
    targetPort: 80
  type: NodePort
```
**Thực hiện deploy app sử dụng cả 2 service**
```sh
kubectl apply -f nginx_deployment_1.yaml
kubectl apply -f nginx_service_1.yaml
kubectl apply -f nginx_service_2.yaml
```
**Kiểm tra service**
```sh
[root@server01 ~]# kubectl get svc
NAME             TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
kubernetes       ClusterIP   10.96.0.1        <none>        443/TCP        10h
mywebservice01   NodePort    10.106.147.245   <none>        80:32429/TCP   9h
mywebservice02   NodePort    10.103.5.229     <none>        80:31774/TCP   9h
```
- Có 2 `service với port khác nhau

**Kiểm tra service sẽ được gán trên các Pod nào:
```sh
[root@server01 ~]# kubectl describe service mywebservice01
Name:                     mywebservice01
Namespace:                default
Labels:                   <none>
Annotations:              kubectl.kubernetes.io/last-applied-configuration:
                            {"apiVersion":"v1","kind":"Service","metadata":{"annotations":{},"name":"mywebservice01","namespace":"default"},"spec":{"ports":[{"port":8...
Selector:                 app=nginx
Type:                     NodePort
IP:                       10.106.147.245
Port:                     <unset>  80/TCP
TargetPort:               80/TCP
NodePort:                 <unset>  32429/TCP
Endpoints:                10.244.1.6:80,10.244.2.6:80,10.244.2.7:80
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>
```
```sh
[root@server01 ~]# kubectl describe service mywebservice02
Name:                     mywebservice02
Namespace:                default
Labels:                   <none>
Annotations:              kubectl.kubernetes.io/last-applied-configuration:
                            {"apiVersion":"v1","kind":"Service","metadata":{"annotations":{},"name":"mywebservice02","namespace":"default"},"spec":{"ports":[{"port":8...
Selector:                 app=nginx,label=two
Type:                     NodePort
IP:                       10.103.5.229
Port:                     <unset>  80/TCP
TargetPort:               80/TCP
NodePort:                 <unset>  31774/TCP
Endpoints:                10.244.1.6:80,10.244.2.6:80,10.244.2.7:80
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>
```
- Cả 2 service cùng chạy trên các Pod giống nhau
