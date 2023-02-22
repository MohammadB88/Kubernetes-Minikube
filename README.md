# **Kubernetes-Minikube**
In this repository, I will document preparing the reuquirement infrustructure and resources as well as the installation path for a minikube and a complete Kubernetes cluster with one controlplane and two worker nodes. 
Besides, we take a look at how to reach the applications and services on these clusters.

Later on, I will use Ansible playbooks to automate the preparation and configurations.

# **Table of Content**
- [**Kubernetes-Minikube**](#kubernetes-minikube)
- [**Table of Content**](#table-of-content)
  - [**Minimal Kubernetes Cluster**](#minimal-kubernetes-cluster)
  - [**Minikube Cluster**](#minikube-cluster)
    - [**Metrics Server**](#metrics-server)
  - [**Auto-Completion and Alias for kubectl**](#auto-completion-and-alias-for-kubectl)
  - [**Health Ckeck**](#health-ckeck)
    - [**ReadinessProbe \& LivenessProbe**](#readinessprobe--livenessprobe)
  - [**Resource Requests and Limits**](#resource-requests-and-limits)
  - [**Auto Scaling**](#auto-scaling)
    - [**Horizontal Pod Autoscaling (HPA)**](#horizontal-pod-autoscaling-hpa)

## **Minimal Kubernetes Cluster**
This Cluster has one controlplane and two worker nodes

## **Minikube Cluster**
Here are the steps to get Minikube installed and running on WSL in Windows 10:
1. Install WSL2 on windows and check it was finished successfully
   Run the following on powershell
```
wsl --list --verbose
```

2. Install Docker Desktop and activate the WSL Integration in
 
   ***Settings*** >> ***Resources*** >> ***WSL Integration*** >> ***Apply & Restart***

Then check the Isntallation with the same command as in the first step

3. Follow the step by step guide in [How To Install Minikube on Ubuntu](https://computingforgeeks.com/how-to-install-minikube-on-ubuntu-debian-linux/) to install Minikube. ***Skip the second step***

### **Metrics Server**
The **Metrics Server addon** for Minikube is an optional component that provides resource utilization metrics for the Kubernetes cluster running in a local Minikube environment.

The Metrics Server is a cluster-wide aggregator of resource usage data, including CPU and memory usage of nodes and containers. It collects this data from the kubelet, which is an agent that runs on each node in the Kubernetes cluster and is responsible for managing containers.

The Metrics Server addon for Minikube provides the following benefits:

   - It allows you to monitor the resource utilization of your Kubernetes cluster, which can help you identify performance issues, bottlenecks, and potential capacity constraints.
   - It enables you to autoscale your applications based on the resource usage metrics collected by the Metrics Server, which can improve the efficiency and reliability of your Kubernetes cluster.
   - It provides the foundation for other Kubernetes components and tools that rely on resource utilization metrics, such as the Horizontal Pod Autoscaler (HPA).

To enable the Metrics Server addon in Minikube, you can use the following command:
````
minikube addons enable metrics-server
````
Once enabled, the Metrics Server will be deployed in the Minikube cluster, and you can access the resource utilization metrics using the kubectl top command. For example, you can use the following command to view the CPU and memory usage of all pods in the default namespace:
````
kubectl top pods/nodes
````
A very nice guide can also be found in the below link:
[Minikube's addon: 'metrics-server'](http://www.mtitek.com/tutorials/kubernetes/kubernetes_metrics.php)

## **Auto-Completion and Alias for kubectl**
I have used this tutorial: [Enable Auto-completion for Kubectl in a Linux](https://spacelift.io/blog/kubectl-auto-completion).
Set an alias for kubectl as ***k*** and enable the alias for auto-completion:
```
echo 'alias k=kubectl' >>~/.bashrc
echo 'complete -o default -F __start_kubectl k' >>~/.bashrc
```

To reach the services and running apps, use port forwarding feature in kubectl
```
kubectl port-forward svc/SVC_NAME HOST_PORT:SVC_PORT --namespace NAMESPACE
```

## **Health Ckeck**

### **ReadinessProbe & LivenessProbe**
Kubernetes uses ***readinessProbe*** and ***livenessProbe*** to determine the health of a container in a pod.

The readinessProbe tells Kubernetes whether the container is ready to receive traffic. It checks if the container is able to respond to requests and if it's ready to start accepting traffic. If the readinessProbe fails, Kubernetes will not send traffic to the container until the probe passes.

The livenessProbe tells Kubernetes whether the container is alive and healthy. It checks if the container is still running and if it's able to respond to requests. If the livenessProbe fails, Kubernetes will restart the container to try to bring it back to a healthy state.

Using readinessProbe and livenessProbe is important to ensure the availability and reliability of your application. By setting up these probes, Kubernetes can automatically manage and scale your containers, replace failed containers, and maintain a high level of availability for your application.

Kubernetes supports several types of probes for readinessProbe and livenessProbe, which can be defined at the container level, inside the ***spec.template.spec.containers*** section of a Deployment or StatefulSet, or at the pod level, inside the ***spec.containers*** section of a Pod.
. These include:

   - **httpGet:** Sends an HTTP GET request to a specified path and port of the container. If the request returns a 200-399 response code, the container is considered healthy.
   ````
   readinessProbe:
      httpGet:
         path: /healthz
         port: 8080
      initialDelaySeconds: 5
      periodSeconds: 10
      timeoutSeconds: 5
      successThreshold: 1
      failureThreshold: 3
   livenessProbe:
      httpGet:
         path: /healthz
         port: 8080
      initialDelaySeconds: 5
      periodSeconds: 10
      timeoutSeconds: 5
      successThreshold: 1
      failureThreshold: 3
   ````
   This probe sends an HTTP GET request to /healthz path on port 8080 of the container. If the response code is in the 200-399 range, the container is considered ready. The probe runs after an initial delay of 5 seconds and then every 10 seconds.

   - **tcpSocket:** Attempts to open a TCP socket to a specified port of the container. If the socket can be opened, the container is considered healthy.
   ````
   readinessProbe:
      tcpSocket:
         port: 5432
      initialDelaySeconds: 15
      periodSeconds: 20
      timeoutSeconds: 5
      successThreshold: 1
      failureThreshold: 3
   livenessProbe:
      tcpSocket:
         port: 5432
      initialDelaySeconds: 15
      periodSeconds: 20
      timeoutSeconds: 5
      successThreshold: 1
      failureThreshold: 3
   ````
   This probe tries to open a TCP socket to port 5432 of the container. If the socket can be opened, the container is considered healthy. The probe starts after an initial delay of 15 seconds and then runs every 20 seconds.

   - **exec:** Executes a specified command inside the container. If the command exits with a zero status code, the container is considered healthy.
   ````
   readinessProbe:
      exec:
         command:
         - sh
         - -c
         - cat /tmp/ready
      initialDelaySeconds: 3
      periodSeconds: 5
      timeoutSeconds: 5
      successThreshold: 1
      failureThreshold: 3
   livenessProbe:
      exec:
         command:
         - sh
         - -c
         - cat /tmp/ready
      initialDelaySeconds: 3
      periodSeconds: 5
      timeoutSeconds: 5
      successThreshold: 1
      failureThreshold: 3
   ````
   This probe executes a command inside the container to check readiness. In this example, it runs the command cat /tmp/ready using the shell (sh) command. If the command exits with a zero status code, the container is considered ready. The probe starts after an initial delay of 3 seconds and then runs every 5 seconds.

For example, you can use httpGet to check if your web server is responding to requests, tcpSocket to check if your database is accepting connections, and exec to run a custom script to determine container health.

By default, httpGet and tcpSocket probes have a failure threshold of three consecutive failures before the container is considered unhealthy. You can adjust the threshold and other probe parameters such as the initial delay, period, and timeout to meet your application needs.


[Kubernetes Liveness and Readiness Probes: How to Avoid Shooting Yourself in the Foot](https://blog.colinbreck.com/kubernetes-liveness-and-readiness-probes-how-to-avoid-shooting-yourself-in-the-foot/)


## **Resource Requests and Limits**
Resource Requests and Limits are important concepts in Kubernetes that help ensure the stability, reliability, and optimal utilization of the cluster.

Resource Requests are the minimum amount of resources, such as CPU and memory, that a container needs to run. These requests are used by the Kubernetes scheduler to determine the best node to schedule the container on, based on the available resources on the node. By setting resource requests, you ensure that your container has enough resources to run reliably and avoid scheduling issues due to resource shortages.

Resource Limits, on the other hand, are the maximum amount of resources that a container is allowed to use. If a container exceeds its resource limit, it may be terminated by Kubernetes to prevent it from hogging the resources and causing issues for other containers on the same node. By setting resource limits, you ensure that your container does not consume too many resources and affect the performance of other containers.

Setting Resource Requests and Limits helps in the following ways:

   - Ensuring stable and reliable operation of the containers by avoiding resource starvation.
   - Helping Kubernetes scheduler in deciding where to schedule the container on the most suitable node based on available resources.
   - Preventing containers from consuming excessive resources, which may cause performance issues in the cluster.
   - Preventing a single container from hogging all the resources of a node, which could cause other containers on the same node to be affected.

Therefore, it is important to set resource requests and limits for your containers to ensure the optimal use of resources in the Kubernetes cluster and avoid resource-related issues.

Here's an example of how resource requests and limits might be set in a Kubernetes deployment:

````
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-web-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-web-app
  template:
    metadata:
      labels:
        app: my-web-app
    spec:
      containers:
      - name: web
        image: my-web-app-image
        resources:
          requests:
            cpu: "100m"
            memory: "256Mi"
          limits:
            cpu: "500m"
            memory: "512Mi"
````

## **Auto Scaling**
Autoscaling in Kubernetes is a feature that allows you to automatically scale your application up or down based on changes in resource utilization. Kubernetes provides several built-in autoscaling options, including Horizontal Pod Autoscaling (HPA), Vertical Pod Autoscaling (VPA), and Cluster Autoscaling.

Horizontal Pod Autoscaling (HPA) is the most commonly used type of autoscaling in Kubernetes. It automatically scales the number of replicas of a deployment or replica set based on the observed CPU or memory utilization of the pods. With HPA, you can define target CPU or memory utilization for your pods, and Kubernetes will automatically adjust the number of replicas to ensure that the target utilization is met.

Vertical Pod Autoscaling (VPA) adjusts the CPU and memory requests and limits for the pods based on the actual usage of resources. It is designed to help optimize the resource utilization of your pods by dynamically adjusting their resource requests and limits.

Cluster Autoscaling automatically scales the number of nodes in the Kubernetes cluster based on the resource utilization of the cluster. With Cluster Autoscaling, you can ensure that your Kubernetes cluster has enough capacity to handle the workload, and you can avoid overprovisioning the cluster, which can lead to wasted resources and increased costs.

Using autoscaling in Kubernetes has several benefits, including:

   - **Improved availability:** Autoscaling helps ensure that your application is available and responsive even under high load conditions.
   - **Cost optimization:** Autoscaling can help you optimize the cost of running your application by only using the resources that are necessary to handle the current workload. This can help you avoid overprovisioning and wasting resources.
   - **Increased efficiency:** Autoscaling can help you achieve more efficient use of resources by automatically adjusting the number of replicas or nodes based on the actual resource utilization of your application.
   - **Scalability:** Autoscaling allows you to easily handle sudden spikes in traffic or workload by automatically adding more resources to your application.

Overall, autoscaling is a powerful feature in Kubernetes that can help you achieve higher availability, efficiency, and scalability for your applications while optimizing costs. It is a recommended practice for managing applications in production environments.

Here are some examples of how to implement each type of autoscaling in Kubernetes:

### **Horizontal Pod Autoscaling (HPA)**

To enable HPA in Kubernetes, you need to have a metric server installed and running in your cluster. You can then create an HPA object that specifies the minimum and maximum number of replicas to maintain for a deployment, as well as the target resource utilization.

Here's an example YAML file that creates an HPA for a deployment that targets 50% CPU utilization and scales between 1 and 10 replicas:
````
apiVersion: autoscaling/v1
kind: HorizontalPodAutoscaler
metadata:
  creationTimestamp: null
  name: NAME -> it usually uses DEPLOYMENT_NAME
spec:
  maxReplicas: 10
  minReplicas: 1
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: DEPLOYMENT_NAME
  targetCPUUtilizationPercentage: 60
````

Or the settings could be directly added inside the Deployment manifest:
```` 
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: my-app
        image: my-image:latest
        ports:
        - containerPort: 80
        resources:
          requests:
            cpu: "100m"
            memory: "256Mi"
          limits:
            cpu: "500m"
            memory: "512Mi"
      autoscaling:
        minReplicas: 3
        maxReplicas: 6
        targetCPUUtilizationPercentage: 50
````

In order to test the nginx stability, I have used Apache Bench (ab) tool to send HTTP requests to the nginx proxy. This command will be executed inside a temporary container as:
````
kubectl run --image httpd -it httpd -- ab -n 20000 -c 1000 http://<nginx-service-ip>:<nginx-service-port>//
````
After the container is created, it runs the *ab* command within the container to send 20,000 requests to the nginx service, with a maximum of 1000 concurrent requests.
