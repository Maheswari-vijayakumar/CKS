### 🔹 What is AppArmor?

**AppArmor** (Application Armor) is a **Linux security module (LSM)** that provides **Mandatory Access Control (MAC)**.
It works by restricting programs to a set of predefined capabilities, limiting what they can do and which files they can access — even if they are compromised.

**Step #1** - Login on a kubernetes nodes and check status of apparmor

```
cat /sys/module/apparmor/parameters/enabled
systemctl status apparmor
aa-status

```

**Step #2** Create a pod and try creating a file from inside the pod. 
        Pod is writing on host filesystem

```
vim apparmor-test-pod.yaml
```

```
apiVersion: v1
kind: Pod
metadata:
  name: testpod
  labels:
   app: testpod
spec:
  nodeName: manager
  containers:
    - name: boxone
      image: lovelearnlinux/webserver:v1
```

```
kubectl apply -f apparmor-test-pod.yaml
 kubectl get pods -o wide
kubectl exec -it testpod -- /bin/bash
echo "File created by the pod" > /tmp/test
cat /tmp/test

File is creating, I want to deny this action,let us deny this next

```

**Step #4** Create a apparmor profile which will deny any writes to host filesystem

```
sudo  -i
cd /etc/apparmor.d
vim deny-file-write
```

```
#include <tunables/global>
 
# profile <profilename> <flags> 
profile deny-file-write flags=(attach_disconnected) {
  #include <abstractions/base>

  file,

  # Deny all file writes.
  deny /** w,
}
```

**Step #5** Check current apparmor loaded profiles & load our profile

```
cd
apparmor_parser /etc/apparmor.d/deny-file-write
```
**To check the Deny file status**
```
aa-status
or

apparmor_parser | grep -i deny-file-writes
```

```
apparmor_parser -C /etc/apparmor.d/deny-file-write  #load in complain mode
apparmor_parser -R /etc/apparmor.d/deny-file-write  #disable profile

````

**Step #6** Re-create the pod with apparmor profile loaded.

```
kubectl delete pod testpod
rm apparmor-test-pod.yaml
vim apparmor-test-pod.yaml
```

```
apiVersion: v1
kind: Pod
metadata:
  name: testpod
  labels:
   app: testpod
spec:
  securityContext:
    appArmorProfile:
      type: Localhost
      localhostProfile: deny-file-write
  nodeName: manager
  containers:
    - name: boxone
      image: lovelearnlinux/webserver:v1
```

```
kubectl apply -f apparmor-test-pod.yaml
```


















