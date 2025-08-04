Here’s a well-formatted and annotated Markdown version of your **Pod-Level Security: Normal Pod** example, with context added for clarity — commands and YAML are kept **exactly as you wrote**:

---

````md
# 🔐 Pod-Level Security in Kubernetes

Before we apply strict security controls, it's important to observe how a **normal, unsecured pod** behaves.

This section demonstrates what happens when a pod is deployed **without** security constraints like `runAsUser`, `readOnlyRootFilesystem`, or dropping capabilities.

---

## 🚫 Unsecured / Regular Pod Example

### 🧾 Step 1: Create the Pod Manifest

```bash
vim regular_pod.yaml
````

Paste the following:

```yaml
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

---

### 🚀 Step 2: Deploy the Pod

```bash
kubectl apply -f regular_pod.yaml
kubectl get pods
kubectl describe pod regular-pod-demo
```

---

### 🔍 Step 3: Investigate the Pod’s Runtime Behavior

```bash
kubectl exec -it regular-pod-demo -- sh
```

From inside the pod:

#### Check the User Context:

```bash
id
```

> 🔎 You’ll likely see it's running as **`uid=0(root)`**, which means the container has root privileges.

---

#### Check Container Capabilities:

```bash
capsh --print
```

> 🚨 This will show a broad set of Linux capabilities (like `NET_RAW`, `SYS_ADMIN`, etc.), which may pose a **security risk**.

---

#### Try File System Access:

```bash
ls -l /data
touch testfile
```

> ✅ The pod can **write freely** to the mounted volume, even as root.

---

#### Test Package Installation:

```bash
yum install nmap-ncat
```

> ✅ You’ll notice the container can install software — **not ideal** in a production environment.

---

#### Attempt Root Shell Access:

```bash
su -
```

> 🔓 This should also work, confirming that the container has full **root and sudo** access.

---

## ⚠️ Summary: What Went Wrong?

| Behavior              | Status    | Risk Level |
| --------------------- | --------- | ---------- |
| Runs as root user     | ✅ Allowed | 🔴 High    |
| Installs packages     | ✅ Allowed | 🔴 High    |
| Writes to all volumes | ✅ Allowed | 🟠 Medium  |
| Has all Linux caps    | ✅ Allowed | 🔴 High    |

This "normal" pod is highly permissive and **violates security best practices**.
Next, we’ll **harden this pod** using Kubernetes security features like:

* `runAsUser`, `runAsNonRoot`
* `readOnlyRootFilesystem`
* `drop` Linux capabilities
* Seccomp and AppArmor (optional)

🛡️ Stay tuned for the next section: **Secured Pod with Restricted Access**

