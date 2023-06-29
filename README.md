## Kubernetes

### What is Kubernetes?
Kubernetes is an open-source container orchestration platform that automates the deployment, scaling, and management of containerized applications.

Containers are a lightweight and portable way to package and run applications, along with their dependencies and configurations. Kubernetes provides a framework for managing these containers across a cluster of machines, abstracting away the underlying infrastructure. This allows developers to focus on writing code rather than dealing with the complexities of infrastructure management.

### Need for a Container Orchestration tool
The shift from the monolith to microservices has increased the usage of container technologies because containers provide the ideal hosting environment for small, independent applications like microservices. The rise of containers in microservice technology has led to applications that consist of hundreds, and sometimes even thousands, of containers. Managing such a large number of containers across multiple environments using scripts and self-made tools can be extremely complex and, at times, even impossible. This particular scenario has highlighted the necessity for container orchestration technologies.

### What features do orchestration tools offer?
- **High Availability** or No Downtime
- **Scalability** or High Performance
- **Disaster Recovery** - Backup and Restore

### Kubernetes Components
#### Usecase
Simple Javascript application with a simple Database. We will see step by step how each component of Kubernetes actually helps you deploy your application and what the role of each of its components is.

**1. Node**

A node is a worker machine that runs containerized applications. It can be a physical or virtual machine that is part of the Kubernetes cluster. Nodes are responsible for running the containers and providing the necessary resources for them to execute.

**2. Pod**
- **Smallest Unit of Kubernetes**
- **Abstraction over container:** A pod creates a running environment or a layer on top of a container. The reason is that Kubernetes wants to abstract away the container runtime or container technologies so that we can replace them if we want to, and we don't have to work with container technology directly. Instead, we only interact with the Kubernetes layer.
- **Usually one application per pod:** A pod is meant to run one application container inside it. We can also run multiple containers inside one pod, but usually, it's only the case if you have one main application and a helper container or some side service that has to run inside that pod.

<p align="center">
  <img src="https://github.com/harshakoneru98/kubernetes-starter/blob/main/images/1.%20Pod.png" alt="Usecase" width="400" height="500">
</p>

Now, in our use case, we have one server and two pods (my-app and DB). Let's see how they communicate with each other.

- **Each pod gets its own IP address:** Kubernetes offers a virtual network, which means that each pod gets its own IP address. Each pod can communicate with another pod using that IP address(Internal IP, not public).

- **New IP address on recreation:** What happens when pods crash? For example, if we lose a database container because the container crashed inside or because the nodes we are running them on are out of resources, then the pod will die. In this case, a new pod will be created in its place. When that happens, it will be assigned a new IP address, which can be inconvenient. If we are communicating with the database using an IP address, we need to adjust to the new IP address every time the pod restarts.

<p align="center">
  <img src="https://github.com/harshakoneru98/kubernetes-starter/blob/main/images/2.%20Pod%20with%20IP.png" alt="Pod with IP" width="400" height="500">
</p>

To overcome this, we will use another Kubernetes component, which is the Service.

**3. Service**

A Service is a static or permanent IP address that can be attached to each pod. So, my-app will have its own service, and the DB pod will have its own service. The lifecycle of the pod and the service are not connected. Even if the pod dies, the service and its IP address will remain unchanged. Therefore, we don't have to change the endpoint anymore.

Now, the app should be accessible through a browser. For this, we need to have an external service for my-app, such as http://my-app-service-ip:port, where it can communicate with external sources. However, for the db pod, we need to have an internal service because it should not be exposed to the public. These types of services will be specified when creating the service.

**4. Ingress**

The use of an external service, such as http://124.89.101.2:8080, is not practical because it exposes the HTTP protocol, IP address, and port number. Instead, we need to have a URL like https://my-app.com, which uses a secure protocol and a domain name. To achieve this proxy functionality in Kubernetes, there is a component named Ingress.

In this setup, the incoming request first goes to the Ingress, which then forwards it to the Service.

<p align="center">
  <img src="https://github.com/harshakoneru98/kubernetes-starter/blob/main/images/3.%20Service%20and%20Ingress.png" alt="Service and Ingress" width="900" height="500">
</p>

**5. ConfigMap**

Pods communicate with each other using Service. In my use case, my-app will have a database endpoint called mongo-db-service. Usually, these configurations are stored in the application properties file, which is typically located inside the built image of the application. If the name of the database endpoint changes to mongo-db, we would need to adjust the URL in the application. Then, we would need to rebuild the application with the new version and push it to the repository. Finally, we would have to pull the new image into our pod and restart the whole setup.

To overcome this challenge, Kubernetes has a component called ConfigMap. We can store all the external configuration data in it. Then, we can connect the ConfigMap to the pod so that the pod can access the data contained within the ConfigMap.

**6. Secret**

There can be configurations such as usernames and passwords for the database. Placing the credentials into ConfigMap in plain text would be insecure. To address this concern, Kubernetes has a component called Secret, which is used to store sensitive data and is encoded in base64. Similar to ConfigMap, we can connect Secret to the pod.

<p align="center">
  <img src="https://github.com/harshakoneru98/kubernetes-starter/blob/main/images/4.%20ConfigMap%20and%20Secret.png" alt="ConfigMap and Secret" width="600" height="500">
</p>

**7. Volumes**

If the database container or the pod is restarted, the data will be lost, which is problematic. To overcome this issue, Kubernetes offers a component called Volumes. The way it works is by attaching a physical storage device, such as a hard drive, to our pod. This storage can be located either on the local machine (on the same server) or on remote storage (outside the Kubernetes cluster), such as cloud or on-premise storage. The use of external storage is necessary because Kubernetes does not manage data persistence.

<p align="center">
  <img src="https://github.com/harshakoneru98/kubernetes-starter/blob/main/images/5.%20Volumes.png" alt="Volumes" width="400" height="500">
</p>

**8. Deployment**

Now everything is running perfectly, and users can access our application through a browser. However, with this setup, what happens if my application pod dies or I have to restart it due to building a new container image? Essentially, I would experience downtime. To overcome this, we need to replicate everything on multiple servers. Therefore, we have another node where a replica of our application will run, and it will also be connected to the same Service, which has a permanent IP and acts as a load balancer.

Instead of replicating the application pod multiple times, we define a blueprint for the my-app pod and specify how many replicas of that pod we would like to run. This blueprint or specification is called a deployment, which is another Kubernetes component. In practice, we will not be working directly with pods; we will be creating deployments because they allow us to specify the desired number of replicas and also provide scalability options.

As we mentioned, the pod is an abstraction layer on top of the container. Similarly, a deployment is another abstraction layer on top of the pod, making it more convenient to interact with pods and replicate them.

**9. StatefulSet**

What if our database pod dies? Our application would also become inaccessible. Therefore, we need a replica of the database as well. However, we cannot replicate the database using a deployment because the database has a state which refers to its data. This means that if we have replicas of the database, they would all need to access the same shared data storage. We require a mechanism to manage which pods are currently writing to or reading from that storage in order to avoid data inconsistencies. This mechanism, along with the replication feature, is provided by another Kubernetes component called StatefulSet. StatefulSet is primarily used for database applications.

<p align="center">
  <img src="https://github.com/harshakoneru98/kubernetes-starter/blob/main/images/6.%20Deployement%20and%20StatefulSet.png" alt="Deployment and StatefulSet" width="600" height="500">
</p>

Deploying database applications using StatefulSet in a Kubernetes Cluster is not an easy task. Hence, it is a common practice to host database applications outside the Kubernetes cluster. Inside the Kubernetes cluster, we can have deployments or stateless applications that can replicate and scale without any issues. These applications can then communicate with an external database. 

### Kubernetes Architecture
There are two types of nodes in Kubernetes: the Master Node and the Worker Node.

#### Master Node
The Master Node in Kubernetes is responsible for managing and coordinating cluster operations. It includes various components for this purpose:
- **API Server:** The API Server in Kubernetes serves as the front-end for the control plane, exposing the Kubernetes API. It validates and processes incoming requests from clients, such as the command-line interface and web UI, to interact with the cluster. The API Server then updates the cluster's desired state based on these requests.
- **Scheduler:** The Scheduler in Kubernetes decides where and how to run pods, considering factors like resource requirements, constraints, and policies. It analyzes the resources available on worker nodes and intelligently determines the best placement for pods. It takes into account factors such as resource availability, affinity/anti-affinity rules, and custom scheduling requirements to assign pods to suitable nodes.
- **Controller Manager:** The Controller Manager in Kubernetes runs multiple controllers to handle different aspects of the cluster's desired state. These controllers continuously monitor the cluster's current state and compare it with the desired state specified in the configuration. If any discrepancies are found, the controller takes necessary actions to align the state accordingly.
- **etcd:** etcd is a distributed key-value store used as the cluster's backing store for storing the Kubernetes API objects, cluster state, and configuration data. It provides a reliable and consistent data store for the entire cluster. The API Server and other components read from and write to etcd to maintain the cluster's state and ensure consistency.

#### Worker Node
Worker Nodes, also known as minion nodes or simply nodes, form the worker layer of the Kubernetes cluster. They are responsible for running the actual containerized workloads (pods). Each worker node includes the following components:
- **Kubelet:** Kubelet is the primary agent that runs on each worker node and interacts with the control plane. It receives pod definitions from the API Server, ensures that the desired state of pods is maintained on the node, and reports back the current state. Kubelet works closely with the container runtime to manage the containers' lifecycle, including starting, stopping, and monitoring them.
- **Container Runtime:** The Container Runtime is the software responsible for running containers. Kubernetes supports multiple container runtimes, with Docker being the most commonly used. Other runtimes include containerd, CRI-O, and rkt. The Container Runtime pulls container images from the registry, starts and stops containers, and manages their namespaces, networking, and storage.
- **Kube-proxy:** Kube-proxy is responsible for network proxying and service discovery within the cluster. It maintains network rules and enables communication between services running on different nodes. Kube-proxy implements the Kubernetes Service concept by managing virtual IP addresses and load balancing across pods in a service.

Each worker node typically runs multiple pods, and the Kubelet on each node ensures that the desired state of those pods is met by interacting with the Master Node and the container runtime.

The collaboration between the Master Node and Worker Nodes allows Kubernetes to efficiently manage the cluster's resources, orchestrate container deployments, handle scaling and resilience, and maintain the desired state of the system.

### Local Setup using Minikube and kubectl
In a production cluster, we typically have multiple Master and Worker Nodes, which are separate machines representing each node. However, when we want to quickly test or try something on our local environment, setting up a full production-like cluster can be challenging or even impossible due to resource limitations. To address this scenario, there is an open-source tool called Minikube.

#### What is Minikube?
Minikube is a tool that enables you to run a single-node Kubernetes cluster locally on your computer using Virtual Box or some other hypervisor like qemu, hyperkit etc. It set up a lightweight cluster that includes the necessary components of Kubernetes, such as the master node, worker node, and other supporting services. It provides a convenient way to learn, develop, and debug applications for Kubernetes without the need for a full-scale production cluster.

After setting up a cluster, we need some way to interact with a cluster. This is where kubectl comes into play.

#### What is kubectl?
kubectl is the command-line interface (CLI) tool used to interact with Kubernetes clusters. It is a powerful tool that allows users to manage and control various aspects of the Kubernetes cluster and its resources. With kubectl, you can perform actions such as deploying and managing applications, inspecting and modifying cluster resources, scaling deployments, viewing logs, and executing commands within containers.

kubectl communicates with the Kubernetes API server, which acts as the control plane for the cluster. It sends requests to the API server to perform operations on the cluster and receives responses with the results. kubectl can interact with any type of cluster, i.e. Minikube cluster or Cloud cluster.
