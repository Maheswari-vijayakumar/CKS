# 🔐 Container-Level Security with `securityContext` in Kubernetes

A **Security Context** in Kubernetes defines privilege and access control settings for **pods or containers**.  
It tells the container runtime **how to run a container**, including:

- What **user** to run as
- Whether it can **escalate privileges**
- Which **Linux capabilities** to drop or allow
- Whether it should run as **privileged**

This example demonstrates how to secure a pod using container-level `securityContext`.

---

## 🛡️ Secured Pod Configuration Example

### Step 1: Create the YAML Manifest

```bash
vim secure-pod.yaml
````

Paste the following content:

```yaml
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

> 🔐 This pod explicitly:
>
> * Disables **privileged mode**
> * Prevents **privilege escalation**
> * Drops **all Linux capabilities**

---

### Step 2: Deploy the Pod

```bash
kubectl apply -f secure-pod.yaml
```

---

### Step 3: Inspect the Container

Access the pod:

```bash
kubectl exec -it non-privileged-pod-demo -- sh
```

Once inside, run the following:

---

#### ✅ Check the Current User

```bash
id
```

> You may still see `uid=0(root)`, but capabilities are removed, and privilege escalation is blocked.

---

#### ✅ Check Capabilities

```bash
capsh --print
```

> Should show `Capabilities: =` (i.e., **none active**)

---

#### ✅ File System Behavior

```bash
ls -l /data
```

> You should still have access to the mounted volume.

---

#### 🚫 Try Escalating Privileges

```bash
su -
```

> ❌ This should fail — privilege escalation is disabled.

---

#### 🚫 Try Installing Packages

```bash
yum install nmap-ncat
```

> ❌ Should be denied — this reduces the attack surface.

---

#### 🚫 Try Destructive Commands

```bash
rm -rf /*
```

> This command is blocked from doing harm when capabilities and permissions are correctly configured.

---

## 🔍 Summary

| Security Feature           | Value Set      | Effect                              |
| -------------------------- | -------------- | ----------------------------------- |
| `privileged: false`        | ✅ Set          | Prevents host-level access          |
| `allowPrivilegeEscalation` | ✅ Set to false | Blocks `sudo`/`su` access           |
| `capabilities.drop: [ALL]` | ✅ Set          | Removes all Linux kernel privileges |

---

### ✅ Best Practice

Always use `securityContext` at the **container level** to control behavior granularly. Combine this with `PodSecurityPolicy` or `PodSecurityAdmission` for cluster-wide enforcement.

```
```
