# Services: ClusterIP

This lecture explains the **ClusterIP Service**, which is the Kubernetes Service type mainly used for **internal communication inside the cluster**—especially between tiers of a microservices-style application.

The instructor starts from a typical full-stack setup where different Pods run different layers: multiple **frontend** Pods, multiple **backend** Pods, a **Redis** key-value store tier, and a **MySQL** (or another database) tier. The frontend needs to talk to the backend, and the backend needs to talk to Redis and the database. The problem is that although each Pod has an IP, **Pod IPs are not stable**: Pods can be recreated at any time, and new Pods constantly come and go. So you cannot build reliable internal communication by hardcoding Pod IP addresses. There is also a load-distribution question: if one frontend Pod needs to connect to the backend tier and there are three backend Pods, **which backend Pod should it call**, and who decides?

A **Kubernetes Service** solves this by **grouping a set of Pods** and exposing them through a **single, stable interface**. For example, you create a Service for the backend Pods, and that Service becomes the single entry point for anything that needs to call the backend. When a request comes in to the Service, Kubernetes forwards it to **one of the matching Pods** behind the Service (the instructor says this forwarding happens **randomly**). You do the same for Redis (a Redis Service) so the backend Pods talk to Redis through a single stable endpoint as well. This makes it practical to deploy microservices: each tier can **scale up/down** or move across nodes without breaking communication, because other components keep calling the Service, not individual Pod IPs.

With ClusterIP specifically, the Service gets an **IP and a name inside the cluster**, and other Pods should use that **Service name** (or ClusterIP) to access it. This internal, stable virtual IP is what defines the **ClusterIP** Service.

### Creating a ClusterIP service (YAML)

As with other Kubernetes objects, you create it with a YAML definition file using the familiar structure:

- `apiVersion`
- `kind`
- `metadata`
- `spec`

For a ClusterIP Service, the instructor’s example is a backend Service called `backend`:

- **apiVersion:** `v1`
- **kind:** `Service`
- **metadata.name:** `backend`
- **spec.type:** `ClusterIP` (and the instructor emphasizes that **ClusterIP is the default**, so even if you omit `type`, Kubernetes will assume ClusterIP)
- **spec.ports:** includes `targetPort` and `port` (both set to 80 in the example)
    - `targetPort` is the port where the backend Pods are listening/exposed
    - `port` is the port exposed by the Service
- **spec.selector:** links the Service to the right Pods by matching the Pods’ labels (copied from the Pod definition file)

Here’s an example consistent with what the teacher describes:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: backend
spec:
  type: ClusterIP   # optional, since ClusterIP is the default
  selector:
    app: backend
  ports:
    - port: 80
      targetPort: 80
```

Once the file is ready, you create the Service and inspect it with:

```bash
kubectl create -f backend-service.yaml
kubectl get services # kubectl get svc
```

After creation, other Pods can reach the backend tier using the **Service name** (`backend`) or the **ClusterIP** assigned to it, rather than trying to track individual Pod IPs.