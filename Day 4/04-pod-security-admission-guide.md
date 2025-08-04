# Pod Security Admission (PSA) in Kubernetes

**Pod Security Admission (PSA)** is a built-in Kubernetes admission controller that enforces security standards for Pods **at creation time**, based on the [Pod Security Standards (PSS)](https://kubernetes.io/docs/concepts/security/pod-security-standards/).

---

## 🔐 Pod Security Levels

Kubernetes defines **three levels of Pod Security**:

| Level            | Description                                                                           |
| ---------------- | ------------------------------------------------------------------------------------- |
| **`restricted`** | Strongest. Blocks any risky settings. Enforces least-privilege. Ideal for production. |
| **`baseline`**   | Blocks high-risk settings like privileged containers but allows common ones.          |
| **`privileged`** | No restrictions. Allows everything. Not recommended for production.                   |

---

## 🔧 PSA Modes

You can enforce Pod Security in **three ways** by labeling namespaces:

- `enforce` – Blocks non-compliant Pods  
- `audit` – Logs violations but allows creation  
- `warn` – Shows a warning to users but still allows creation

---

## 🧪 Example: Applying PSA in a Namespace

```bash
kubectl create namespace restricted-psa-demo

kubectl label namespace restricted-psa-demo \
  pod-security.kubernetes.io/enforce=restricted \
  pod-security.kubernetes.io/audit=restricted \
  pod-security.kubernetes.io/warn=restricted
````

---

## ❌ Test a Non-Compliant Pod

Create a **privileged** Pod to see the enforcement in action.

### `privileged-pod.yaml`

```yaml
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

Apply the Pod:

```bash
kubectl apply -f privileged-pod.yaml -n restricted-psa-demo
```

### Expected Output:

```bash
Error from server: pods "bad-pod" is forbidden: violates PodSecurity "restricted:latest": privileged container
```

✅ PSA is working as expected.

---

## ✅ Deploy a Compliant (Good) Pod

### `good-pod.yaml`

```yaml
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
        drop: ["ALL"]
```

Apply the compliant Pod:

```bash
kubectl apply -f good-pod.yaml
```

✔️ This should succeed without any warnings or errors.

---

## 🧭 PSA Usage Guidelines

| Environment    | Recommended PSA Level       |
| -------------- | --------------------------- |
| **Production** | `restricted`                |
| **Test**       | `baseline`                  |
| **Dev**        | `privileged` (with caution) |

---

## 📌 Summary

* PSA helps enforce pod-level security using built-in admission controllers.
* Use namespace labels to apply policy (`enforce`, `audit`, `warn`).
* Choose a PSA level (`restricted`, `baseline`, `privileged`) based on your environment.
* Always validate pods against expected security controls.


Let me know if you'd like this integrated into a larger `.md` file covering all your Kubernetes security work — or if you'd prefer a table of contents at the top.
```
