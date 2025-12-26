# Replication Controller and ReplicaSets

This lecture introduces **ReplicationController** and **ReplicaSet** as examples of Kubernetes **controllers**, described as “the brain behind Kubernetes” because controllers continuously **monitor objects** and **take action** to keep the system in the desired state.

The problem they solve starts with a simple risk: if you run your app as a **single Pod** and that Pod crashes, users lose access. A replication-focused controller prevents that by ensuring there are **multiple instances** of the Pod running at the same time. If one fails, users can still be served by the others, improving **high availability**. The instructor also emphasizes that replication controllers (and ReplicaSets) are useful even when you want *only one* Pod: you still get self-healing, because if that one Pod dies the controller will automatically create a replacement. In general, the controller’s job is to ensure the **specified number of Pods** are running at all times—whether that number is 1 or 100.

Replication also supports **scaling**. When traffic increases, you add more Pod instances to share load. If one node runs out of resources, replicas can be placed on **other nodes**, meaning replication can span the cluster and help distribute workload across nodes as capacity grows.

### ReplicationController vs ReplicaSet

The lecture then clarifies an important distinction: **ReplicationController** and **ReplicaSet** serve the same overall purpose (maintain a desired number of Pod replicas), but they are not the same. ReplicationController is the **older** mechanism and is being replaced by ReplicaSet. ReplicaSet is the **new recommended** approach, and the instructor says the course will mostly use ReplicaSets going forward. Most concepts you learn here apply to both, with some key differences.

---

## ReplicationController (RC) concept and YAML structure

To create a ReplicationController, you write a Kubernetes definition file with the usual four top-level fields: `apiVersion`, `kind`, `metadata`, `spec`.

- **apiVersion:** for ReplicationController it is **`v1`**
- **kind:** **`ReplicationController`**
- **metadata:** name + labels (example name: `myapp-rc`; labels like `app` and `type`)
- **spec:** this is where replication is defined, and it has two crucial parts:
    1. **template:** a *Pod template* that tells the RC what Pod to create as a replica
    2. **replicas:** how many copies you want

The “template” is basically the Pod definition you already learned, embedded inside the RC spec (except you don’t repeat the Pod’s own `apiVersion` and `kind` at the top—those belong to the RC). Once you do that, it becomes clear you have two nested sets of metadata/spec: one for the RC (parent) and one for the Pod (child). The instructor also stresses indentation: `replicas` and `template` are both direct children of `spec`, so they must align as siblings.

After the file is ready, you create it with:

```bash
kubectl create -f rc-definition.yaml

```

Then you can list replication controllers and see desired/current/ready counts:

```bash
kubectl get replicationcontroller
# (often shortened as)
kubectl get rc
```

And you can list the Pods created:

```bash
kubectl get pods
```

The Pods created by an RC typically start with the RC name, showing they were generated automatically.

**Example ReplicationController YAML (as described):**

```yaml
apiVersion: v1
kind: ReplicationController
metadata:
  name: myapp-rc
  labels:
    app: myapp
    type: front-end
spec:
  replicas: 3
  template:
    metadata:
      name: myapp-pod
      labels:
        app: myapp
        type: front-end
    spec:
      containers:
        - name: nginx-container
          image: nginx
```

---

## ReplicaSet concept, YAML differences, and selectors

ReplicaSet is very similar in structure: it still has `apiVersion`, `kind`, `metadata`, and `spec`, and it still uses a Pod **template** plus a **replicas** count. The first difference is the API group/version:

- **apiVersion:** for ReplicaSet it is **`apps/v1`**
    
    If you get it wrong, Kubernetes may error with something like “no match for kind ReplicaSet” because that API version doesn’t support it.
    

The major functional difference is that a ReplicaSet **requires a selector**. The selector tells the ReplicaSet which Pods it should consider “its” Pods—i.e., which Pods it should monitor and maintain. This matters because a ReplicaSet can manage Pods that were not created by it originally, as long as those Pods match the selector labels (for example, Pods that already existed before the ReplicaSet was created). In ReplicationController, the selector isn’t required as input; if you omit it, it effectively assumes the selector matches the labels in the Pod template. In ReplicaSet, you must explicitly provide it, typically using **`matchLabels`**.

**Example ReplicaSet YAML (as described):**

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: myapp-rs
  labels:
    app: myapp
    type: front-end
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
      type: front-end
  template:
    metadata:
      labels:
        app: myapp
        type: front-end
    spec:
      containers:
        - name: nginx-container
          image: nginx

```

Notice how the **labels in the Pod template** match the **selector**. That relationship is what allows the ReplicaSet to identify which Pods belong to it.

---

## Why labels and selectors matter

The instructor uses a scenario where you already have three Pods running a frontend app. You want a ReplicaSet to ensure there are always three running. The ReplicaSet is essentially a monitoring process—but it needs to know **which Pods to monitor** among potentially hundreds. Labels on Pods provide that identity, and the ReplicaSet uses a selector (like `matchLabels`) as a filter to pick the right Pods. This label/selector pattern appears throughout Kubernetes, not just here.

A subtle but important point: even if the ReplicaSet is created when the required number of matching Pods already exist (so it doesn’t create any new Pods immediately), the **template is still required**. The template is needed so that if a Pod fails later, the ReplicaSet knows how to create a replacement Pod to get back to the desired replica count. TODO!

---

## Scaling a ReplicaSet

To scale from 3 replicas to 6, the lecture presents two common methods:

1. **Edit the YAML and apply it**
    
    Update `replicas: 6` in the file, then:
    

```bash
kubectl apply -f rs-definition.yaml
```

1. **Scale from the command line**
    
    You can use `kubectl scale` and provide either the file or the ReplicaSet name:
    

```bash
kubectl scale --replicas=6 -f rs-definition.yaml
# or
kubectl scale --replicas=6 replicaset myapp-rs
```

The instructor warns about an important detail: if you scale using the command line with the file/name, Kubernetes will scale the live object, but **your YAML file won’t automatically update**—it might still say 3 even though the cluster is now running 6.

They also mention there are advanced ways to auto-scale based on load, but that comes later.

---

## Commands reviewed at the end

The lecture ends by quickly reviewing the main CLI actions you’ll use around ReplicaSets:

```bash
kubectl create -f <file.yaml> # create objects (ReplicaSet, etc.)
kubectl get replicaset # list ReplicaSets (often: kubectl get rs)
kubectl get pods # see Pods created/managed
kubectl delete replicaset <name> # delete a ReplicaSet
kubectl replace -f <file.yaml> # replace/update (another way to update objects)
kubectl scale --replicas=<n> ... # scale without editing files
```

Overall, the key mental model is that ReplicationController/ReplicaSet continuously enforce a desired replica count for Pods (availability + scaling), and ReplicaSet is the modern approach mainly distinguished by its explicit and more flexible **label selector** mechanism.