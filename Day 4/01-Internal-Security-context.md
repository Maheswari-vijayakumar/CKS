=> Pod level security

=> Normal pod
```
vim regular_pod.yaml

```

```
apiVersion: v1
kind: Pod
metadata:
  name: regular-pod-demo
spec:
  volumes:
  - name: demo-volume
    emptyDir: {}
  containers:
  - name: regular-demo
    image: docker.io/redhat/ubi8
    command: [ "tail", "-f", "/dev/null" ]
    volumeMounts:
    - name: demo-volume
      mountPath: /data/demo
```

```
kubectl apply -f vim regular_pod.yaml
kubectl get pods
kubectl describe pod regular-pod-demo
kubectl exec -it regular-pod-demo -- sh

# you can see it is running as a user using below command id
id

# To check the cabability of pod
capsh --print




```






