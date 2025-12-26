# Pods

This lecture introduces **Pods** as the core unit Kubernetes uses to run applications, and it starts by assuming a few things are already true: your application has been built into **Docker images** and pushed to a registry such as **Docker Hub** (so Kubernetes can pull it), and you already have a working Kubernetes cluster (single-node or multi-node). The cluster services must be up and running because the ultimate goal is to deploy your app as containers onto the worker nodes in the cluster.

A key idea is that **Kubernetes does not deploy containers directly onto worker nodes**. Instead, containers are wrapped inside a Kubernetes object called a **Pod**. A Pod represents a **single instance of an application**, and it is the **smallest object you can create in Kubernetes**. In the simplest scenario, you can imagine one node running one Pod that contains one container running your application.

When traffic increases and you need to scale, Kubernetes scales by creating **more Pods**, not by adding more containers to the same Pod. So if your web app needs a second instance, you don’t “add another container into the existing Pod”; you create a **new Pod** with another instance of the same application. If demand grows further and the current node runs out of capacity, you can place additional Pods on **another node** by expanding the cluster (adding nodes). The main scaling mental model in this lecture is: Pods usually have a **1-to-1 relationship** with the application container, and scaling up/down is done by **creating or deleting Pods**.

That said, Pods are not limited to a single container. A Pod **can contain multiple containers**, but the lecture emphasizes that this is typically **not** used to run multiple copies of the same app for scaling. Multiple containers in one Pod are for cases where you have a **supporting/helper container** that must live tightly alongside the main application container—examples given include processing user-entered data or handling files uploaded by users. In this “sidecar/helper” style pattern, both containers are part of the same Pod so they share the same lifecycle: when the Pod is created, both start together; when the Pod is removed or dies, both go away together. Because they are inside the same Pod, these containers share the **same network namespace**, so they can talk to each other via **localhost**, and they can also share **storage** easily.

To make the idea feel more intuitive, the instructor explains the same problem in “plain Docker” terms. If you were manually managing containers, you might start your app by running it with something like `docker run ...`, and when load increases you’d run more instances. But if your architecture evolves and you introduce a helper container that must pair 1-to-1 with each app container, your operational complexity jumps: you must keep track of which helper goes with which app container, create networking links or custom networks, create and manage shared volumes, and also monitor the app container so that when it dies you remember to kill the corresponding helper container (and when you create a new app container, you also create its helper). The point is that **Kubernetes Pods automate that pairing**: you declare what containers belong in the Pod, and Kubernetes gives them shared networking, shared storage options, and a shared “fate” (created and destroyed together). Even if today you only need one container per Pod, Kubernetes still requires Pods, and the instructor frames that as beneficial because it prepares you for future architectural changes.

The lecture also notes that **multi-container Pods are relatively rare**, and for the course they’ll mostly stick to the common approach: **one container per Pod**.

Finally, it briefly shows how Pods are deployed and inspected. The `kubectl run` command (introduced earlier) effectively deploys a container by **creating a Pod**. In the example, it creates a Pod and runs an instance of the **nginx** image. You specify which image to use via the image parameter, and Kubernetes pulls that image from a registry such as **Docker Hub** (or it could be configured to pull from a private registry inside an organization). After creating the Pod, you can list Pods with `kubectl get pods`, where you’ll see state transitions such as “ContainerCreating” before it becomes “Running.” The instructor also points out an important limitation at this stage: simply running the Pod does not automatically make the nginx service accessible to external users—at this point you can access it internally from the node, and the details of exposing it to end users will be covered later when networking and **Services** are introduced.

# Pod demostration

In this demo, the instructor shows the simplest way to deploy a **Pod** into a **minikube** cluster and then inspect it using `kubectl`. The starting assumption is that, if you previously set up minikube, your `kubectl` is already configured to talk to that cluster, so you can immediately begin running commands against it.

To create the Pod, they run a command in the form:

- `kubectl run nginx --image=nginx`

Here, **`nginx`** is the Pod name (and it could technically be any name you choose), while `--image=nginx` is where you specify the **container image** Kubernetes should run. The image name must exist in a container registry such as **Docker Hub**, unless you provide an alternative registry address. You can also specify an image **tag** (e.g., a specific version), or point to an image stored outside Docker Hub by using its full registry path.

After running the command, Kubernetes creates the Pod, and the instructor checks its status with:

- `kubectl get pods`

From this output, you can see the Pod name (`nginx`), its **status** (e.g., Running), and other useful columns. The demo calls out the meaning of the common fields: the **READY** column shows how many containers in the Pod are ready (and out of how many total), the **RESTARTS** column indicates whether the container(s) have restarted since creation, and the **AGE** column shows how long the Pod has been running.

To get deeper details, the instructor uses:

- `kubectl describe pod nginx`

This provides far more information than `get`. It shows the Pod name and the **labels** that were automatically added when the Pod was created using `kubectl run`. It also shows when the Pod started and, importantly, which **node** it was scheduled onto. Because this is a single-node minikube cluster, the node is named **minikube**, and the output includes the node’s IP address. The description also shows the Pod’s own IP address (in the demo, something like `172.16.0.3`), which is the Pod’s **internal cluster IP**—the instructor notes they’ll explain Pod IPs more later in the networking section.

The `describe` output also lists container-level information. In this demo there is **one container** using the `nginx` image, and if there were multiple containers, they would appear in that section. The instructor notes that multi-container Pods will be covered later. You can also see that the nginx image was pulled from **Docker Hub**.

At the bottom of `kubectl describe`, the demo highlights the **Events** section, which is a timeline of what happened as the Pod came to life. You can see steps like the Pod being **assigned** to the minikube node, the image being **pulled** successfully, and then the container being **created** and **started**.

Finally, they show a more informative version of the listing command:

- `kubectl get pods -o wide`

This adds extra columns such as the **node** the Pod is running on and the Pod’s **IP address**, reinforcing the idea that each Pod gets an internal IP within the Kubernetes cluster. The demo ends by saying that upcoming videos will move beyond `kubectl run` and show how to create Pods using a **YAML definition file**, which is the more declarative and common approach in real Kubernetes work.