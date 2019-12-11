
Ví dụ về 1 `file.yaml`
```sh
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-deployment
  labels:
    track: canary
spec:
  selector:
    matchLabels:
      any-name: my-app
  template:
    metadata:
      labels:
        any-name: my-app
    spec:
      containers:
      - name: cont1
        image: learnk8s/app:1.0.0
        ports:
        - containerPort: 8080
---
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  ports:
  - port: 80
    targetPort: 8080
  selector:
    any-name: my-app
```
3 điều cần nhớ:
- Service `selector` nên khớp với ít nhất 1 label của Pod
- Service `targetPort` nên khớp với `containerPort` của container trong Pod
- Service `port` có thể là bất kỳ số nào. Nhiều Services có thể sử dụng cùng 1 port vì chúng có IP khác nhau.

3 loại port 
- `port`: Expose service trên port được chỉ địng trong cluster. Service sẽ hiển thị trên `port` này và sẽ gửi requests đến `port` này đến service được lựa chọn.
- `targetPort`: Là port trên Pod để nhận các request tới. Ứng dụng của bạn cần listening các requests trên port này để service hoạt động.
- `nodePort`: Port này dùng để expose service ra bên ngoài bằng IP của Node.

Label
- `matchLabels`: Luôn phải khớp với label của Pod được sử dụng bởi Deployment để track Pod

2 điều cần khớp trong Ingress và Service
- `servicePort` của Ingress nên khớp với `port` của Service
- `serviceName` của Ingress nên khớp với `name` của Service

<img src=https://i.imgur.com/vTZ2OoM.png>

## 3 bước để troubleshoot Kubernetes deployments
- Đảm bảo các Pods đang running
- Tập trung vào Service để định tuyến traffic đến Pods
- Kiểm tra Ingress có cấu hình đúng không

### 3.1. Troubleshooting Pods
```sh
kubectl logs <pod name> is helpful to retrieve the logs of the containers of the Pod
kubectl describe pod <pod name> is useful to retrieve a list of events associated with the Pod
kubectl get pod <pod name> is useful to extract the YAML definition of the Pod as stored in Kubernetes
kubectl exec -ti <pod name> bash is useful to run an interactive command within one of the containers of the Pod
```
**Common Pods errors**

Pods can have startup and runtime errors.

Startup errors include:
- ImagePullBackoff
- ImageInspectError
- ErrImagePull
- ErrImageNeverPull
- RegistryUnavailable
- InvalidImageName

Runtime errors include:
- CrashLoopBackOff
- RunContainerError
- KillContainerError
- VerifyNonRootError
- RunInitContainerError
- CreatePodSandboxError
- ConfigPodSandboxError
- KillPodSandboxError
- SetupNetworkError
- TeardownNetworkError

## Tài liệu tham khảo
- https://learnk8s.io/troubleshooting-deployments?fbclid=IwAR1CKwgpOoV0VoqIVa7nkSP65tiYMsDeqZlrutI9tMVEarfUz1m-sMV2bTs
- https://matthewpalmer.net/kubernetes-app-developer/articles/kubernetes-ports-targetport-nodeport-service.html
