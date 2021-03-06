# etcd
## etcd là gi
- K8s chạy trên nhiều host cùng 1 lúc nên cần 1 distributed database để dễ dàng lưu trữ data trên cluster.
- K8s sử dụng etcd như 1 key-value database store. Nó lưu cấu hình cluster bên trong etcd nên hãy đảm bảo việc back up cho nó.
- Sử dụng chức năng watch để giám sát các thay đổi. Nếu bị chia rẽ, k8s thực hiện thay đổi để điều chỉnh trạng thái hiên tại và trạng thái mong muốn.
- Lưu trữ output của `kubectl get`, Node crashing, process dying hoặc `kubectl create` cũng làm values trong etcd thay đổi.
- Tập hợp các process tạo nên Kubernetes sử dụng etcd để lưu trữ dữ liệu và thông báo cho nhau về các thay đổi.
- etcd sử dụng thuật toán đồng thuận [Raft](http://thesecretlivesofdata.com/raft/)

## Etcd trong Kubernetes

Trong ví dụ này `etcd` được deployed như Pods trên masters.

<img src=https://i.imgur.com/sihMtB7.png>

Để tăng security và khả năng phục hồi nó cũng có thể được triển khai như một external cluster.

<img src=https://i.imgur.com/qi2JPT5.png>

Đưới đây là ví dụ minh họa API Server tương tác với `ectd`.

<img src=https://i.imgur.com/gfn5sNr.png>

## The Etcd Pod
List tất cả các Pods chạy trong cluster
```sh
[root@server01 ~]# kubectl get pods --all-namespaces
NAMESPACE     NAME                                   READY   STATUS    RESTARTS   AGE
kube-system   coredns-5644d7b6d9-ngkcf               1/1     Running   0          2d22h
kube-system   coredns-5644d7b6d9-thpz9               1/1     Running   0          2d22h
kube-system   etcd-server01                          1/1     Running   0          2d22h
kube-system   kube-apiserver-server01                1/1     Running   0          2d22h
kube-system   kube-controller-manager-server01       1/1     Running   0          2d22h
kube-system   kube-flannel-ds-amd64-4jp5w            1/1     Running   0          2d22h
kube-system   kube-flannel-ds-amd64-flqxh            1/1     Running   0          2d22h
kube-system   kube-flannel-ds-amd64-xw4hg            1/1     Running   0          2d22h
kube-system   kube-proxy-9lpxc                       1/1     Running   0          2d22h
kube-system   kube-proxy-hqs7j                       1/1     Running   0          2d22h
kube-system   kube-proxy-l788n                       1/1     Running   0          2d22h
kube-system   kube-scheduler-server01                1/1     Running   0          2d22h
```
Ta quan tâm đến Pods `etcd-server01`. Chạy lệnh `shell` trong Pod `etcd` và kiểm tra cấu hình của container `etcd` đang chạy.

<img src=https://i.imgur.com/1f0e8gu.png>

Sử dụng giá trị của flag `--advertise-client-urls`, lấy tất cả các cặp key/value tồn tại sử dụng `etcdctl` và lưu vào `etcd-kv.json`.
```sh
$ ADVERTISE_URL="https://134.209.178.162:2379"
$ kubectl exec etcd-node-01 -n kube-system -- sh -c \
"ETCDCTL_API=3 etcdctl \
--endpoints $ADVERTISE_URL \
--cacert /etc/kubernetes/pki/etcd/ca.crt \
--key /etc/kubernetes/pki/etcd/server.key \
--cert /etc/kubernetes/pki/etcd/server.crt \
get \"\" --prefix=true -w json" > etcd-kv.json
```
File này list key và values tương ứng đều được mã hóa base64.

<img src=https://i.imgur.com/m2q6FLA.png>

Lấy tất cả keys dưới dạng plain text để xem.
```sh
$ for k in $(cat etcd-kv.json | jq '.kvs[].key' | cut -d '"' -f2); do echo $k | base64 --decode; echo; done
/registry/apiregistration.k8s.io/apiservices/v1.
/registry/apiregistration.k8s.io/apiservices/v1.apps
/registry/apiregistration.k8s.io/apiservices/v1.authentication.k8s.io
/registry/apiregistration.k8s.io/apiservices/v1.authorization.k8s.io
/registry/apiregistration.k8s.io/apiservices/v1.autoscaling
/registry/apiregistration.k8s.io/apiservices/v1.batch
/registry/apiregistration.k8s.io/apiservices/v1.coordination.k8s.io
/registry/apiregistration.k8s.io/apiservices/v1.networking.k8s.io
/registry/apiregistration.k8s.io/apiservices/v1.rbac.authorization.k8s.io
/registry/apiregistration.k8s.io/apiservices/v1.scheduling.k8s.io
/registry/apiregistration.k8s.io/apiservices/v1.storage.k8s.io
/registry/apiregistration.k8s.io/apiservices/v1beta1.admissionregistration.k8s.io
/registry/apiregistration.k8s.io/apiservices/v1beta1.apiextensions.k8s.io
/registry/apiregistration.k8s.io/apiservices/v1beta1.apps
/registry/apiregistration.k8s.io/apiservices/v1beta1.authentication.k8s.io
/registry/apiregistration.k8s.io/apiservices/v1beta1.authorization.k8s.io
/registry/apiregistration.k8s.io/apiservices/v1beta1.batch
/registry/apiregistration.k8s.io/apiservices/v1beta1.certificates.k8s.io
/registry/apiregistration.k8s.io/apiservices/v1beta1.coordination.k8s.io
/registry/apiregistration.k8s.io/apiservices/v1beta1.events.k8s.io
/registry/apiregistration.k8s.io/apiservices/v1beta1.extensions
/registry/apiregistration.k8s.io/apiservices/v1beta1.networking.k8s.io
/registry/apiregistration.k8s.io/apiservices/v1beta1.node.k8s.io
/registry/apiregistration.k8s.io/apiservices/v1beta1.policy
/registry/apiregistration.k8s.io/apiservices/v1beta1.rbac.authorization.k8s.io
/registry/apiregistration.k8s.io/apiservices/v1beta1.scheduling.k8s.io
/registry/apiregistration.k8s.io/apiservices/v1beta1.storage.k8s.io
/registry/apiregistration.k8s.io/apiservices/v1beta2.apps
/registry/apiregistration.k8s.io/apiservices/v2beta1.autoscaling
/registry/apiregistration.k8s.io/apiservices/v2beta2.autoscaling
/registry/certificatesigningrequests/csr-h9mcg
/registry/certificatesigningrequests/csr-qwnxf
/registry/certificatesigningrequests/csr-xklls
/registry/clusterrolebindings/cluster-admin
/registry/clusterrolebindings/kubeadm:kubelet-bootstrap
/registry/clusterrolebindings/kubeadm:node-autoapprove-bootstrap
/registry/clusterrolebindings/kubeadm:node-autoapprove-certificate-rotation
/registry/clusterrolebindings/kubeadm:node-proxier
/registry/clusterrolebindings/system:basic-user
/registry/clusterrolebindings/system:controller:attachdetach-controller
/registry/clusterrolebindings/system:controller:certificate-controller
/registry/clusterrolebindings/system:controller:clusterrole-aggregation-controller
/registry/clusterrolebindings/system:controller:cronjob-controller
/registry/clusterrolebindings/system:controller:daemon-set-controller
/registry/clusterrolebindings/system:controller:deployment-controller
/registry/clusterrolebindings/system:controller:disruption-controller
/registry/clusterrolebindings/system:controller:endpoint-controller
/registry/clusterrolebindings/system:controller:expand-controller
/registry/clusterrolebindings/system:controller:generic-garbage-collector
/registry/clusterrolebindings/system:controller:horizontal-pod-autoscaler
/registry/clusterrolebindings/system:controller:job-controller
/registry/clusterrolebindings/system:controller:namespace-controller
/registry/clusterrolebindings/system:controller:node-controller
/registry/clusterrolebindings/system:controller:persistent-volume-binder
/registry/clusterrolebindings/system:controller:pod-garbage-collector
/registry/clusterrolebindings/system:controller:pv-protection-controller
/registry/clusterrolebindings/system:controller:pvc-protection-controller
/registry/clusterrolebindings/system:controller:replicaset-controller
/registry/clusterrolebindings/system:controller:replication-controller
/registry/clusterrolebindings/system:controller:resourcequota-controller
/registry/clusterrolebindings/system:controller:route-controller
/registry/clusterrolebindings/system:controller:service-account-controller
/registry/clusterrolebindings/system:controller:service-controller
/registry/clusterrolebindings/system:controller:statefulset-controller
/registry/clusterrolebindings/system:controller:ttl-controller
/registry/clusterrolebindings/system:coredns
/registry/clusterrolebindings/system:discovery
/registry/clusterrolebindings/system:kube-controller-manager
/registry/clusterrolebindings/system:kube-dns
/registry/clusterrolebindings/system:kube-scheduler
/registry/clusterrolebindings/system:node
/registry/clusterrolebindings/system:node-proxier
/registry/clusterrolebindings/system:public-info-viewer
/registry/clusterrolebindings/system:volume-scheduler
/registry/clusterrolebindings/weave-net
/registry/clusterroles/admin
/registry/clusterroles/cluster-admin
/registry/clusterroles/edit
/registry/clusterroles/system:aggregate-to-admin
/registry/clusterroles/system:aggregate-to-edit
/registry/clusterroles/system:aggregate-to-view
/registry/clusterroles/system:auth-delegator
/registry/clusterroles/system:basic-user
/registry/clusterroles/system:certificates.k8s.io:certificatesigningrequests:nodeclient
/registry/clusterroles/system:certificates.k8s.io:certificatesigningrequests:selfnodeclient
/registry/clusterroles/system:controller:attachdetach-controller
/registry/clusterroles/system:controller:certificate-controller
/registry/clusterroles/system:controller:clusterrole-aggregation-controller
/registry/clusterroles/system:controller:cronjob-controller
/registry/clusterroles/system:controller:daemon-set-controller
/registry/clusterroles/system:controller:deployment-controller
/registry/clusterroles/system:controller:disruption-controller
/registry/clusterroles/system:controller:endpoint-controller
/registry/clusterroles/system:controller:expand-controller
/registry/clusterroles/system:controller:generic-garbage-collector
/registry/clusterroles/system:controller:horizontal-pod-autoscaler
/registry/clusterroles/system:controller:job-controller
/registry/clusterroles/system:controller:namespace-controller
/registry/clusterroles/system:controller:node-controller
/registry/clusterroles/system:controller:persistent-volume-binder
/registry/clusterroles/system:controller:pod-garbage-collector
/registry/clusterroles/system:controller:pv-protection-controller
/registry/clusterroles/system:controller:pvc-protection-controller
/registry/clusterroles/system:controller:replicaset-controller
/registry/clusterroles/system:controller:replication-controller
/registry/clusterroles/system:controller:resourcequota-controller
/registry/clusterroles/system:controller:route-controller
/registry/clusterroles/system:controller:service-account-controller
/registry/clusterroles/system:controller:service-controller
/registry/clusterroles/system:controller:statefulset-controller
/registry/clusterroles/system:controller:ttl-controller
/registry/clusterroles/system:coredns
/registry/clusterroles/system:csi-external-attacher
/registry/clusterroles/system:csi-external-provisioner
/registry/clusterroles/system:discovery
/registry/clusterroles/system:heapster
/registry/clusterroles/system:kube-aggregator
/registry/clusterroles/system:kube-controller-manager
/registry/clusterroles/system:kube-dns
/registry/clusterroles/system:kube-scheduler
/registry/clusterroles/system:kubelet-api-admin
/registry/clusterroles/system:node
/registry/clusterroles/system:node-bootstrapper
/registry/clusterroles/system:node-problem-detector
/registry/clusterroles/system:node-proxier
/registry/clusterroles/system:persistent-volume-provisioner
/registry/clusterroles/system:public-info-viewer
/registry/clusterroles/system:volume-scheduler
/registry/clusterroles/view
/registry/clusterroles/weave-net
/registry/configmaps/kube-public/cluster-info
/registry/configmaps/kube-system/coredns
/registry/configmaps/kube-system/extension-apiserver-authentication
/registry/configmaps/kube-system/kube-proxy
/registry/configmaps/kube-system/kubeadm-config
/registry/configmaps/kube-system/kubelet-config-1.15
/registry/configmaps/kube-system/weave-net
/registry/controllerrevisions/kube-system/kube-proxy-84c6b844cd
/registry/controllerrevisions/kube-system/weave-net-7db89b6d4
/registry/daemonsets/kube-system/kube-proxy
/registry/daemonsets/kube-system/weave-net
/registry/deployments/kube-system/coredns
/registry/events/default/node-01.15b9e0cd75ea6932
/registry/events/default/node-02.15b9e0ae0342c323
/registry/events/default/node-02.15b9e0ae0f9c2ae3
/registry/events/default/node-02.15b9e0ae0f9c5fa9
/registry/events/default/node-02.15b9e0ae0f9c7206
/registry/events/default/node-02.15b9e0ae1575182e
/registry/events/default/node-02.15b9e0aea1c4eeaf
/registry/events/default/node-02.15b9e0af99ba73a2
/registry/events/default/node-02.15b9e0ca43c5e760
/registry/events/default/node-03.15b9e0ae9bdae96c
/registry/events/default/node-03.15b9e0aea880813c
/registry/events/default/node-03.15b9e0aea880ae05
/registry/events/default/node-03.15b9e0aea880c0de
/registry/events/default/node-03.15b9e0aeb13cfeef
/registry/events/default/node-03.15b9e0afcbcf299b
/registry/events/default/node-03.15b9e0b02f28fa3c
/registry/events/default/node-03.15b9e0cadf7dce89
/registry/events/kube-system/coredns-5c98db65d4-5kjjv.15b9ddb67e6ab700
/registry/events/kube-system/coredns-5c98db65d4-5kjjv.15b9e0af3bdb47fe
/registry/events/kube-system/coredns-5c98db65d4-5kjjv.15b9e0cbbbb7579d
/registry/events/kube-system/coredns-5c98db65d4-5kjjv.15b9e0cc279fbd33
/registry/events/kube-system/coredns-5c98db65d4-5kjjv.15b9e0cc34fb8de2
/registry/events/kube-system/coredns-5c98db65d4-5kjjv.15b9e0cc4994ad54
/registry/events/kube-system/coredns-5c98db65d4-88hkq.15b9ddb6850e5ff1
/registry/events/kube-system/coredns-5c98db65d4-88hkq.15b9e0aea988964f
/registry/events/kube-system/coredns-5c98db65d4-88hkq.15b9e0cbbb3af928
/registry/events/kube-system/coredns-5c98db65d4-88hkq.15b9e0cc2ffb9d11
/registry/events/kube-system/coredns-5c98db65d4-88hkq.15b9e0cc3a4def6c
/registry/events/kube-system/coredns-5c98db65d4-88hkq.15b9e0cc4bd20265
/registry/events/kube-system/coredns-5c98db65d4-88hkq.15b9e0cc6e488534
/registry/events/kube-system/kube-proxy-7642v.15b9e0ae1444b38c
/registry/events/kube-system/kube-proxy-7642v.15b9e0ae7ff6f434
/registry/events/kube-system/kube-proxy-7642v.15b9e0af631fa3d0
/registry/events/kube-system/kube-proxy-7642v.15b9e0af7632698a
/registry/events/kube-system/kube-proxy-7642v.15b9e0af85356aad
/registry/events/kube-system/kube-proxy-jsp4r.15b9e0aeadc2ce3a
/registry/events/kube-system/kube-proxy-jsp4r.15b9e0af27535c1b
/registry/events/kube-system/kube-proxy-jsp4r.15b9e0affc7fc79e
/registry/events/kube-system/kube-proxy-jsp4r.15b9e0b00a290340
/registry/events/kube-system/kube-proxy-jsp4r.15b9e0b01b0a4eef
/registry/events/kube-system/kube-proxy.15b9e0ae1333a730
/registry/events/kube-system/kube-proxy.15b9e0aeaad76df0
/registry/events/kube-system/weave-net-2hvbx.15b9e0c6e9b6c1de
/registry/events/kube-system/weave-net-2hvbx.15b9e0c71a365ad4
/registry/events/kube-system/weave-net-2hvbx.15b9e0c88a5af203
/registry/events/kube-system/weave-net-2hvbx.15b9e0c8a5998774
/registry/events/kube-system/weave-net-2hvbx.15b9e0c8b54252cb
/registry/events/kube-system/weave-net-2hvbx.15b9e0c8b5543df6
/registry/events/kube-system/weave-net-2hvbx.15b9e0c98384d3e1
/registry/events/kube-system/weave-net-2hvbx.15b9e0c9916478ce
/registry/events/kube-system/weave-net-2hvbx.15b9e0c9a090c521
/registry/events/kube-system/weave-net-5mrjl.15b9e0c6e9523ad2
/registry/events/kube-system/weave-net-5mrjl.15b9e0c7194191cb
/registry/events/kube-system/weave-net-5mrjl.15b9e0c89c46497c
/registry/events/kube-system/weave-net-5mrjl.15b9e0c8b335c817
/registry/events/kube-system/weave-net-5mrjl.15b9e0c8c714f12d
/registry/events/kube-system/weave-net-5mrjl.15b9e0c8c770ebdd
/registry/events/kube-system/weave-net-5mrjl.15b9e0c995196184
/registry/events/kube-system/weave-net-5mrjl.15b9e0c9a24d099d
/registry/events/kube-system/weave-net-5mrjl.15b9e0c9b2e0cdef
/registry/events/kube-system/weave-net-c76fx.15b9e0c6ec0133eb
/registry/events/kube-system/weave-net-c76fx.15b9e0c7255593bb
/registry/events/kube-system/weave-net-c76fx.15b9e0c8d4f52821
/registry/events/kube-system/weave-net-c76fx.15b9e0c90ebfeb95
/registry/events/kube-system/weave-net-c76fx.15b9e0c922410c3a
/registry/events/kube-system/weave-net-c76fx.15b9e0c922580ded
/registry/events/kube-system/weave-net-c76fx.15b9e0c9f7834364
/registry/events/kube-system/weave-net-c76fx.15b9e0ca15411664
/registry/events/kube-system/weave-net-c76fx.15b9e0ca2d254f2c
/registry/events/kube-system/weave-net.15b9e0c6e7edf622
/registry/events/kube-system/weave-net.15b9e0c6e9c8d2c1
/registry/events/kube-system/weave-net.15b9e0c6ea880cd2
/registry/leases/kube-node-lease/node-01
/registry/leases/kube-node-lease/node-02
/registry/leases/kube-node-lease/node-03
/registry/masterleases/134.209.178.162
/registry/minions/node-01
/registry/minions/node-02
/registry/minions/node-03
/registry/namespaces/default
/registry/namespaces/kube-node-lease
/registry/namespaces/kube-public
/registry/namespaces/kube-system
/registry/pods/kube-system/coredns-5c98db65d4-5kjjv
/registry/pods/kube-system/coredns-5c98db65d4-88hkq
/registry/pods/kube-system/etcd-node-01
/registry/pods/kube-system/kube-apiserver-node-01
/registry/pods/kube-system/kube-controller-manager-node-01
/registry/pods/kube-system/kube-proxy-7642v
/registry/pods/kube-system/kube-proxy-jsp4r
/registry/pods/kube-system/kube-proxy-xj8qm
/registry/pods/kube-system/kube-scheduler-node-01
/registry/pods/kube-system/weave-net-2hvbx
/registry/pods/kube-system/weave-net-5mrjl
/registry/pods/kube-system/weave-net-c76fx
/registry/priorityclasses/system-cluster-critical
/registry/priorityclasses/system-node-critical
/registry/ranges/serviceips
/registry/ranges/servicenodeports
/registry/replicasets/kube-system/coredns-5c98db65d4
/registry/rolebindings/kube-public/kubeadm:bootstrap-signer-clusterinfo
/registry/rolebindings/kube-public/system:controller:bootstrap-signer
/registry/rolebindings/kube-system/kube-proxy
/registry/rolebindings/kube-system/kubeadm:kubelet-config-1.15
/registry/rolebindings/kube-system/kubeadm:nodes-kubeadm-config
/registry/rolebindings/kube-system/system::extension-apiserver-authentication-reader
/registry/rolebindings/kube-system/system::leader-locking-kube-controller-manager
/registry/rolebindings/kube-system/system::leader-locking-kube-scheduler
/registry/rolebindings/kube-system/system:controller:bootstrap-signer
/registry/rolebindings/kube-system/system:controller:cloud-provider
/registry/rolebindings/kube-system/system:controller:token-cleaner
/registry/rolebindings/kube-system/weave-net
/registry/roles/kube-public/kubeadm:bootstrap-signer-clusterinfo
/registry/roles/kube-public/system:controller:bootstrap-signer
/registry/roles/kube-system/extension-apiserver-authentication-reader
/registry/roles/kube-system/kube-proxy
/registry/roles/kube-system/kubeadm:kubelet-config-1.15
/registry/roles/kube-system/kubeadm:nodes-kubeadm-config
/registry/roles/kube-system/system::leader-locking-kube-controller-manager
/registry/roles/kube-system/system::leader-locking-kube-scheduler
/registry/roles/kube-system/system:controller:bootstrap-signer
/registry/roles/kube-system/system:controller:cloud-provider
/registry/roles/kube-system/system:controller:token-cleaner
/registry/roles/kube-system/weave-net
/registry/secrets/default/default-token-nz988
/registry/secrets/kube-node-lease/default-token-4w7tf
/registry/secrets/kube-public/default-token-pzhnr
/registry/secrets/kube-system/attachdetach-controller-token-69mzv
/registry/secrets/kube-system/bootstrap-signer-token-584pq
/registry/secrets/kube-system/bootstrap-token-w1d2kx
/registry/secrets/kube-system/certificate-controller-token-rff4s
/registry/secrets/kube-system/clusterrole-aggregation-controller-token-6hks4
/registry/secrets/kube-system/coredns-token-b2874
/registry/secrets/kube-system/cronjob-controller-token-55pgx
/registry/secrets/kube-system/daemon-set-controller-token-nhcdf
/registry/secrets/kube-system/default-token-f5kl4
/registry/secrets/kube-system/deployment-controller-token-lm58l
/registry/secrets/kube-system/disruption-controller-token-4tw6s
/registry/secrets/kube-system/endpoint-controller-token-qdh8q
/registry/secrets/kube-system/expand-controller-token-6stw5
/registry/secrets/kube-system/generic-garbage-collector-token-hqfqx
/registry/secrets/kube-system/horizontal-pod-autoscaler-token-h6czj
/registry/secrets/kube-system/job-controller-token-nmw8f
/registry/secrets/kube-system/kube-proxy-token-zcrj8
/registry/secrets/kube-system/namespace-controller-token-trhl9
/registry/secrets/kube-system/node-controller-token-mmf4d
/registry/secrets/kube-system/persistent-volume-binder-token-wnh9s
/registry/secrets/kube-system/pod-garbage-collector-token-h7vvp
/registry/secrets/kube-system/pv-protection-controller-token-lcqb6
/registry/secrets/kube-system/pvc-protection-controller-token-k2kf8
/registry/secrets/kube-system/replicaset-controller-token-zhc7k
/registry/secrets/kube-system/replication-controller-token-l8hr6
/registry/secrets/kube-system/resourcequota-controller-token-bglb2
/registry/secrets/kube-system/service-account-controller-token-5dhxz
/registry/secrets/kube-system/service-controller-token-l98rk
/registry/secrets/kube-system/statefulset-controller-token-dj85r
/registry/secrets/kube-system/token-cleaner-token-qz8hs
/registry/secrets/kube-system/ttl-controller-token-6vbv6
/registry/secrets/kube-system/weave-net-token-87h6x
/registry/serviceaccounts/default/default
/registry/serviceaccounts/kube-node-lease/default
/registry/serviceaccounts/kube-public/default
/registry/serviceaccounts/kube-system/attachdetach-controller
/registry/serviceaccounts/kube-system/bootstrap-signer
/registry/serviceaccounts/kube-system/certificate-controller
/registry/serviceaccounts/kube-system/clusterrole-aggregation-controller
/registry/serviceaccounts/kube-system/coredns
/registry/serviceaccounts/kube-system/cronjob-controller
/registry/serviceaccounts/kube-system/daemon-set-controller
/registry/serviceaccounts/kube-system/default
/registry/serviceaccounts/kube-system/deployment-controller
/registry/serviceaccounts/kube-system/disruption-controller
/registry/serviceaccounts/kube-system/endpoint-controller
/registry/serviceaccounts/kube-system/expand-controller
/registry/serviceaccounts/kube-system/generic-garbage-collector
/registry/serviceaccounts/kube-system/horizontal-pod-autoscaler
/registry/serviceaccounts/kube-system/job-controller
/registry/serviceaccounts/kube-system/kube-proxy
/registry/serviceaccounts/kube-system/namespace-controller
/registry/serviceaccounts/kube-system/node-controller
/registry/serviceaccounts/kube-system/persistent-volume-binder
/registry/serviceaccounts/kube-system/pod-garbage-collector
/registry/serviceaccounts/kube-system/pv-protection-controller
/registry/serviceaccounts/kube-system/pvc-protection-controller
/registry/serviceaccounts/kube-system/replicaset-controller
/registry/serviceaccounts/kube-system/replication-controller
/registry/serviceaccounts/kube-system/resourcequota-controller
/registry/serviceaccounts/kube-system/service-account-controller
/registry/serviceaccounts/kube-system/service-controller
/registry/serviceaccounts/kube-system/statefulset-controller
/registry/serviceaccounts/kube-system/token-cleaner
/registry/serviceaccounts/kube-system/ttl-controller
/registry/serviceaccounts/kube-system/weave-net
/registry/services/endpoints/default/kubernetes
/registry/services/endpoints/kube-system/kube-controller-manager
/registry/services/endpoints/kube-system/kube-dns
/registry/services/endpoints/kube-system/kube-scheduler
/registry/services/specs/default/kubernetes
/registry/services/specs/kube-system/kube-dns
compact_rev_key
```
Kết quả trên show 342 keys xác định cấu hình và trạng thái của tất cả resources trong cluster:

- Nodes
- Namespaces
- ServiceAccounts
- Roles and RoleBindings, ClusterRoles / ClusterRoleBindings
- ConfigMaps
- Secrets
- Workloads: Deployments, DaemonSets, Pods, …
- Cluster’s certificates
- The resources within each apiVersion
- The events that bring the cluster in the current state

Chọn 1 trong các keys có thể nhận được các giá trị liên quan bằng command sau:
```sh
$ kubectl exec etcd-node-01 -n kube-system —- sh -c \
"ETCDCTL_API=3 etcdctl \
--endpoints $ADVERTISE_URL \
--cacert /etc/kubernetes/pki/etcd/ca.crt \
--key /etc/kubernetes/pki/etcd/server.key \
--cert /etc/kubernetes/pki/etcd/server.crt \
get \"KEY\" -w json"
```
Ví dụ lấy các giá trị liên quan đến key `/registry/deployments/kube-system/coredns`:

<img src=https://i.imgur.com/yxI3MXx.png>

Từ kết quả có thể suy ra key được sử dụng để lưu trữ thông số kỹ thuật và trạng thái của việc deployment managing Pods `coredns`.

## Creation of a Pod

Create 1 Pod và check cách thay đổi trạng thái của cluster và keys mới được thêm.
```sh
$ cat <<EoF | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: www
spec:
  containers:
  - name: nginx
    image: nginx:1.16-alpine
EoF```
```
Sử dụng command lấy tất cả keys và lưu trong `etcd-kv-after-nginx-pod.json`. So sánh 2 danh sách keys, 1 danh sách lấy ngay sau khi tạo cluster `etcd-kv.json` và 1 danh sách ngay sau khi deployed Pod `www` là `etcd-kv-after-nginx-pod.json`, show nội dung:
```sh
> /registry/events/default/www.15b9e3051648764f
> /registry/events/default/www.15b9e3056b8ce3f0
> /registry/events/default/www.15b9e306918312ea
> /registry/events/default/www.15b9e306a32beb6d
> /registry/events/default/www.15b9e306b5892b60
> /registry/pods/default/www
```
Năm events được tạo và 1 Pod, theo thứ tự chúng được liên kết với các actions sau:
- Successfully assigned `default/www to node-02`
- Pulling image `nginx:1.16-alpine`
- Successfully pulled image `nginx:1.16-alpine`
- Created container `nginx`
- Started `Started container nginx`

Các events được listed ở cuối command: 
```sh
kubectl describe pod www
```

Key cuối cùng `/registry/pods/default/www` cuung cấp tất cả thông tin liên quan đến Pod mới tạo:

- The last applied configuration
- The associated token
- Its status
- …

<img src=https://i.imgur.com/RLSNKnp.png>

## Tài liệu tham khảo
- https://matthewpalmer.net/kubernetes-app-developer/articles/how-does-kubernetes-use-etcd.html
- https://medium.com/better-programming/a-closer-look-at-etcd-the-brain-of-a-kubernetes-cluster-788c8ea759a5
