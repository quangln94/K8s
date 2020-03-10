# Triển khai ứng dụng sử dụng metallb làm Loadbalancer

## 1. Mô hình
- Cụm k8s gồm:

|Server|Master 01|Worker 01|
|------|---------|---------|
IP|192.168.1.11|192.168.1.21|
	
## 2. Cài đặt và cấu hình

- Thực hiện trên Node Master:

Tải và cài đặt `metallb`
```
mkdir -p ~/metallb
cd metallb
wget https://raw.githubusercontent.com/google/metallb/v0.8.3/manifests/metallb.yaml
```
- Apply file `metallb.yaml`

```
kubectl apply -f metallb.yaml
```
- Check MetalLB running chưa:

```
kubectl get pods -n metallb-system
NAME                          READY   STATUS    RESTARTS   AGE
controller-6bcfdfd677-shmqx   1/1     Running   0          33m
speaker-d797d                 1/1     Running   0          32m
```
- Config MetalLB. Tạo file config.yaml 
```
apiVersion: v1
kind: ConfigMap
metadata:
  namespace: metallb-system
  name: config
data:
  config: |
    address-pools:
    - name: default
      protocol: layer2
      addresses:
      - 192.168.1.30-192.168.1.40   # Dải IP làm LoadBalancer
```
- Apply file confile metallb
```
kubectl apply -f config.yaml
```
- Tạo file `hello-kubernetes-first.yaml`
```
apiVersion: v1
kind: Service
metadata:
  name: hello-kubernetes-first
spec:
  type: LoadBalancer
  ports:
  - port: 80
    targetPort: 8080
  selector:
    app: hello-kubernetes-first
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-kubernetes-first
spec:
  replicas: 3
  selector:
    matchLabels:
      app: hello-kubernetes-first
  template:
    metadata:
      labels:
        app: hello-kubernetes-first
    spec:
      containers:
      - name: hello-kubernetes
        image: paulbouwer/hello-kubernetes:1.5
        ports:
        - containerPort: 8080
        env:
        - name: MESSAGE
          value: Hello from the first deployment!
```
- Apply `file hello-kubernetes-first.yaml`
```sh
kubectl apply -f nginx-service.yaml
```
- Check service vừa tạo:
```sh
$ kubectl get svc
NAME                      TYPE           CLUSTER-IP       EXTERNAL-IP     PORT(S)        AGE
hello-kubernetes-first    LoadBalancer   10.106.89.17     192.168.1.30    80:32026/TCP   28m
```
## Tài liệu tham khảo
- https://metallb.universe.tf/installation/
- https://github.com/thangtq710/Kubernetes/blob/master/docs/10.Loadbalancer_service_metallb.md
