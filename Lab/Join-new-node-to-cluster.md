- Command tạo cụm cluster
```sh
kubeadm init --config=kubeadm-config.yaml --upload-certs
```
- Tạo token
```sh
kubeadm token create --config=kubeadm-config.yaml --print-join-command
```
- Tạo cert mới
```sh
kubeadm init phase upload-certs --config=kubeadm-config.yaml --upload-certs
```
- Join vào cụm cert mới
```sh
kubeadm join 172.16.68.100:6445 --token 0rhev0.mrunhgzi07zb9cao \
  --discovery-token-ca-cert-hash sha256:923716564e6113f5c83c559ed345e766c81119dda427769e5df151a018dd42ed \
  --control-plane --certificate-key 845be50a981f3bb69198d52203c77db87f6d01062bfe635fc4d7bd1d7ae3262e
```
