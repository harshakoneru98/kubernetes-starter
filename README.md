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
If the database container or the pod is restarted, the data would be lost, which is problematic. To overcome this issue, Kubernetes offers a component called Volumes. The way it works is by attaching a physical storage device, such as a hard drive, to our pod. This storage can be located either on the local machine (on the same server) or on remote storage (outside the Kubernetes cluster), such as cloud or on-premise storage. The use of external storage is necessary because Kubernetes does not manage data persistence.
