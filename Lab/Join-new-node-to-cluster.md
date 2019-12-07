Command tạo cụm cluster từ 1 file config có sẵn.
```sh
kubeadm init --config=kubeadm-config.yaml --upload-certs
```
1. Join 1 Node mới vào cluster với vai trò là Worker
- Tạo token trên máy Master có sẵn
```sh
kubeadm token create --config=kubeadm-config.yaml --print-join-command
```
Lệnh này sẽ in ra màn hình command để join 1 node Worker vào cụm
```sh
kubeadm join 172.16.68.100:6445 --token 0rhev0.mrunhgzi07zb9cao \
  --discovery-token-ca-cert-hash sha256:923716564e6113f5c83c559ed345e766c81119dda427769e5df151a018dd42ed \
  ```
2. join 1 Node mới vào cluster với vai trò là Master
- Tạo cert mới trên Node Master có với file config ban đầu.
```sh
kubeadm init phase upload-certs --config=kubeadm-config.yaml --upload-certs
```
Lệnh này in ra 1 `certificate-key`. Kết hợp `certificate-key` này với command tạo token ở trên tương tự như sau:
```sh
kubeadm join 172.16.68.100:6445 --token 0rhev0.mrunhgzi07zb9cao \
  --discovery-token-ca-cert-hash sha256:923716564e6113f5c83c559ed345e766c81119dda427769e5df151a018dd42ed \
  --control-plane --certificate-key 845be50a981f3bb69198d52203c77db87f6d01062bfe635fc4d7bd1d7ae3262e
```
