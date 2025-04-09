# <p align="center">Kubernetes for Certified Kubernetes Administrator (CKA)</p>


# <p align="center">Core Concepts</p>

## Kubernetes Cluster Architecture
- [Refer Here](https://kubernetes.io/docs/concepts/architecture/) for the Official docs.
- Kubernetes (K8s) is used to manage containerized applications automatically.
- It enables easy deployment, scaling, and communication between services.
- **Cluster Structure:**
  - **Master Node (Control Plane)**: Manages the cluster.
  - **Worker Nodes**: Run containerized applications.
  ![preview](./Images/Kubernetes_CKA1.png)
### Master Node Components
- **etcd**: A key-value store that holds cluster data.
- **Kube API Server**: Main communication hub, exposing the Kubernetes API.
- **Scheduler**: Assigns workloads (Pods) to worker nodes based on resources and constraints.
- **Controllers**: Maintain cluster state (e.g., Node Controller, Replication Controller).
### Worker Node Components
- **Kubelet**: The agent that manages containers on the node, communicating with the API Server.
- **Kube Proxy**: Manages networking and enables communication between Pods.
- **Container Runtime**: Runs containers (Docker, containerd, etc.).

## Docker vs. ContainerD
- Docker and containerd are both container runtimes.
- Older Kubernetes versions supported Docker, while newer versions use containerd.
- Several CLI tools exist to interact with these runtimes: `ctr`, `crictl`, `nerdctl`.
### History of Containers
- Initially, **Docker** was the dominant container tool.
- Kubernetes was built to work **only with Docker** in the beginning.
- Other container runtimes (e.g., **rkt**) wanted support in Kubernetes.
### CRI (Container Runtime Interface)
- Kubernetes introduced **CRI** to support multiple container runtimes.
- **CRI follows OCI (Open Container Initiative) standards**:
  - **ImageSpec**: Defines how container images should be built.
  - **RuntimeSpec**: Defines how container runtimes should work.
- Now, any runtime that follows **OCI standards** can work with Kubernetes.
- **Docker was NOT built for CRI** since it existed before CRI.
- Kubernetes had to create **Dockershim** (a workaround) to support Docker.
### Shift from Docker to ContainerD
- Docker is a full suite with CLI, API, build tools, security, volumes, `runc`, and `containerd`.
- **Containerd is the core runtime inside Docker.** It is **CRI-compatible**, so Kubernetes can use it **without Docker**.
- Now, **Containerd is a separate project** under CNCF. You can install **Containerd directly** without installing Docker.
- Kubernetes **removed `dockershim` in v1.24**, eliminating direct Docker support.
- Docker-built images still work because they follow OCI standards.
### CLI Tools for Working with Containers
- **`ctr` (ContainerD CLI):**
  - CLI tool for **containerd**.
  - Used mainly for debugging.
  - Not user-friendly, supports only basic functions.
- **`nerdctl` (Docker-like CLI for ContainerD):**
  - Docker-compatible CLI for containerd.
  - Supports additional features like **encrypted images, P2P distribution**.
  - Works almost like Docker CLI.
- **`crictl` (CRI CLI for Kubernetes Runtimes):**
  - Kubernetes-specific CLI to interact with **CRI-compatible runtimes** (containerd, CRI-O, etc.).
  - Used for **debugging and troubleshooting**, not creating containers.
  - Unlike Docker, `crictl` can manage **pods** (`crictl pods`).

## ETCD
- [Refer Here](https://etcd.io/) for the Official site.
- ETCD is a **distributed, reliable key-value store** used in Kubernetes.
- It is **simple, secure, and fast**.
- **Key-Value Store:**
  - Traditional databases store data in tables (rows & columns).
  - A **key-value store** stores data as **key-value pairs**.
  ![preview](./Images/Kubernetes_CKA2.png)
- By default, ETCD runs on **port 2379**.
- Use the `etcdctl` command-line tool to interact with ETCD.
- **Basic Commands:**
  ```bash
  ./etcdctl put key1 value1
  # To store a key-value pair
  ./etcdctl get key1
  # To retrieve stored data
  ./etcdctl
  # To view available commands
  ```
### ETCD Versioning
- API versions **v2.0 vs v3.0** have different `etcdctl` commands.
- **Check ETCDCTL Version:** `etcdctl version`
  - The output shows the **ETCDCTL utility version** and **API version**.
  - **Default API version is v2.0**, but Kubernetes uses v3.0.
- **To set API Version 3 Use:** `export ETCDCTL_API=3`
### ETCD in Kubernetes
- [Refer Here](https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/) for the Official docs.
- Stores cluster data (nodes, pods, configs, secrets, roles, etc.).
- All `kubectl get` commands retrieve data from ETCD.
- Cluster changes (e.g., adding nodes, deploying pods) are recorded in ETCD.
#### Installation of ETCD
- **Manually**:
  - If you setup your Cluster from scratch then you deploy `ETCD` by downloading ETCD Binaries:
    ```sh
    wget -q --https-only "https://github.com/etcd-io/etcd/releases/download/v3.3.11/etcd-v3.3.11-linux-amd64.tar.gz"
    ```
  - Install Binaries and Configure `ETCD` as a service in your master node.
  - `ETCD` runs as a **systemd service**.
- **Using kubeadm**:
  - Automatically deployed as a pod in the `kube-system` namespace.
    ```sh
    kubectl get pods -n kube-system | grep etcd-master
    ```
#### EETCD Data Structure in Kubernetes
- List all keys stored in ETCD:
  ```bash
  kubectl exec etcd-master -n kube-system -- sh -c "ETCDCTL_API=3 etcdctl \
                --cert=/etc/kubernetes/pki/etcd/server.crt \
                --key=/etc/kubernetes/pki/etcd/server.key \
                --cacert=/etc/kubernetes/pki/etcd/ca.crt get / --prefix --keys-only"
  ```
- Data is stored in a **directory-like structure**.
- Kubernetes stores data under the **`registry`** directory (The Root directory).
  - Nodes: `/registry/minions`
  - Pods: `/registry/pods`
  - Deployments: `/registry/deployments`
#### ETCD in High Availability (HA) Setup
- Multiple master nodes run multiple **ETCD instances**.
- ETCD instances must be **aware of each other** (configured via `initial-cluster` option).
- **ETCD must always have an odd number of nodes** in an HA setup to **avoid split-brain scenarios**.
- Ensures redundancy and fault tolerance.
##### Add that `backups` are critical in ETCD, and the `etcdctl snapshot save` command is used for backups.

## Kube API Server
- The **main component** of Kubernetes that **manages all requests**.
- Responsible for:
  - **Authentication** (who can access)
  - **Validation** (checking if requests are correct)
  - **Retrieving & Updating** data in **ETCD** (the cluster database).
- It is the **only component** that interacts directly with ETCD.
- Other components (Scheduler, Controller Manager, Kubelet) **use the API Server** to update the cluster.
### Installation of Kube API Server
- **Manually**:
  - If you installing manually (`the hard way`), download the binary from Kubernetes releases:
    ```sh
    wget https://storage.googleapis.com/kubernetes-release/release/v1.13.0/bin/linux/amd64/kube-apiserver
    ```
  - Install and Configure `Kube API Server` as a service in your master node.
  - `Kube API Server` runs as a **systemd service**.
  - Config is in: `/etc/systemd/system/kube-apiserver.service`
  - Running processes can be checked using `ps aux | grep kube-apiserver`.
- **Using kubeadm**:
  - Automatically deployed as a pod in the `kube-system` namespace.
    ```sh
    kubectl get pods -n kube-system | grep kube-apiserver-master
    ```
  - Config is in: `/etc/kubernetes/manifests/kube-apiserver.yaml`

## Kube Controller Manager
- It **manages all controllers** in Kubernetes.
- A **controller** is responsible for monitoring the cluster state and ensuring it reaches the desired condition.
- Works through the **kube-apiserver** to track and control cluster components.
- **Types of Controllers:**
  1. **Node Controller**
     - Checks the status of nodes every **5 seconds**.
     - Marks a node **unreachable** after **40 seconds**.
     - Waits **5 minutes** before reassigning pods to healthy nodes.
  2. **Replication Controller**
     - Ensures the desired number of pods in a **ReplicaSet** is always maintained.
     - If a pod dies, it creates a new one automatically.
  3. **Other Controllers:**
     - **Deployment Controller**: Manages Deployments.
     - **Service Controller**: Handles Services.
     - **Namespace Controller**: Manages namespaces.
     - **Persistent Volume Controller**: Manages storage.
     - **Job Controller**: Manages Jobs & CronJobs.
### Installation of Kube Controller Manager
- Comes as a **single process** that includes all controllers.
- **Manually**:
  - Download the kube-controller-manager binary from the kubernetes release page:
    ```sh
    wget https://storage.googleapis.com/kubernetes-release/release/v1.13.0/bin/linux/amd64/kube-controller-manager
    ```
  - Install and Configure `Kube Controller Manager` as a service in your master node.
  - `Kube Controller Manager` runs as a **systemd service**.
  - By default all controllers are enabled, but you can choose to enable specific one from `kube-controller-manager.service`.
  - Config is in: `/etc/systemd/system/kube-controller-manager.service`
  - Running processes can be checked using `ps aux | grep kube-controller-manager`.
- **Using kubeadm**:
  - Automatically deployed as a pod in the `kube-system` namespace.
    ```sh
    kubectl get pods -n kube-system | grep kube-controller-manager-master
    ```
  - Config is in: `/etc/kubernetes/manifests/kube-controller-manager.yaml`
### Customizing Controllers
- You can pass **options** (modify configuration) when running the Kube Controller Manager.
- Example settings:
  - **Node monitor period** (How often nodes are checked).
  - **Grace period** (Time before marking a node as failed).
  - **Eviction timeout** (When to reassign pods).
- `--controllers` option allows enabling/disabling specific controllers.

## Kube-Scheduler
- **Kube-Scheduler** is responsible for **deciding** which node a pod should be placed on.
- **It does not create pods** on nodes—that's the job of the **kubelet**.
### Work of Kube-Scheduler
- **Filters Nodes:**
   - Eliminates nodes that **don’t meet the pod’s resource requests** (CPU, memory, etc.).
   - **Example:** If a pod needs 4 CPU but a node only has 2 CPU available, that node is filtered out.
- **Ranks Remaining Nodes:**
   - Assigns a **score (0 to 10)** to each valid node.
   - The node with the **highest score** gets selected.
   - **Example:** If one node would have **6 CPUs free** after placing the pod and another only **2 CPUs free**, the first one gets a **higher score** and wins.
### Customizing Scheduling Rules
- Kubernetes allows **custom schedulers** to be defined.
- Additional factors like **taints, tolerations, affinity rules, and node selectors** influence scheduling decisions (covered in detail later).
### Installation of Kube-Scheduler
- **Manually**:
  - Download the kube-scheduler binary from the kubernetes release page:
    ```sh
    wget https://storage.googleapis.com/kubernetes-release/release/v1.13.0/bin/linux/amd64/kube-scheduler
    ```
  - Install and Configure `Kube-Scheduler` as a service in your master node.
  - `Kube-Scheduler` runs as a **systemd service**.
  - Config is in: `/etc/systemd/system/kube-scheduler.service`
  - Running processes can be checked using `ps aux | grep kube-scheduler`.
- **Using kubeadm**:
  - Automatically deployed as a pod in the `kube-system` namespace.
    ```sh
    kubectl get pods -n kube-system | grep kube-scheduler-master
    ```
  - Config is in: `/etc/kubernetes/manifests/kube-scheduler.yaml`

## Kubelet
- **Kubelet** is the **primary agent** running on every worker node in a Kubernetes cluster.
- It **registers the node** with the cluster and ensures that pods run correctly.
- It is like the **captain of a ship**, handling all tasks related to containers on that node.
- Kubelet **does not manage containers directly**—it interacts with the **container runtime via CRI**.
- Kubelet **periodically sends node status to the API Server** to ensure the node remains active.
### Functions of Kubelet
- **Registers the worker node** with the Kubernetes cluster.
- **Receives pod instructions** from the scheduler.
- **Interacts with the container runtime** (e.g., Docker, containerd) to pull images and create containers.
- **Monitors the health** of pods and containers.
- **Reports node and pod status** back to the Kube API server.
### Installation of Kubelet
- **If using `kubeadm`** unlike other components, **kubeadm does not automatically install kubelet** on worker nodes.
- It must be **manually installed** on each worker node.
- **Steps to Install:**
  ```sh
  sudo apt-get update
  sudo apt-get install -y kubelet
  ```
- After installation, it runs as a **service**.
- **To Check Kubelet Status:**
  ```sh
  sudo systemctl status kubelet
  # Check if kubelet is running
  sudo systemctl restart kubelet
  # Start/Restart kubelet service
  ps aux | grep kubelet
  # Check running kubelet process
  ```

## Kube-Proxy
- `kube-proxy` is a **networking component** that runs on every node in a Kubernetes cluster.
- It **forwards traffic** from services to the correct backend pods.
- It ensures that **every pod can communicate with every other pod** inside the cluster.
- **Manages networking rules** for Kubernetes services.
- Uses **iptables** (or other networking tools) to create routing rules.
### Services Work with Kube-Proxy
- Pods communicate using **Pod IPs**, but Pod IPs can change.
- Services provide a **fixed IP** and **DNS name** to access backend pods.
- `kube-proxy` ensures that **requests to the service** are forwarded to the right pod.
- **Example:**
  - A web app pod needs to talk to a database pod.
  - Instead of using the database pod's IP (which can change), it uses a **service** (`db-service`).
  - `kube-proxy` forwards traffic from `db-service` to the correct database pod.
### Installation of Kube-Proxy
- **Deployed as a DaemonSet**, runs on **every node** in the cluster.
- **Manually**:
  - Download the kube-proxy binary from the kubernetes release page:
    ```sh
    wget https://storage.googleapis.com/kubernetes-release/release/v1.13.0/bin/linux/amd64/kube-proxy
    ```
  - Install and Configure `Kube-Proxy` as a service in your master node.
  - `Kube-Proxy` runs as a **systemd service**.
  - Config is in: `/etc/kubernetes/kube-proxy-config.yaml` or `/etc/kube-proxy/config.yaml`
  - Running processes can be checked using `ps aux | grep kube-proxy`.
- **Using kubeadm**:
  - Automatically deployed as a pod in the `kube-system` namespace.
    ```sh
    kubectl get pods -n kube-system
    ```
  - Config is in: `/var/lib/kube-proxy/config.conf`

## Kubernetes Pods
- [Refer Here](https://kubernetes.io/docs/concepts/workloads/pods/) for the Official docs.
- **Pod** is the **smallest deployable unit** in Kubernetes.
- A **pod encapsulates one or more containers**.
- Containers inside a pod **share the same network** and **storage**.
- Pods are used to **deploy your application containers** on Kubernetes.
- Kubernetes **does not deploy containers directly**, it **deploys Pods**.
- Usually, **one pod = one container** (one-to-one relationship).
- Containers inside a pod run on the **same node** and can communicate through `localhost`.
  ![preview](./Images/Kubernetes_CKA3.png)
### Scaling
- Want to scale? **Don’t add containers to the same pod.**
- **Create more pods** (each with one container).
- To scale down, **delete pods**.
- Pods can be spread across **multiple nodes** if needed.
### Multi-Container Pods
- A pod **can have multiple containers**, but:
  - They’re **not multiple replicas** of the same app.
  - Usually used for **helper or sidecar containers** (e.g., logging, syncing).
### Deploying a Pod using `kubectl`
- To Create a Pod:
  ```sh
  kubectl run <pod_name> --image=<image_name>
  ```
- Check the list of running pods:
  ```sh
  kubectl get pods
  ```
  ![preview](./Images/Kubernetes_CKA4.png)
- Help Command:
  ```sh
  kubectl run --help
    # A description of what `kubectl run` does.
    # The usage pattern, like: how to use it, its syntax, options, and examples.
  ```
### Pod YAML Configuration
- [Refer Here](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.32/) for `One-page API Reference for Kubernetes` and choose required Version.
  - Select required API, in this case `Pod`.
- [Refer Here](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.32/#pod-v1-core) for the **Pod Workloads APIs**.
  - `apiVersion` → Group/Version
    - If **Group** is `core`, provide only `Version`
    - If **Group** is other than `core`, provide `Group/Version`
  - `kind` → Kind
  - `metadata` & `spec` → Use Field and Description.
- **Must-Have Root Keys:** Every Kubernetes Manifest YAML files must have these 4 top level fields.
  - `apiVersion`: Version of Kubernetes API
  - `kind`: Type of object
  - `metadata`: Data about the object
  - `spec`: Details about what to run
- **Nginx Pod YAML:**
  - Based on `Pod Workloads APIs`, Write the Pod YAML file:
    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: nginx-pod
      labels:
        app: nginx
        tier: frontend
    spec:
      containers:
        - name: nginx
          image: nginx
    ```
- **Commands to Work with Pods:**
  ```sh
  kubectl create -f <filename>
  # Create a pod using YAML
  kubectl get pods
  # View all pods
  kubectl get pods -o wide
  # View all pods with more details
  kubectl describe pod <name>
  # Detailed pod info
  kubectl exec -it <pod-name> -- <command>
  # Execute a command inside a pod
  kubectl delete pod <name>
  # Delete pod
  ```

## Kubernetes Controllers
- Controllers are **the brain of Kubernetes**.
- They **monitor Kubernetes objects** and take action when needed.
- Example: Replication Controller, ReplicaSet, etc.
### Replication Controller (RC)
- [Refer Here](https://kubernetes.io/docs/concepts/workloads/controllers/replicationcontroller/) for the Official docs.
- Ensures a **specific number of pods** are running at all times.
- Provides **high availability** by running multiple copies of a pod.
- Can be used even for **1 pod** to auto-recover if it fails.
- **Use Cases:**
  - Prevent app downtime by having multiple pod replicas.
  - Load balancing across multiple pods.
  - Auto-healing (restart pods if they crash).
  ![preview](./Images/Kubernetes_CKA5.png)
#### Replication Controller YAML Configuration
- [Refer Here](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.32/) for `One-page API Reference for Kubernetes` and choose required Version.
  - Select required API, in this case `ReplicationController`.
- [Refer Here](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.32/#replicationcontroller-v1-core) for the **ReplicationController Workloads APIs**.
  - `apiVersion` → Group/Version
    - If **Group** is `core`, provide only `Version`
    - If **Group** is other than `core`, provide `Group/Version`
  - `kind` → Kind
  - `metadata` & `spec` → Use Field and Description.
- **`apiVersion`, `kind`, `metadata` & `spec`** these are must have Root Keys in every Kubernetes Manifest YAML files.
- **Nginx Replication Controller YAML:**
  - Based on `ReplicationController Workloads APIs`, Write the Replication Controller YAML file:
    ```yaml
    apiVersion: v1
    kind: ReplicationController
    metadata:
      name: myapp-rc
      labels:
        app: myapp
        type: frontend
    spec:
      replicas: 3
      template:
        metadata:
          labels:
            app: myapp
            type: frontend
        spec:
          containers:
          - name: nginx-container
            image: nginx
    ```
- **Commands:**
  ```sh
  kubectl create -f <filename>
  # Create replication controller using YAML
  kubectl get replicationcontroller
  # View all replication controllers
  kubectl delete replicationcontroller <name>
  # Delete replication controller
  ```
### ReplicaSet (RS)
- [Refer Here](https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/) for the Official docs.
- **Modern version of Replication Controller** (recommended to use).
- Works **similar to RC** but more **flexible with label selectors**.
- **Key Difference:**
  - Requires a **selector** (matchLabels) to track pods.
  - Can **adopt existing pods** if they match the selector.
  ![preview](./Images/Kubernetes_CKA6.png)
#### ReplicaSet YAML Configuration
- [Refer Here](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.32/) for `One-page API Reference for Kubernetes` and choose required Version.
  - Select required API, in this case `ReplicaSet`.
- [Refer Here](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.32/#replicaset-v1-apps) for the **ReplicaSet Workloads APIs**.
  - `apiVersion` → Group/Version
    - If **Group** is `core`, provide only `Version`
    - If **Group** is other than `core`, provide `Group/Version`
  - `kind` → Kind
  - `metadata` & `spec` → Use Field and Description.
- **`apiVersion`, `kind`, `metadata` & `spec`** these are must have Root Keys in every Kubernetes Manifest YAML files.
- **Nginx ReplicaSet YAML:**
  - Based on `ReplicaSet Workloads APIs`, Write the ReplicaSet YAML file:
    ```yaml
    apiVersion: apps/v1
    kind: ReplicaSet
    metadata:
      name: myapp-rs
      labels:
        app: myapp
    spec:
      replicas: 3
      selector:
        matchLabels:
          app: myapp
      template:
        metadata:
          labels:
            app: myapp
        spec:
          containers:
          - name: nginx-container
            image: nginx
    ```
- **Commands:**
  ```sh
  kubectl create -f <filename>
  # Create replicaset using YAML
  kubectl get replicasets
  # View all replicasets
  kubectl delete rs <name>
  # Delete replicaset
  kubectl replace -f <file>
  # Update a resource
  ```
#### Scaling ReplicaSet
- **Option:1** Modify YAML file
  - Update `replicas: 3` to `replicas: 6`
  - Run: `kubectl replace -f rs-definition.yaml`
- **Option:2** Using CLI directly
  - Run:  
    ```sh
    kubectl scale --replicas=6 -f rs-definition.yaml
                           or
    kubectl scale replicaset myapp-rs --replicas=6
    ```
  - **Note:** If you scale using the CLI, the YAML file will not auto-update.
### Labels & Selectors
- **Labels**: Key-value pairs added to pods (like `app: myapp`)
- **Selectors**: Used by ReplicaSet to identify which pods to manage.
- Helps ReplicaSet **monitor** the right pods among many.

## Kubernetes Deployments
- [Refer Here](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/) for the Official docs.
- A **Deployment** in Kubernetes is a higher-level object used to **manage applications**.
- It handles:
  - Running **multiple instances (replicas)** of your app.
  - **Upgrading** your app with **rolling updates**.
  - **Rolling back** to a previous version if needed.
  - **Pausing and resuming** changes for better control.
  ![preview](./Images/Kubernetes_CKA7.png)
### Real-World Deployment Needs
- **Multiple Instances:** Run several copies of your app (e.g., multiple web servers) for **high availability**.
- **Rolling Updates:** Update pods **one at a time**, not all at once — avoids app downtime for users.
- **Rollback:** If a new version causes issues, **revert** to the previous stable version easily.
- **Pause and Resume Updates:** Make **multiple changes** (like scaling, version update, resource config) and **apply all at once** after reviewing.
### Deployment vs ReplicaSet vs Pods
- **Pod:** Runs a single instance of your container
- **ReplicaSet:** Manages **multiple identical pods**
- **Deployment:** Manages **ReplicaSets**, adds **upgrades, rollbacks, pause/resume** features
  - Think of Deployment as the **controller** for ReplicaSet + extra powers.
### Deployment YAML Configuration
- [Refer Here](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.32/) for `One-page API Reference for Kubernetes` and choose required Version.
  - Select required API, in this case `Deployment`.
- [Refer Here](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.32/#deployment-v1-apps) for the **Deployment Workloads APIs**.
  - `apiVersion` → Group/Version
    - If **Group** is `core`, provide only `Version`
    - If **Group** is other than `core`, provide `Group/Version`
  - `kind` → Kind
  - `metadata` & `spec` → Use Field and Description.
- **`apiVersion`, `kind`, `metadata` & `spec`** these are must have Root Keys in every Kubernetes Manifest YAML files.
- **Nginx Deployment YAML:**
  - Based on `Deployment Workloads APIs`, Write the Deployment YAML file:
  - Similar to ReplicaSet, just change `kind: Deployment`
    ```yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: myapp-deployment
      labels:
        app: myapp
    spec:
      replicas: 3
      selector:
        matchLabels:
          app: myapp
      template:
        metadata:
          labels:
            app: myapp
        spec:
          containers:
          - name: nginx-container
            image: nginx
    ```
- **Commands:**
  ```sh
  kubectl create -f <filename>
  # Create deployment
  kubectl get deployments
  # List all deployments
  kubectl get all
  # View all resources (deployments, RS, pods, etc.)
  kubectl delete deploy <name>
  # Delete deployment
  ```
- **When You Create a Deployment:**
  - Deployment is created.
  - Deployment creates a **ReplicaSet**.
  - ReplicaSet creates **pods**.

## Kubernetes Services
- [Refer Here](https://kubernetes.io/docs/concepts/services-networking/service/) for the Official docs.
- Services allow communication **between Pods**, **between apps**, and **from external users** to Pods.
- Services **abstract** the dynamic IPs of Pods (since Pods can be recreated and get new IPs).
- Services allow **stable access** to Pods.
- **Types of Services:**
  - NodePort
  - ClusterIP
  - LoadBalancer
  ![preview](./Images/Kubernetes_CKA8.png)
### Services YAML Configuration
- [Refer Here](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.32/) for `One-page API Reference for Kubernetes` and choose required Version.
  - Select required API, in this case `Service`.
- [Refer Here](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.32/#deployment-v1-apps) for the **Service in Service APIs**.
  - `apiVersion` → Group/Version
    - If **Group** is `core`, provide only `Version`
    - If **Group** is other than `core`, provide `Group/Version`
  - `kind` → Kind
  - `metadata` & `spec` → Use Field and Description.
  - `type in spec` → Give Type of Service
- **`apiVersion`, `kind`, `metadata` & `spec`** these are must have Root Keys in every Kubernetes Manifest YAML files.
### Types of Services
#### NodePort
- Makes a Pod accessible **from outside the cluster** using `<NodeIP>:<NodePort>`.
- Opens a **specific port on every Node** (between `30000-32767`).
- Traffic goes from NodePort ➝ ClusterIP ➝ Pod.
  ![preview](./Images/Kubernetes_CKA9.png)
- **NodePort Service YAML:**
  ```yaml
  apiVersion: v1
  kind: Service
  metadata:
    name: my-nodeport-service
  spec:
    type: NodePort
    selector:
      app: myapp
    ports:
      - port: 80         # Service port (ClusterIP)
        targetPort: 80   # Pod port
        nodePort: 30008  # Node's exposed port
  ```
- **Notes:**
  - `port` = service’s own port
  - `targetPort` = Pod’s port
  - `nodePort` = port exposed on Node (external access)
  - If `targetPort` not specified ➝ assumed same as `port`
  - If `nodePort` not specified ➝ random port is chosen from range
#### ClusterIP (Default)
- Used for **internal communication** between services in the cluster.
- Each service gets a **virtual IP (ClusterIP)**.
- **Not accessible from outside the cluster**.
  ![preview](./Images/Kubernetes_CKA10.png)
- **ClusterIP Service YAML:**
  ```yaml
  apiVersion: v1
  kind: Service
  metadata:
    name: redis
  spec:
    type: ClusterIP
    selector:
      app: redis-pod
    ports:
      - port: 6379
        targetPort: 6379
  ```
#### LoadBalancer
- Exposes service **externally using cloud provider’s load balancer** (AWS, GCP, Azure).
- Best for production **external traffic**.
- **LoadBalancer Service YAML:**
  ```yaml
  apiVersion: v1
  kind: Service
  metadata:
    name: frontend
  spec:
    type: LoadBalancer
    selector:
      app: frontend
    ports:
      - port: 80
        targetPort: 80
  ```
- Creates external IP via cloud LB ➝ traffic gets routed to NodePort ➝ ClusterIP ➝ Pod.
### Service Selectors and Labels
- Services **use selectors** to find matching Pods.
- Pods must have **matching labels** for the service to route traffic.
- **Example:**
  ```yaml
  selector:
    app: myapp
  ```
### Load Balancing Behavior
- Services **load balance automatically** between matching Pods.
- Uses **random algorithm** to distribute traffic.
- Works across **multiple nodes** too:
  - NodePort is open on **every node**
  - Requests can hit any node ➝ routed to correct Pod
### Commands & Real-World Use Cases
- **Commands:**
  ```sh
  kubectl get svc
  # List all services
  kubectl describe svc <svc-name>
  # Detailed service info
  kubectl expose pod <pod-name> --type=NodePort --port=80
  # Quickly expose a pod as a NodePort
  kubectl delete svc <name>
  # Delete service
  ```
- **Use Case:**
  - **ClusterIP:** Internal microservice communication
  - **NodePort:** Access app from browser on Node IP
  - **LoadBalancer:** Expose service on a domain like `myapp.com` via AWS/GCP

## Kubernetes Namespaces
- [Refer Here](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/) for the Official docs.
- A **namespace** in Kubernetes is like a **virtual cluster** inside your real cluster.
- It is used to **divide cluster resources** among multiple users/projects/teams.
- All objects like Pods, Services, Deployments are created **inside a namespace**.
- Useful when running **multiple teams or environments** (e.g., dev, prod) on the **same cluster**.
- If you don’t specify a namespace, objects are created in the `default` namespace.
- **Default Namespaces in Kubernetes:**
  - `default` ➝ Your playground, used if no namespace is specified
  - `kube-system` ➝ For Kubernetes system components (like DNS, networking)
  - `kube-public` ➝ Public resources, viewable by all users
- **Use of Custom Namespaces:**
  - Separate environments: **dev**, **test**, **prod**
  - Avoid accidentally modifying prod while working on dev
  - Apply **policies** and **resource limits** per namespace
- **Communication Between Namespaces:**
  - Pods **within the same namespace**: use service name directly `service-name`
  - Pods **across namespaces**: use full DNS `service-name.namespace-name.svc.cluster.local`
  ![preview](./Images/Kubernetes_CKA11.png)
### Namespace YAML Configuration
- [Refer Here](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.32/) for `One-page API Reference for Kubernetes` and choose required Version.
  - Select required API, in this case `Namespace`.
- [Refer Here](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.32/#namespace-v1-core) for the **Namespace Cluster APIs**.
  - `apiVersion` → Group/Version
    - If **Group** is `core`, provide only `Version`
    - If **Group** is other than `core`, provide `Group/Version`
  - `kind` → Kind
  - `metadata` → Use Field and Description.
- **Namespace YAML:**
  - Based on `Namespace Cluster APIs`, Write the Namespace YAML file:
    ```yaml
    apiVersion: v1
    kind: Namespace
    metadata:
      name: dev
    ```
- **Commands:**
  ```sh
  kubectl apply -f <filename>
  # Create namespace
  kubectl get namespaces
  # List all namespaces
  kubectl get pods
  # List pods in default namespace (ns)
  kubectl get pods -n <namespace-name>
  # List pods in a specific ns
  kubectl get pods --all-namespaces
  # List pods in all namespaces
  kubectl config set-context $(kubectl config current-context) --namespace=<namespace-name>
                            (or)
  kubectl config set-context --current --namespace=<namespace-name>
  # Switch current namespace permanently (Set the default namespace to given namespace)
  kubectl config view --minify | grep namespace
  # Check which namespace you’re in
  kubectl create namespace <namespace-name>
  # Create a namespace
  ```
### Create Resources in a Specific Namespace
- **Option:1** From Command-Line
  ```bash
  kubectl create -f yaml-filename -n <namespace-name>
  ```
- **Option:2** In YAML
  - Add under `metadata`:
    ```yaml
    metadata:
      name: resource-name
      namespace: namespace-name
    ```
- **Note:** If you don’t specify a namespace, objects are created in the `default` namespace.
### Resource Quotas in Namespace
- [Refer Here](https://kubernetes.io/docs/concepts/policy/resource-quotas/) for the Official docs.
- You can **limit usage of resources** (CPU, memory, pod count, etc.) using `ResourceQuota`.
#### ResourceQuota YAML Configuration
- [Refer Here](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.32/) for `One-page API Reference for Kubernetes` and choose required Version.
  - Select required API, in this case `ResourceQuota`.
- [Refer Here](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.32/#resourcequota-v1-core) for the **ResourceQuota Cluster APIs**.
  - `apiVersion` → Group/Version
    - If **Group** is `core`, provide only `Version`
    - If **Group** is other than `core`, provide `Group/Version`
  - `kind` → Kind
  - `metadata` & `spec` → Use Field and Description.
- **`apiVersion`, `kind`, `metadata` & `spec`** these are must have Root Keys in every Kubernetes Manifest YAML files.
- **ResourceQuota `dev` Namespace YAML:**
  - Based on `ResourceQuota Cluster APIs`, Write the ResourceQuota YAML file:
    ```yaml
    apiVersion: v1
    kind: ResourceQuota
    metadata:
      name: dev-quota
      namespace: dev
    spec:
      hard:
        pods: "10"
        requests.cpu: "10"
        requests.memory: "10Gi"
    ```
- **Commands:**
  ```bash
  kubectl apply -f <filename>
  # Create resourcequota
  kubectl get resourcequota -n <namespace-name>
  # View resourcequota in a namespace
  kubectl describe resourcequota <resourcequota-name> -n <namespace-name>
  # Describe a resourcequota
  kubectl delete resourcequota <resourcequota-name> -n <namespace-name>
  # Delete a resourcequota
  ```

## Kubernetes - Imperative vs Declarative Approaches
- **Imperative:**
  - Direct commands to create/update/delete resources
  - **Examples:** `kubectl run`, `kubectl create`, `kubectl edit`, `kubectl delete`, `kubectl set image`
- **Declarative:**
  - Use YAML files that define the desired state, and apply them
  - **Command:** `kubectl apply -f <yaml-filename>`
### Key Points
- **Imperative Approach:**
  - Quick and handy in exams.
  - Good for small/simple changes.
  - Can generate YAML templates with `--dry-run=client -o yaml`.
  - Not ideal for teamwork or tracking changes.
  - Example:
    ```bash
    kubectl run nginx --image=nginx
    kubectl edit pod nginx
    kubectl scale deployment nginx --replicas=4
    ```
- **Declarative Approach:**
  - Ideal for production.
  - Use YAML files to describe resources.
  - Trackable (can be stored in Git).
  - Use `kubectl apply` to create or update.
  - Kubernetes compares current state with desired state and makes changes.
### Certification Tips
- Use **imperative commands** for speed (e.g. creating a pod/deployment quickly).
- Use **kubectl edit** for fast one-time edits.
- Use **kubectl apply** with YAML files for complex tasks.
- Prefer YAML when:
  - Using multiple containers
  - Adding env vars, init containers, etc.
  - You want version control (Git)
- Generate YAMLs from imperative commands:
  - `--dry-run=client -o yaml`: **Generate YAML** without creating resources.
    ```bash
    kubectl run <pod-name> --image=<image-name> --dry-run=client -o yaml
    ```
### Imperative Commands
- Pods
  ```bash
  kubectl run <pod-name> --image=<image-name>
  # Create a Pod
  kubectl run <pod-name> --image=<image-name> --dry-run=client -o yaml
  # Generate pod manifest yaml file (-o yaml), don't create it(--dry-run)
  ```
- **Deployments:**
  ```bash
  kubectl create deployment <deployment-name> --image=<image-name>
  # Create a deployment
  kubectl create deployment <deployment-name> --image=<image-name> --replicas=4 --dry-run=client -o yaml
  # Generate deployment manifest yaml file (-o yaml), don't create it(--dry-run)
  kubectl create deployment <deployment-name> --image=<image-name> --replicas=4 --dry-run=client -o yaml > <filename>.yaml
  # Generate deployment manifest yaml file (-o yaml), don't create it(--dry-run) and save it to a file
  kubectl scale deployment <deployment-name> --replicas=4
  # To scale deployment
  ```
- **Services:**
  ```bash
  kubectl expose pod <pod-name> --port=<pod-port> --name=<service-name> --dry-run=client -o yaml
  # Create a service of type `ClusterIP`
  kubectl expose pod <pod-name> --type=NodePort --port=80 --name=<service-name> --dry-run=client -o yaml
  # Create a service of type `NodePort` to expose pod port `80` on node port
  ```

## kubectl apply
- **Creates the resource if it does not exist**, but **updates it if it already exists**.
- It **works declaratively**, meaning Kubernetes will only update the parts that have changed in the YAML.
- Recommended for managing resources efficiently in a production environment.
- **Example Scenario:**
  - Initial Deployment:
    ```sh
    kubectl create -f <filename>
    ```
  - Modify the YAML file (e.g., change the container image)
  - Apply the changes:
    ```sh
    kubectl apply -f <filename>
    ```
- Tracks changes across three versions of the configuration:
  1. **Local Configuration** – the YAML file on your system.
  2. **Live Configuration** – the current state of the object in the cluster.
  3. **Last Applied Configuration** – stored as an annotation (`kubectl.kubernetes.io/last-applied-configuration`) in the live object by `kubectl apply`.
- **`kubectl apply` Updates Work:**
  - When you run `kubectl apply` again:
    - **Local YAML** is compared with:
      - The **live object**
      - The **last applied configuration**
    - Any differences are used to **update the live object**.
    - The **last applied config is also updated** after the change.
- **Deleting a Field in YAML:**
  - If a field (like a label) is **deleted** from the local YAML:
    - `kubectl apply` checks:
      - If the field was present in the **last applied config**
      - And is **missing in the new local config**
    - Then it **removes** it from the **live configuration**.
  - If a field is in the live object but **not in local or last applied**, it is **left unchanged**.
- **Note:**
  - `last-applied-configuration` is **only stored** when you use `kubectl apply`.
  - **Other commands like `kubectl create` or `replace` do not save** this annotation.
  - Avoid mixing **imperative** (`kubectl create`, `kubectl delete`) and **declarative** (`kubectl apply`) approaches to manage the same object.


# <p align="center">Scheduling</p>

## Kubernetes Manual Scheduling
- Normally, the **Kubernetes Scheduler** automatically decides **which node** a pod should run on.
- **Manual Scheduling** means **you choose** which node a pod runs on **without using the scheduler**.
### Manual Scheduling Works
- Every pod has an internal field called **`nodeName`**.
- The scheduler automatically fills this field when assigning a pod to a node.
- In manual scheduling, **you** set this `nodeName` directly in the pod definition.
#### Manually Assign a Pod to a Node
- Based on `Pod Workloads APIs`, Write the Pod YAML file with `nodeName` field:
  ```yaml
  apiVersion: v1
  kind: Pod
  metadata:
    name: nginx
    labels:
      name: nginx
  spec:
    containers:
    - name: nginx
      image: nginx
      ports:
      - containerPort: 8080
    nodeName: node02    # Manually specify node name
  ```
- This pod will be scheduled directly on `node02`.
### No Scheduler Available
- If the scheduler is **disabled or unavailable**, you can still **bind the pod manually** using a **Binding object**.
#### Using Binding API
- First create the pod **without nodeName** using `Pod Workloads APIs`:
  ```yaml
  apiVersion: v1
  kind: Pod
  metadata:
    name: nginx
    labels:
      name: nginx
  spec:
    containers:
    - name: nginx
      image: nginx
      ports:
      - containerPort: 8080
  ```
- Check the Pod status, it is in `Pending` state because there is no `Scheduler` present.
  ![preview](./Images/Kubernetes_CKA12.png)
- **Then create a `Binding` object:**
  - [Refer Here](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.32/) for `One-page API Reference for Kubernetes` and choose required Version.
    - Select required API, in this case `Binding`.
  - [Refer Here](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.32/#binding-v1-core) for the **Binding Cluster APIs**.
    - `apiVersion` → Group/Version
      1. If **Group** is `core`, provide only `Version`
      2. If **Group** is other than `core`, provide `Group/Version`
    - `kind` → Kind
    - `metadata` & `target` → Use Field and Description.
  - Based on `Binding Cluster APIs`, Write the Binding YAML file:
    ```yaml
    apiVersion: v1
    kind: Binding
    metadata:
      name: nginx
    target:
      apiVersion: v1
      kind: Node
      name: node01
    ```
  - This binding will assign the pod named `nginx` to `node01`.
  ![preview](./Images/Kubernetes_CKA14.png)
  ![preview](./Images/Kubernetes_CKA13.png)
### Important Points
- Manual scheduling is not recommended for production but useful for learning and testing.
- If you use `nodeName`, the **scheduler skips** the pod entirely.
- Make sure the node exists and is **Ready**, otherwise the pod will stay in **Pending** state.
- We **cann't move a running Pod from one Node to another Node**. We **need to delete Pod** from one Node and **recreate** it from another Node.
  ```sh
  kubectl replace --force -f <yaml-filename>
  ```

## Kubernetes Labels, Selectors & Annotations
### Labels and Selectors
- [Refer Here](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/) for the Official docs.
- **Labels:**
  - **Labels** are **key-value pairs** attached to Kubernetes objects.
  - Used to **group** and **identify** objects (pods, deployments, services, etc.)
  - **Example:**
    ```yaml
    metadata:
      labels:
        app: my-app
        env: prod
    ```
- **Selectors:**
  - **Selectors** are used to **filter** and **select** objects based their labels.
  - **Example:**
    ```bash
    kubectl get pods --selector <label-key>=<label-value>
    kubectl get pods --selector <label-key>=<label-value>,<label-key>=<label-value>,<label-key>=<label-value>
    ```
#### Use Case Examples
- **ReplicaSet:**
  - Selects Pods using labels.
  - In `metadata`, we have field `labels`, and in `spec`, we have field `selector`.
    ```yaml
    apiVersion: apps/v1
    kind: ReplicaSet
    metadata:
      name: simple-webapp
      labels:
        app: my-app
        function: Front-end
    spec:
      replicas: 3
      selector:
        matchLabels:
          app: my-app
      template:
        metadata:
          labels:
            app: my-app # Important: this must match the selector
            function: Front-end
        spec:
          containers:
          - name: simple-webapp
            image: simple-webapp
    ```
  - **Common Mistake**: Confusing ReplicaSet's own labels with Pod labels under `template`.
- **Service:**
  - Uses selectors to send traffic to the right Pods.
  - In `spec`, we have field `selector`.
    ```yaml
    apiVersion: v1
    kind: Service
    metadata:
      name: my-service
    spec:
      selector:
        app: App1
      ports:
      - protocol: TCP
        port: 80
        targetPort: 9376
    ```
### Annotations
- [Refer Here](https://kubernetes.io/docs/concepts/overview/working-with-objects/annotations/) for the Official docs.
- Used to **store metadata/info**, that’s not used to select or group.
- Example Uses: version info, contact info, build ID, etc.
- **Labels and selectors** are used to `group` and `select` objects, while **Annotations** are used to `record other details for informational purposes`.
- **Example:**
  - In `metadata`, we have field `annotations`.
    ```yaml
    metadata:
      annotations:
        build-version: "v1.2.3"
        contact: "admin@example.com"
    ```

## Taints and Tolerations
- [Refer Here](https://kubernetes.io/docs/concepts/scheduling-eviction/taint-and-toleration/) for the Official docs.
- **Taints and Tolerations** are used to **control which Pods can be scheduled on which Nodes**.
- You **taint a node** to **repel pods**, and **pods can tolerate** taints using tolerations.
- **Taints are applied to Nodes**, **Tolerations are applied to Pods YAML**.
- Only pods that have a **matching toleration** can land on (be scheduled to) that node. Pods without toleration **cannot be scheduled** on tainted nodes.
### Taint
- Use `kubectl taint nodes` command to taint a node.
- Syntax:
  ```bash
  kubectl taint nodes <node-name> <key>=<value>:<taint-effect>
  ```
  - There are three taint effects:
    - **`NoSchedule:`** Pod **will not be scheduled** unless it has a matching toleration
    - **`PreferNoSchedule:`** Try to avoid scheduling if not tolerated (soft restriction)
    - **`NoExecute:`** Pod **will be evicted** from the node if it doesn’t tolerate this taint
- **Example:**
  ```bash
  kubectl taint nodes node1 app=blue:NoSchedule
  ```
  - This means: node1 won’t accept any pod **unless** it has a **toleration** for `app=blue`.
- **Commands:**
  ```sh
  kubectl describe node <node-name>
  # To view taints on a node
  kubectl taint nodes <node-name> <key>=<value>:<taint-effect>-
  # To remove a taint (The '-' at the end removes the taint.)
  ```
### Pod Toleration
- Tolerations are added to pods by adding a `tolerations` section in pod definition.
- In `spec`, we have field `tolerations`.
  ```yaml
  apiVersion: v1
  kind: Pod
  metadata:
    name: myapp-pod
  spec:
    containers:
    - name: nginx-container
      image: nginx
    tolerations:
    - key: "app"
      operator: "Equal"
      value: "blue"
      effect: "NoSchedule"
  ```
- This pod is now **allowed** to run on a node with the taint `app=blue:NoSchedule`.
### Important Notes
- Taints **don’t tell pod where to go**, they tell **node who is allowed**.
- To force pod to go to specific node, use **Node Affinity**.
- **Master Node Taint:**
  - By default, **control-plane node is tainted**: **`node-role.kubernetes.io/control-plane:NoSchedule`**
  - So, **no pods are scheduled on it** by default.

## Node Selectors
- [Refer Here](https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/#nodeselector) for the Official docs.
- `nodeSelector` is a **basic method to schedule a Pod on a specific node** based on node labels.
- You **label a node**, then use `nodeSelector` in your pod spec to match that label.
### Example Workflow
- **Label a Node:**
  - Syntax:
    ```bash
    kubectl label nodes <node-name> <key>=<value>
    ```
  - **Example:**
    ```bash
    kubectl label nodes node-1 size=Large
    ```
    - This adds the label `size=Large` to `node-1`.
- **Pod with Node Selector:**
  - Based on `Pod Workloads APIs`, Write the Pod YAML file with `nodeSelector` field.
  - In `spec`, we have field `nodeSelector`.
    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: myapp-pod
    spec:
      containers:
      - name: data-processor
        image: data-processor
      nodeSelector:
        size: Large
    ```
    - This pod will only be scheduled on nodes with label `size=Large`.
- **Create the Pod:**
  ```bash
  kubectl apply -f pod-definition.yaml
  ```
### Limitations of Node Selector
- `nodeSelector` only supports **exact match** with **a single label**.
- Not suitable for complex scheduling needs (like OR, IN, NOT IN, etc.).

## Node Affinity
- [Refer Here](https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/#node-affinity) for the Official docs.
- Node Affinity is an **advanced version of `nodeSelector`**.
- It lets you schedule pods **on specific nodes based on labels**, using **expressions**.
- More powerful and flexible than `nodeSelector`.
### Pod with Node Affinity
- Based on `Pod Workloads APIs`, Write the Pod YAML file with `nodeAffinity` field.
- In `spec`, we have field `nodeAffinity`.
  ```yaml
  apiVersion: v1
  kind: Pod
  metadata:
    name: myapp-pod
  spec:
    containers:
      - name: data-processor
        image: data-processor
    affinity:
      nodeAffinity:
        requiredDuringSchedulingIgnoredDuringExecution:
          nodeSelectorTerms:
            - matchExpressions:
                - key: size
                  operator: In
                  values:
                    - Large
                    - Medium
  ```
- Pod can run **only** on nodes labeled with `size=Large` or `size=Medium`.
### Match Expressions:
- `key`: the label key (e.g. `size`)
- `operator`: how to compare
  - `In`: value is in list (e.g. `large`, `medium`)
  - `NotIn`: value is **not** in list
  - `Exists`: key must **exist**
  - `DoesNotExist`: key must **not exist**
### Types of Node Affinity:
- **`requiredDuringSchedulingIgnoredDuringExecution`**:
  - Required at pod creation, Ignored after pod runs (Pod **won’t be scheduled** if no matching node found)
- **`preferredDuringSchedulingIgnoredDuringExecution`**:
  - Try best to match, Ignored after pod runs (Pod is placed on best-matching node, but **can go anywhere** if needed)
- **`requiredDuringSchedulingRequiredDuringExecution`**:
  - This will **evict** the pod if node no longer meets affinity rules (not available yet, Future type)
### Taints & Tolerations vs Node Affinity
- **Taints & Tolerations:**
  - **Prevent** pods from scheduling on nodes
  - **Node controls** which pods are allowed
  - Blocks pods **unless they tolerate taint**
  - Avoid certain nodes (e.g., GPU-only)
- **Node Affinity:**
  - **Prefer or require** pods to go to specific nodes
  - **Pod chooses** preferred node using labels
  - Pod tries to find **matching node label**
  - Target specific nodes (e.g., SSD-only)

## Resource Requests and Limits
- [Refer Here](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/) for the Official docs.
- Kubernetes lets you define **how much CPU and memory** (RAM) a container **requests** and the **maximum it can use**.
  - **Request:** Minimum resources the container is guaranteed to get
  - **Limit:** Maximum resources the container is allowed to use
- **Default Behavior:**
  - If you **don’t define** resource requests, Kubernetes assumes:
    - `CPU = 0.5`
    - `Memory = 256Mi`
  - Pod might go into **Pending** state if no node has enough resources.
### Pod with Resource Request
- Based on `Pod Workloads APIs`, Write the Pod YAML file with `resources` field.
- In `spec`, we have field `resources`.
  ```yaml
  apiVersion: v1
  kind: Pod
  metadata:
    name: simple-webapp
  spec:
    containers:
    - name: app
      image: simple-webapp
      resources:
        requests:
          memory: "1Gi"
          cpu: "1"
  ```
- This pod **requires 1 CPU and 1Gi memory** to be scheduled.
### Pod with Resource Limits
- Based on `Pod Workloads APIs`, Write the Pod YAML file with `resources` field.
- In `spec`, we have field `resources`.
  ```yaml
  apiVersion: v1
  kind: Pod
  metadata:
    name: simple-webapp
  spec:
    containers:
    - name: app
      image: simple-webapp
      resources:
        requests:
          memory: "1Gi"
          cpu: "1"
        limits:
          memory: "2Gi"
          cpu: "2"
  ```
- This pod **won’t be allowed to use more than 2 CPU or 2Gi memory**.
### Exceeding Limits
- If the container tries to use **more than its limits**:
  - **CPU**: throttled (slowed down)
  - **Memory**: container is **killed and restarted** (OOM error = Out Of Memory)
### Scenarios
- **No requests, no limits:** Pod can consume everything, bad for others
- **No requests, but limits set:** Request = limit by default
- **Requests + limits set:** Guaranteed request, capped by limit
- **Requests set, limits not set:** Best option - guaranteed minimum, use extra if available
### Notes
- Requests and limits are defined **per container**, not per pod.
- Use them to **protect nodes** from being overloaded.
- Helps Kubernetes **schedule pods more efficiently**.

## Editing Pods & Deployments
### Editing Pods (Manually Created Pods)
- You **cannot** change most parts of a running pod.
  - **You can only edit:**
    - `spec.containers[*].image`  
    - `spec.initContainers[*].image`  
    - `spec.activeDeadlineSeconds`  
    - `spec.tolerations`  
  - **Cannot edit:**
    - Environment variables, service accounts, resource limits, etc.
- There are two ways to **Edit** a Pod Properly:
  1. **`kubectl edit`**
  2. **`Export → Edit → Recreate`**
#### Method:1 `kubectl edit` (Quick but Not Effective Directly)
- Edit the Pod:
  ```bash
  kubectl edit pod <pod-name>
  ```
  - Opens pod YAML in an editor (usually `vi`).
  - If you edit uneditable fields → **save will be denied**.
  - But your changes are saved temporarily, e.g., `/tmp/kubectl-edit-xxxx.yaml`.
- **To apply changes:**
  ```bash
  kubectl delete pod <pod-name>
  # Delete pod
  kubectl create -f /tmp/kubectl-edit-xxxx.yaml
  # Create pod with temporarily saved file.
  ```
#### Method:2 `Export → Edit → Recreate` (Recommended)
- Export the Pod YAML:
  ```bash
  kubectl get pod <pod-name> -o yaml > my-new-pod.yaml
  ```
- Edit and Make Changes: `vi my-new-pod.yaml`
- Delete the Pod and Recreate with the above file:
  ```sh
  kubectl delete pod <pod-name>
  kubectl create -f my-new-pod.yaml
  ```
### Editing Deployments
- Deployments manage pods using a **template**.
- You **can edit any field** in the pod template, and the deployment will automatically:
  - Delete old pods
  - Create new pods with updated values
- Command:
  ```bash
  kubectl edit deployment <deployment-name>
  ```
- No need to delete/recreate anything manually.
- Works well for changing environment variables, resource limits, image versions, etc.

## Kubernetes DaemonSets
- [Refer Here](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/) for the Official docs.
- A **DaemonSet** ensures **one pod is running on every node** (or selected nodes) in the cluster.
- It runs a **copy of the pod on every node**.
- If a **new node is added**, it creates a pod there automatically. If a **node is removed**, the pod is removed too.
- Similar to ReplicaSet, but **runs exactly one copy per node**.
- **Use Cases of DaemonSet:**
  - Monitoring agents (e.g., Prometheus node exporter)
  - Log collectors (e.g., Fluentd, Filebeat)
  - Kubernetes system components (e.g., `kube-proxy`)
  - Networking tools (e.g., CNI plugins like Calico or Weave Net)
  ![preview](./Images/Kubernetes_CKA15.png)
### DaemonSet YAML Configuration
- [Refer Here](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.32/) for `One-page API Reference for Kubernetes` and choose required Version.
  - Select required API, in this case `DaemonSet`.
- [Refer Here](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.32/#daemonset-v1-apps) for the **DaemonSet Workloads APIs**.
  - `apiVersion` → Group/Version
    - If **Group** is `core`, provide only `Version`
    - If **Group** is other than `core`, provide `Group/Version`
  - `kind` → Kind
  - `metadata` & `spec` → Use Field and Description.
- **`apiVersion`, `kind`, `metadata` & `spec`** these are must have Root Keys in every Kubernetes Manifest YAML files.
- **DaemonSet YAML:**
  - Based on `DaemonSet Workloads APIs`, Write the Deployment YAML file.
  - Almost identical to a ReplicaSet, but with:
    - `kind: DaemonSet`
    - Uses `selector` and `template` for pod spec
    ```yaml
    apiVersion: apps/v1
    kind: DaemonSet
    metadata:
      name: monitoring-daemon
    spec:
      selector:
        matchLabels:
          name: monitoring-agent
      template:
        metadata:
          labels:
            name: monitoring-agent
        spec:
          containers:
          - name: monitoring-agent
            image: some-monitoring-image
    ```
- **Commands:**
  ```sh
  kubectl create -f <filename>
  # Create DaemonSet
  kubectl get daemonset
  # List DaemonSets
  kubectl describe daemonset <name>
  # Get details of a DaemonSet
  ```
### DaemonSet Schedules Pods
- **Before Kubernetes v1.12**: Manually set `nodeName` in pod spec.
- **From v1.12 onwards**: Uses the **default scheduler** and **node affinity rules**.

## Kubernetes Static Pods
- [Refer Here](https://kubernetes.io/docs/tasks/configure-pod-container/static-pod/) for the Official docs.
- **Static Pods** are **created directly by the kubelet**, not by the kube-apiserver.
- They are **not managed by the control plane** (no scheduler, no controllers, no etcd).
- Kubelet watches a folder on the node for pod YAML files and manages pods from there.
- You can **differentiate a static pod** from other pods by looking at **its name — it usually ends with the node name** where it’s running.
  ![preview](./Images/Kubernetes_CKA16.png)
### Working of Static Pods
- Kubelet checks a folder (e.g. `/etc/kubernetes/manifests`) for pod definition files.
- When a YAML file is placed there:
  - Kubelet creates the pod.
  - Kubelet **restarts** the pod if it crashes.
  - **Updates** to the file will recreate the pod.
  - **Deleting** the file will delete the pod.
### Configuring Static Pod Path
- You can specify the static pod path in **two ways**:
  1. **In kubelet service file** (systemd):
      - **Kubelet watches a directory** on the host machine for pod definition files.
      - This directory is specified using the `--pod-manifest-path` option when starting the kubelet.
        ```
        --pod-manifest-path=/etc/kubernetes/manifests
        ```
  2. **In a kubelet config file** (used by kubeadm):
      - Instead of passing the path directly in the systemd `kubelet.service` file, you can use a kubelet config file.
        ```yaml
        # kubelet-config.yaml
        staticPodPath: /etc/kubernetes/manifests
        ```
      - And reference this file:
        ```arduino
        --config=/var/lib/kubelet/config.yaml
        ```
- Check the kubelet config with:
  ```bash
  ps -ef | grep kubelet
  ```
### View Static Pods
- Static pods are containers run by Docker (or container runtime), not listed by `kubectl` unless there's an API server.
- Use:
  ```bash
  docker ps
  # For docker
  crictl ps
  # For containerd
  ```
- If part of a cluster, static pods will appear in `kubectl get pods`, **but**:
  - They are shown as **mirror pods** (read-only, can't delete via kubectl).
  - You must **edit or remove the file** in the manifest directory to manage them.
### Limitations and Use Cases of Static Pods
- **Limitations:**
  - Only supports **Pods**.
  - No support for Deployments, ReplicaSets, Services, etc.
  - These features require the full Kubernetes control plane.
- **Use Cases:**
  - Useful for **bootstrapping the control plane** itself:
    - Place YAML files for `kube-apiserver`, `etcd`, `controller-manager`, etc., into `/etc/kubernetes/manifests`
    - Kubelet runs them as static pods.
  - This is how **kubeadm** sets up the control plane.
### Static Pods vs DaemonSets
- **Static Pods:**
  - Managed by **kubelet** only
  - Ignored by scheduler
  - Control plane not needed
  - **Use case:** Bootstrap control plane, isolated pods
- **DaemonSets:**
  - Managed by **DaemonSet controller** via kube-apiserver
  - Ignored by scheduler
  - Control plane needed
  - **Use case:** Deploying agents/loggers on all nodes

## Multiple Schedulers in Kubernetes
- [Refer Here](https://kubernetes.io/docs/tasks/extend-kubernetes/configure-multiple-schedulers/) for the Official docs.
- **Scheduler:**
  - The **scheduler** decides **which node** a pod runs on.
  - The **default scheduler** (`kube-scheduler`) balances pods across nodes and respects:
    - Taints & Tolerations
    - Node Affinity/Anti-Affinity
    - Resource Requests & Limits
- **Multiple Schedulers:**
  - You may need **custom logic** for placing pods (e.g., based on special checks).
  - Kubernetes lets you **run multiple schedulers** in the same cluster.
    - One for general workloads (default).
    - Another (custom) for specific apps.
### Working of Multiple Schedulers
- **Default Scheduler**:
   - Named: `default-scheduler` (implicit name if not specified)
   - Configured via `kube-scheduler` config file
- **Custom Scheduler**:
   1. Create a **new config file** with a unique `schedulerName`
   2. Deploying a Custom Scheduler (Pod or Deployment)
      - Use the same `kube-scheduler` binary (or your own built one).
      - Deploy with:
        - `--config` pointing to your scheduler config
        - `--kubeconfig` for API access
      - Mount config via:
        - **Volume** or
        - **ConfigMap** (preferred)
   3. Auth Requirements:
      - Needs:
        - `ServiceAccount`
        - `ClusterRole` and `ClusterRoleBinding`
      - **Note:** Covered in Authentication section of the course

### Create a Custom Scheduler in Kubernetes
- **Create a Custom Scheduler Config File:**
  - This file gives your scheduler a unique name.
  - [Refer Here](https://kubernetes.io/docs/reference/config-api/kube-scheduler-config.v1/#kubescheduler-config-k8s-io-v1-KubeSchedulerConfiguration) for the Official docs.
    - Write YAML file based on `Fields`.
  - Create a file called `my-scheduler-config.yaml`:
    ```yaml
    apiVersion: kubescheduler.config.k8s.io/v1
    kind: KubeSchedulerConfiguration
    leaderElection:
      leaderElect: false  # Set false because we are running only one replica
    schedulerName: my-custom-scheduler
    ```
- **Create a ConfigMap from this File:**
  - This makes the config file available in the cluster to mount into a pod.
    ```bash
    kubectl create configmap my-scheduler-config \
      --from-file=my-scheduler-config.yaml \
      -n kube-system
    ```
- **Deploy the Custom Scheduler as a Deployment:**
  - Create a file `my-scheduler-deployment.yaml` based on `Deployment Workloads APIs`:
    ```yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: my-custom-scheduler
      namespace: kube-system
    spec:
      replicas: 1
      selector:
        matchLabels:
          app: my-custom-scheduler
      template:
        metadata:
          labels:
            app: my-custom-scheduler
        spec:
          containers:
          - name: my-custom-scheduler
            image: k8s.gcr.io/kube-scheduler:v1.28.0  # Change version as needed
            command:
              - kube-scheduler
              - --config=/etc/kubernetes/my-scheduler-config.yaml
            volumeMounts:
              - name: config
                mountPath: /etc/kubernetes
          volumes:
            - name: config
              configMap:
                name: my-scheduler-config
    ```
  - Apply the deployment:
    ```bash
    kubectl apply -f my-scheduler-deployment.yaml
    ```
- **Check if It’s Running:**
  ```bash
  kubectl get pods -n kube-system | grep my-custom-scheduler
  # You should see your scheduler pod running.
  ```
- **Test with a Pod That Uses Custom Scheduler:**
  - Create a pod using your custom scheduler based on `Pod Workloads APIs`:
    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: test-nginx
    spec:
      schedulerName: my-custom-scheduler  # <-- important!
      containers:
        - name: nginx
          image: nginx
    ```
  - Save this as `nginx-custom.yaml` and apply:
    ```bash
    kubectl apply -f nginx-custom.yaml
    ```
- **Verify the Pod Was Scheduled by Your Scheduler:**
  ```bash
  kubectl get events -o wide
  ```
  - Check if the **source** of the "Scheduled" event is `my-custom-scheduler`.
- **Troubleshooting:**
  - Pod stays **Pending** ➜ Scheduler may not be working/configured.
  - Check with:
    ```bash
    kubectl describe pod <pod-name>
    kubectl get events -o wide
    kubectl logs <scheduler-pod-name> -n kube-system
    ```

## Kubernetes Scheduler Configuration & Scheduler Profiles
### Scheduling Process
1. **Pod Created** → Added to Scheduling Queue.
2. **Priority Sort Phase**: Pods sorted by `PriorityClass`.
3. **Filter Phase**: Remove nodes that don't meet requirements.
   - Plugins:
     - `NodeResourcesFit` → Filters nodes without enough CPU/memory.
     - `NodeName` → Filters nodes not matching `nodeName` in Pod spec.
     - `NodeUnschedulable` → Filters cordoned (unschedulable) nodes.
4. **Score Phase**: Score remaining nodes.
   - Plugins:
     - `NodeResourcesFit` → Scores based on leftover CPU/memory.
     - `ImageLocality` → Prefers nodes with the pod’s image.
5. **Bind Phase**: Pod is bound to the highest scored node using `DefaultBinder`.
### Scheduler Plugins & Extension Points
- Kubernetes scheduling uses **plugins** plugged into **extension points** at different phases.
- Common extension points:
  - `QueueSort`, `PreFilter`, `Filter`, `PostFilter`
  - `PreScore`, `Score`, `Reserve`, `Permit`
  - `PreBind`, `Bind`, `PostBind`
- You can plug in **custom plugins** at any of these stages!
### Custom Scheduler Options
- **Option 1: Old Method**
  - Run **multiple scheduler binaries** (`kube-scheduler`, `my-scheduler`, etc.).
  - Issues: Hard to manage, potential race conditions (multiple schedulers might pick the same node).
- **Option 2: New Method (K8s v1.18+)**
  - Use **Scheduler Profiles** in a single scheduler config.
  - Allows multiple virtual schedulers by `schedulerName` (Each profile = a virtual scheduler with a unique `schedulerName`).
### Example Scheduler Profile Configuration
- [Refer Here](https://kubernetes.io/docs/reference/config-api/kube-scheduler-config.v1/#kubescheduler-config-k8s-io-v1-KubeSchedulerConfiguration) for the Official docs.
    - Write YAML file based on `Fields`.
- Create a file called `scheduler-config.yaml`:
  ```yaml
  apiVersion: kubescheduler.config.k8s.io/v1
  kind: KubeSchedulerConfiguration
  clientConnection:
    kubeconfig: "/etc/kubernetes/scheduler.conf"
  leaderElection:
    leaderElect: true
  profiles:
  - schedulerName: default-scheduler
    plugins:
      score:
        disabled:
          - name: NodeResourcesBalancedAllocation
        enabled:
          - name: NodeResourcesLeastAllocated
  ```
  - In this config, we change the **scoring algorithm** by disabling one plugin and enabling another.
### Using Custom Config with Kubeadm Clusters
- Save your config as `scheduler-config.yaml`.
- Scheduler runs as a static pod (`kube-system` namespace).
- Modify `/etc/kubernetes/manifests/kube-scheduler.yaml`:
  ```yaml
  command:
  - kube-scheduler
  - --leader-elect=true
  - --config=/etc/kubernetes/scheduler-config.yaml
  ```
- Saving the file, the Static Pod will auto-restarts the scheduler with the new config.
### Scheduler High-Level Flow
1. **Watch** the API server for unscheduled pods.
2. **Filter** nodes that can run the pod.
3. **Score** the filtered nodes.
4. **Select** the best node.
5. **Bind** the pod to that node.

## Admission Controllers in Kubernetes
- Components in the **API server pipeline** that **intercept requests** after:
  1. **Authentication** – Validates who the user is (via certs).
  2. **Authorization** – Checks if user **has permission** (via RBAC).
- Admission controllers **validate, modify, or reject requests** before they are stored in etcd.
### Use of Admission Controllers
- RBAC handles **who can access what**, but **not how resources should be used**.
- Admission controllers help **enforce cluster usage rules** like:
  - Only allow images from internal registries.
  - Reject `latest` image tags.
  - Disallow containers running as root.
  - Require specific labels in metadata.
### Role in Request Flow
1. **kubectl request** → User sends a request (e.g., create pod)
2. API Server:
   - **Authentication** → API server authenticates the request
   - **Authorization** → API server authorizes using RBAC
   - **Admission Controllers** → Admission Controllers validate/modify/reject request
   - Object stored in **etcd** if all checks pass.
### Examples of Built-in Admission Controllers
- `NamespaceExists` → Rejects requests to **non-existent namespaces**.
- `AlwaysPullImages` → Forces image to be pulled every time a pod is created.
- `DefaultStorageClass` → Assigns default storage class if none specified.
- `EventRateLimit` → Limits rate of requests to API server.
- `NamespaceLifecycle` → Replaces `NamespaceExists` & `NamespaceAutoProvision`. Prevents use/deletion of system namespaces like `default`, `kube-system`.
### Enabling / Disabling Admission Controllers
- View current enabled plugins:
  ```bash
  kube-apiserver -h | grep enable-admission-plugins
  ```
- **In kubeadm-based setups**, the API server runs as a static pod:
  - Modify:
    ```bash
    /etc/kubernetes/manifests/kube-apiserver.yaml
    ```
  - Add or modify:
    ```yaml
    --enable-admission-plugins=...,NamespaceAutoProvision,...
    --disable-admission-plugins=...
    ```
  - Check the process to see enabled and disabled plugins:
    ```sh
    ps -ef | grep kube-apiserver | grep admission-plugins
    ```
### Example: Namespace Auto Creation
1. Without `NamespaceAutoProvision`:
   - Creating a pod in a non-existent namespace fails.
2. With `NamespaceAutoProvision` enabled:
   - Namespace is **auto-created**, and the pod is deployed.
3. Note: This is now deprecated and replaced by `NamespaceLifecycle`.

## Admission Controllers (Types & Webhooks)
### Types of Admission Controllers
- Kubernetes Admission Controllers are of **two main types**:
  1. **Validating:**
     - Only checks/validates the request and either **accepts or rejects** it.
     - **Examples:** `NamespaceLifecycle`, `NamespaceExists`
  2. **Mutating:**
     - Can **modify (mutate)** the request before creation.
     - **Examples:** `DefaultStorageClass`, `NamespaceAutoProvision`
- Some can do **both** mutation and validation.
### Work of Admission Controllers
- **Order of Execution:**
  1. Request → **Authentication**
  2. **Authorization (RBAC)**
  3. **Mutating Admission Controllers** (run **first**)
  4. **Validating Admission Controllers** (run **after mutation**)
  5. Object stored in **etcd**
- Mutating comes first so that any changes made can be **validated** after.
- **Example:**
  - `NamespaceAutoProvision` (mutating) creates missing namespace.
  - Then `NamespaceExists` (validating) confirms it exists.
### `DefaultStorageClass` Example (Mutating Controller)
- You create a **PVC without storageClass**.
- `DefaultStorageClass` controller:
  - Detects no storage class set.
  - Automatically adds the default one.
- **Result:** PVC gets created **with** default storage class.
### Custom Admission Controllers using Webhooks
- Want to apply **your own logic** to requests? Use **Admission Webhooks**.
- They are Two Types:
  1. **`MutatingAdmissionWebhook`** (Mutating)
  2. **`ValidatingAdmissionWebhook`** (Validating)
- These call an **external HTTP server** that you create.
#### Work of Admission Webhooks
1. API server sends **AdmissionReview** request (JSON) to your webhook server.
2. Your server:
   - **Reads** the user, operation, object, etc.
   - **Responds** with allowed: `true` or `false`
   - (Mutating webhooks can return a **patch** to modify the object)
#### Setting Up Your Own Webhook (Steps)
- **Step:1 `Develop Webhook Server`**
  - Can be written in Go, Python, etc.
  - Must accept `mutate` and/or `validate` HTTP requests.
  - Responds with JSON.
  - Example mutation:
    - Add username as label to every object created.
- **Step:2 `Deploy Webhook Server`**
  - Option 1: Host **outside cluster** (provide a URL)
  - Option 2: Host **inside cluster** as a **Deployment + Service**
- **Step:3 `Configure Webhook in Kubernetes`**
  - Create either:
    - `MutatingWebhookConfiguration` **or**
    - `ValidatingWebhookConfiguration`
#### Example manifest snippet
- [Refer Here](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.32/) for `One-page API Reference for Kubernetes` and choose required Version.
  - Select required API, in this case `MutatingWebhookConfiguration` or `ValidatingWebhookConfiguration`.
- [Refer Here](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.32/#mutatingwebhookconfiguration-v1-admissionregistration-k8s-io) for the **MutatingWebhookConfiguration Metadata APIs** and [Refer Here](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.32/#validatingwebhookconfiguration-v1-admissionregistration-k8s-io) for the **ValidatingWebhookConfiguration Metadata APIs**.
  - `apiVersion` → Group/Version
    - If **Group** is `core`, provide only `Version`
    - If **Group** is other than `core`, provide `Group/Version`
  - `kind` → Kind
  - `metadata` & `webhooks` → Use Field and Description.
- **Example manifest snippet:**
  ```yaml
  apiVersion: admissionregistration.k8s.io/v1
  kind: ValidatingWebhookConfiguration
  metadata:
    name: pod-policy
  webhooks:
    - name: pod-policy.example.com
      clientConfig:
        service:
          name: webhook-service
          namespace: default
          path: "/validate"
        caBundle: <BASE64_CERT>
      rules:
        - operations: ["CREATE"]
          apiGroups: [""]
          apiVersions: ["v1"]
          resources: ["pods"]
  ```
  - This webhook will be called **only when pods are created**.
#### TLS Requirement
- API server must communicate with webhook **over HTTPS**.
- Your webhook server needs:
  - TLS certificate (Have a **valid certificate and private key**)
  - The certificate must be passed to Kubernetes as a `caBundle`
- Creates a **TLS secret** for `certificate` and `private key`:
  ```bash
  kubectl -n <namespace-name> create secret tls <tls-secret-name> \
      --cert "/root/keys/webhook-server-tls.crt" \
      # Path to certificate
      --key "/root/keys/webhook-server-tls.key"
      # Path to key
  ```
  - This command creates a **TLS secret** in the required namespace, which contains the **certificate and private key** for your **webhook server**.
- This secret provides those **valid certificate and private key** to use **TLS (HTTPS)**.
- You mount this **TLS secret** into the **webhook server pod**, so the server can start with HTTPS enabled.
  - The public certificate (`.crt`) is also **base64-encoded** and added to the `caBundle` in the webhook configuration like this:
    ```yaml
    clientConfig:
      service:
        name: webhook-service
        namespace: webhook-demo
        path: "/validate"
      caBundle: <base64_encoded_crt_here>
    ```