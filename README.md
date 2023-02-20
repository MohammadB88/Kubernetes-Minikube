# **Kubernetes-Minikube**
In this repository, I will document preparing the reuquirement infrustructure and resources as well as the installation path for a minikube and a complete Kubernetes cluster with one controlplane and two worker nodes. 
Besides, we take a look at how to reach the applications and services on these clusters.

Later on, I will use Ansible playbooks to automate the preparation and configurations.

# **Table of Content**
- [**Kubernetes-Minikube**](#kubernetes-minikube)
- [**Table of Content**](#table-of-content)
  - [**Minimal Kubernetes Cluster**](#minimal-kubernetes-cluster)
  - [**Minikube Cluster**](#minikube-cluster)
      - [Auto-Completion and Alias for kubectl](#auto-completion-and-alias-for-kubectl)
  - [**Health Ckeck**](#health-ckeck)
    - [**ReadinessProbe \& LivenessProbe**](#readinessprobe--livenessprobe)

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


#### Auto-Completion and Alias for kubectl
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
Kubernetes uses *readinessProbe* and *livenessProbe* to determine the health of a container in a pod.

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