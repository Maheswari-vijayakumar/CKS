=> Container level security

```
vim secure-pod.yaml
```

```
apiVersion: v1
kind: Pod
metadata:
  name: non-privileged-pod-demo
spec:
  volumes:
  - name: demo-volume
    emptyDir: {}
  containers:
  - name: non-privileged-demo
    image: docker.io/redhat/ubi8
    command: [ "tail", "-f", "/dev/null" ]
    volumeMounts:
    - name: demo-volume
      mountPath: /data/demo
    securityContext:
      privileged: false
      allowPrivilegeEscalation: false
      capabilities:
        drop:
          - ALL
```

```
kubectl apply -f secure-pod.yaml

Go inside the pod and execute:
id
capsh --print
ls -l /data
su -
yum install nmap-ncat
rm -rf /*
```


  
