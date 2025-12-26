# Pods with YAML

In this lecture, the instructor moves from “general YAML basics” to **writing YAML specifically for Kubernetes**, focusing on creating a **Pod** using a YAML definition file. The big idea is that Kubernetes uses YAML files as input to create many kinds of objects—Pods, ReplicaSets, Deployments, Services, etc.—and although each object has its own details, they all follow a common overall structure.

A Kubernetes definition file always starts with **four required top-level fields** (root properties): **`apiVersion`**, **`kind`**, **`metadata`**, and **`spec`**. You must include these, and they must be at the top level of the YAML file.

The first field, **`apiVersion`**, tells Kubernetes which API version is being used to create the object. The correct value depends on the object type. Since this lecture is about Pods, the instructor uses **`v1`**. They mention that for other objects you may see values like **`apps/v1`**, and older groupings like `extensions/v1beta...` (they’ll explain those later), but for now `v1` is the right choice for a basic Pod.

Next is **`kind`**, which defines what type of Kubernetes object you’re creating. In this case it’s a **Pod**, so `kind: Pod`. The instructor points out that `kind` could also be things like `ReplicaSet`, `Deployment`, or `Service`, depending on what you are creating.

Third is **`metadata`**, which stores information *about* the object—mainly things like its **name** and **labels**. This section is a **dictionary**, so everything under `metadata` is indented. A lot of the instructor’s focus here is on getting indentation right: `name` and `labels` must be children of `metadata`, so they must be indented more than `metadata`, and they must be aligned with each other because they are siblings. If `labels` is indented more than `name`, YAML would interpret `labels` as a child of `name` (which is wrong). And if `metadata`, `name`, and `labels` all have the same indentation, then YAML treats them as siblings at the top level (also wrong). The specific number of spaces isn’t the important part; what matters is that the indentation consistently expresses the intended parent/child relationships.

Within metadata, the instructor highlights two important constraints. First, under **`metadata`** you can only use fields that Kubernetes expects there (like `name`, `labels`, etc.)—you can’t just invent arbitrary fields at that level. Second, within **`labels`**, you *can* choose any key/value pairs you want, because labels are intended exactly for custom identification and filtering.

The lecture explains why labels matter in real clusters: if you have hundreds of Pods running different parts of your system (front end, back end, databases), labels let you group and filter them later. For example, you might label Pods as `frontend`, `backend`, or `database` so you can target and query them more easily.

The fourth top-level field is **`spec`**, which is where you describe what you want Kubernetes to actually run and how the object should behave. The contents of `spec` vary by object type, so the instructor notes you should refer to documentation when you’re unsure. For a basic Pod, it’s straightforward: `spec` is a dictionary, and you add a field called **`containers`**. `containers` is a **list**, because a Pod can contain multiple containers (even though most Pods have just one). Each list item is a dictionary describing one container, typically with at least a **`name`** and **`image`**. In the example, the image is `nginx`, which Kubernetes pulls from a container registry (like Docker Hub).

Once the YAML file is written, you create the Pod with:

- `kubectl create -f pod-definition.yaml`

And then you verify and inspect it using:

- `kubectl get pods` to list Pods
- `kubectl describe pod <pod-name>` to see detailed info like creation time, labels, containers in the Pod, and event history

### Example YAML (matching what the instructor describes)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
spec:
  containers:
    - name: nginx-container
      image: nginx
```

This example shows the same structure the instructor is teaching: the four required top-level fields, a `metadata` section with a name and a label, and a `spec` section defining a containers list with one container using the `nginx` image.