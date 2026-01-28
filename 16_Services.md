# Services

This lecture introduces **Kubernetes Services** as the mechanism that enables reliable communication **between components inside an application** and also **between users and applications**. The instructor frames Services as the “glue” that connects groups of Pods together (front end → back end → external data source), and highlights that Services help achieve **loose coupling** in microservice-style architectures: front-end Pods can talk to back-end Pods, back-end Pods can reach databases or external sources, and users can reach the front-end.

### Why Services are needed (external access example)

The lecture uses a simple networking scenario to show why a Service is necessary. You have a Kubernetes node with an IP like **192.168.1.2**, and your laptop is on the same network with an IP like **192.168.1.1**. Your Pod lives on the internal Pod network **10.244.0.0**, and the Pod has an internal IP like **10.244.0.2**. From your laptop, you can’t directly access or ping the Pod IP because it’s on a separate internal network.

One workaround would be to **SSH into the node** (192.168.1.2) and then access the Pod internally (for example using `curl http://10.244.0.2`), but that’s not what you want. You want to access the web app directly from your laptop **without SSH**, ideally using the node’s IP address. That requires a “middle layer” that maps traffic arriving at the node to the correct Pod. That is exactly one role of a Kubernetes Service.

A **Service** is another Kubernetes object (like Pods, ReplicaSets, Deployments) and one of its use cases is to **listen on a port on the node** and **forward traffic** to a port on the Pod. This type is called a **NodePort Service**.

### Service types overview

The instructor introduces three service types:

- **NodePort**: exposes a Pod internally by opening a port on each node and forwarding traffic to the Pod.
- **ClusterIP**: creates an internal virtual IP in the cluster to enable communication between different tiers (e.g., front end → back end).
- **LoadBalancer**: provisions an external load balancer in supported cloud providers, typically for distributing traffic across front-end Pods.

The lecture then focuses mainly on **NodePort**.

---

## NodePort: the three ports you must understand

NodePort involves **three different ports**, and the instructor explains them from the Service’s point of view:

1. **targetPort**: the port on the **Pod** where the application actually listens (example: **80** for a web server). This is where the Service forwards traffic.
2. **port**: the port on the **Service object itself** (also set to **80** in the example). The Service behaves like a virtual server inside the cluster and has its own IP called the **ClusterIP**.
3. **nodePort**: the port opened on the **node** for external access (example: **30008**). NodePort must be within a valid range, which by default is **30000–32767**.

So external users hit: `NodeIP:nodePort` → Service forwards to `PodIP:targetPort`.

---

## Creating a NodePort Service with YAML

Just like other objects, Services are created using a definition file with the same top-level structure:

- `apiVersion`
- `kind`
- `metadata`
- `spec`

For a Service:

- **apiVersion** is `v1`
- **kind** is `Service`
- **metadata** contains at least a `name` (labels are possible but not needed for the basic example)
- **spec** is where Service-specific configuration lives, and for NodePort the key parts are:
    - `type: NodePort`
    - `ports:` (an array/list)
    - `selector:` (to connect the Service to the right Pods)

### Ports rules the instructor emphasizes

- `ports` is an **array**, so each mapping starts with a dash of the port fields, **only `port` is mandatory**:
- If you omit **targetPort**, Kubernetes assumes it equals `port`.
- If you omit **nodePort**, Kubernetes automatically assigns a free port in **30000–32767**.
- A Service can define **multiple port mappings** in the same Service by adding more items to the `ports` list.

### Linking Service to Pods: selectors

A Service definition that only defines ports is incomplete, because it still doesn’t say *which Pod(s)* should receive the traffic. Many Pods could be listening on port 80. The link is created using **labels and selectors**, the same pattern you saw with ReplicaSets and Deployments.

Pods are created with labels (e.g., `app: myapp`). The Service `spec.selector` includes those labels, and Kubernetes uses the selector to find matching Pods and route traffic to them.

**Example NodePort Service YAML (exactly reflecting the lecture structure):**

```yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
spec:
  type: NodePort
  selector:
    app: myapp
  ports:
    - port: 80
      targetPort: 80
      nodePort: 30008
```

After creating the Service:

```bash
kubectl create -f service-definition.yaml
kubectl get services
```

`kubectl get services` shows the Service type (NodePort), its **ClusterIP**, and the port mapping (including the NodePort).

To access the application from outside (from your laptop), you use the **node IP** and the **nodePort**:

```bash
curl http://192.168.1.2:30008
```

(or open it in a browser with the same address).

---

## How NodePort behaves with multiple Pods and multiple nodes

The lecture emphasizes that NodePort Services are not just for “one Pod” scenarios.

### Multiple Pods on the same node

If you have multiple identical web app Pods (for high availability / load balancing) and they all share the same label (e.g., `app: myapp`), then when the Service is created it discovers all matching Pods automatically and uses them as **endpoints**. You don’t have to manually list Pods anywhere. The Service effectively becomes a **built-in load balancer** for the matching Pods. The instructor notes the load distribution uses a **random algorithm**.

### Pods distributed across multiple nodes

If the matching Pods are spread across different nodes in the cluster, Kubernetes still creates the Service the same way, and **it opens the same nodePort on every node** in the cluster automatically. That means you can access the application using the IP of **any node** and the same port (30008 in the example), and Kubernetes will route your request to an appropriate Pod endpoint anywhere in the cluster.

---

## Final takeaway

Whether you have **one Pod on one node**, **many Pods on one node**, or **many Pods across many nodes**, you create the NodePort Service **the same way**. Kubernetes keeps the Service updated automatically as Pods are added or removed, so once it’s created you usually don’t need to keep modifying it.