
Pod Security Admission (PSA) is a built-in Kubernetes admission controller that enforces security standards for Pods at creation time, based on the Pod Security Standards (PSS).

There are three predefined levels of security 

| Level            | Description                                                                           |
| ---------------- | ------------------------------------------------------------------------------------- |
| **`restricted`** | Strongest. Blocks any risky settings. Enforces least-privilege. Ideal for production. |
| **`baseline`**   | Blocks high-risk settings like privileged containers but allows common ones.          |
| **`privileged`** | No restrictions. Allows everything. Not recommended for production.                   |


You can also configure:

enforce: Blocks non-compliant pods

audit: Logs violations but allows pod creation

warn: Sends warnings to users but allows creation


```
kubectl create namespace restricted-psa-demo

kubectl label namespace restricted-psa-demo \
  pod-security.kubernetes.io/enforce=restricted \
  pod-security.kubernetes.io/audit=restricted \
  pod-security.kubernetes.io/warn=restricted
```

```
vim privileged-pod.yaml
```

```
apiVersion: v1
kind: Pod
metadata:
  name: bad-pod
  namespace: restricted-psa-demo
spec:
  containers:
  - name: nginx
    image: nginx

```

```
kubectl apply -f privileged-pod.yaml -n restricted-psa-demo
```

you should get error => working as expected
```
Error from server: pods "bad-pod" is forbidden: violates PodSecurity "restricted:latest": privileged container
```

=> in prod => we can use restricted
=> in test => we can use baseline
=> in dev => we can use privileged

Next, Create a Compliant Pod

```
vim good-pod.yaml
```

```
apiVersion: v1
kind: Pod
metadata:
  name: good-pod
  namespace: restricted-psa-demo
spec:
  containers:
  - name: nginx
    image: nginx
    securityContext:
      allowPrivilegeEscalation: false
      runAsNonRoot: true
      capabilities:
        drop: ["ALL"] # this is making the pod good because restricted all previllages
```

```
kubectl apply -f good-pod.yaml
```
✔️ This should succeed.




