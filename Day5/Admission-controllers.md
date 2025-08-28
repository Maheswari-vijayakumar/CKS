## 🔹 What are **Admission Controllers** in Kubernetes?

**Admission Controllers** are **plugins in the Kubernetes API server** that intercept requests to the cluster **after authentication/authorization** but **before objects are persisted to etcd**.

They allow Kubernetes to **validate, mutate, or reject** requests when you create, update, or delete resources.

### 🔹 Types of Admission Controllers

1. **Validating Admission Controllers**

   * Check if the request complies with cluster rules.
   * Example: Prevent running privileged pods.

2. **Mutating Admission Controllers**

   * Can **modify the request object** before it’s stored.
   * Example: Automatically inject sidecar containers (like Istio).


```
vim test-admission.yaml
```

```
apiVersion: v1
kind: Pod
metadata:
  name: admission-test
  namespace: notexist-ns   # 🚨 This namespace doesn't exist
spec:
  containers:
  - name: nginx
    image: nginx
```

```
kubectl apply -f test-admission.yaml
```

you should get error like below

```
Error from server (Forbidden): error when creating "test-admission.yaml":
namespaces "notexist-ns" not found
```

let’s do one with the **LimitRanger Admission Controller** and the **Burstable QoS class**.

Kubernetes Admission Controllers help enforce resource policies. If you don’t set **requests/limits** properly, the **LimitRanger** controller can step in.

---

## 🔹 Example: Create a Pod with `Burstable` QoS

Kubernetes divides pods into 3 QoS classes:

* **Guaranteed** → requests == limits for all containers.
* **Burstable** → requests < limits (but requests are set).
* **BestEffort** → no requests or limits set.

---

### Step 1: Pod manifest (`burstable-pod.yaml`)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: burstable-test
spec:
  containers:
  - name: nginx
    image: nginx
    resources:
      requests:        # Minimum resources requested
        cpu: "100m"
        memory: "64Mi"
      limits:          # Maximum allowed
        cpu: "200m"
        memory: "128Mi"
```

Here:

* `requests < limits` → Pod is **Burstable**.
* Admission Controller (`LimitRanger`) ensures requests/limits are respected.

---

### Step 2: Apply it

```bash
kubectl apply -f burstable-pod.yaml
```

---

### Step 3: Verify QoS class

```bash
kubectl get pod burstable-test -o=jsonpath='{.status.qosClass}'
```

✅ Output should be:

```
Burstable
```

---

### 🔹 What this shows

* The **LimitRanger admission controller** validates resource specs.
* Kubernetes assigns QoS class at **admission time**.

---


let’s extend the demo so you see Admission Controllers actively rejecting a pod.

We’ll use the LimitRanger admission controller (enabled by default in most clusters).
If a LimitRange is set in a namespace, then pods must define resource requests/limits.
Otherwise, the Admission Controller blocks them.


We’ll create **two namespaces** — one with a *hard enforcement* LimitRange, and another with a *defaulting (mutating)* LimitRange.

---

# 🟢 Example 1: **Enforce-only (rejects pod)**

### Step 1: Create namespace

```bash
kubectl create namespace enforce-demo
```

### Step 2: Create LimitRange with only `min`/`max`

`enforce-lr.yaml`

```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: enforce-range
  namespace: enforce-demo
spec:
  limits:
  - min:
      memory: 128Mi
    max:
      memory: 512Mi
    type: Container
```

Apply it:

```bash
kubectl apply -f enforce-lr.yaml
```

### Step 3: Create pod without resources

`enforce-pod.yaml`

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: enforce-test
  namespace: enforce-demo
spec:
  containers:
  - name: nginx
    image: nginx
```

Apply it:

```bash
kubectl apply -f enforce-pod.yaml
```

👉 **Expected Result:**

```
Error from server (Forbidden): Pod "enforce-test" is forbidden:
Minimum memory usage per Container is 128Mi, but no request is specified.
```

---

# 🟢 Example 2: **Defaulting (mutates pod)**

### Step 1: Create namespace

```bash
kubectl create namespace default-demo
```

### Step 2: Create LimitRange with defaults

`default-lr.yaml`

```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: default-range
  namespace: default-demo
spec:
  limits:
  - default:
      memory: 256Mi
    defaultRequest:
      memory: 128Mi
    type: Container
```

Apply it:

```bash
kubectl apply -f default-lr.yaml
```

### Step 3: Create pod without resources

`default-pod.yaml`

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: default-test
  namespace: default-demo
spec:
  containers:
  - name: nginx
    image: nginx
```

Apply it:

```bash
kubectl apply -f default-pod.yaml
```

👉 **Expected Result:** Pod is created ✅

Now check:

```bash
kubectl get pod default-test -n default-demo -o yaml | grep -A5 resources:
```

You’ll see the Admission Controller **mutated the pod spec**:

```yaml
resources:
  requests:
    memory: "128Mi"
  limits:
    memory: "256Mi"
```

---

## 🔹 Summary

* **Enforce-only LimitRange (min/max)** → ❌ Rejects pod without resources.
* **Defaulting LimitRange (default/defaultRequest)** → ✅ Mutates pod spec with defaults.

---









