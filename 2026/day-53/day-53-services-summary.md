# Day 53 - Kubernetes Services - Learnings

## 1. What problem Services solve

Pods are **ephemeral**:

* IPs change when recreated
* Not stable for communication

Deployments manage Pods but **don’t provide stable networking**

👉 **Service solves this by:**

* Providing a **stable virtual IP (ClusterIP)**
* Load balancing traffic across Pods
* Decoupling clients from Pod lifecycle

**Flow:**

```
Client → Service (stable IP) → Pod (dynamic IPs)
```

---

## 2. Service Manifests

### 🔹 ClusterIP (default)

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-clusterip
spec:
  selector:
    app: myapp
  ports:
    - port: 80
      targetPort: 8080
```

**Explanation:**

* Internal communication only
* Accessible inside cluster via DNS
* Default type

---

### 🔹 NodePort

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-nodeport
spec:
  type: NodePort
  selector:
    app: myapp
  ports:
    - port: 80
      targetPort: 8080
      nodePort: 30007
```

**Explanation:**

* Exposes service on every node IP
* Accessible via: `NodeIP:30007`
* Useful for testing/debugging

---

### 🔹 LoadBalancer

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-loadbalancer
spec:
  type: LoadBalancer
  selector:
    app: myapp
  ports:
    - port: 80
      targetPort: 8080
```

**Explanation:**

* Creates external load balancer (cloud)
* Public IP assigned
* Production-ready external access

---

## 3. ClusterIP vs NodePort vs LoadBalancer

| Type         | Scope    | Access Method    | Use Case              |
| ------------ | -------- | ---------------- | --------------------- |
| ClusterIP    | Internal | DNS / Cluster IP | Service-to-service    |
| NodePort     | External | NodeIP:Port      | Debug / simple access |
| LoadBalancer | External | Cloud LB IP      | Production traffic    |

---

## 4. Kubernetes DNS (Service Discovery)

Kubernetes runs a DNS server (CoreDNS).

Each Service gets a DNS entry:

```
<service-name>.<namespace>.svc.cluster.local
```

Example:

```
my-clusterip.default.svc.cluster.local
```

👉 Inside cluster:

```bash
curl http://my-clusterip
```

DNS resolves → Service IP → forwarded to Pods

---

## 5. What are Endpoints

Endpoints = **actual Pod IPs behind a Service**

Service does NOT track Pods directly
It uses Endpoints object:

```
Service → Endpoints → Pod IPs
```

---

### 🔍 Inspect Endpoints

```bash
kubectl get endpoints
kubectl describe endpoints my-clusterip
```

Example output:

```
Addresses: 10.244.1.5, 10.244.1.6
```

👉 These are real Pod IPs receiving traffic

---

## 6. Screenshots 

attched in the [day-53-services.md](./day-53-services.md)
---

## 🔑 Key Takeaway

* Pods = dynamic, unreliable IPs
* Service = stable abstraction + load balancer
* Endpoints = real backend Pod IPs
* DNS = automatic service discovery

---
