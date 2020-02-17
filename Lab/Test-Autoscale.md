## Increase load
Now, we will see how the autoscaler reacts to increased load. We will start a container, and send an infinite loop of queries to the php-apache service (please run it in a different terminal):
```sh

kubectl run --generator=run-pod/v1 -it --rm load-generator --image=busybox /bin/sh
while true; do wget -q -O- http://php-apache.default.svc.cluster.local; done
```

## Tài liệu tham khảo
- https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/
