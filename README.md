# Kubernetes-Minikube
In this repository, I will document preparing the reuquirement infrustructure and resources as well as the installation path for a minikube and a complete Kubernetes cluster with one controlplane and two worker nodes. 
Besides, we take a look at how to reach the applications and services on these clusters.

Later on, I will use Ansible playbooks to automate the preparation and configurations.

## Kubernetes Cluster with one controlplane and two worker nodes

## Minikube Cluster
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


Set an alias for kubectl as ***k*** and enable the alias for auto-completion:
```
echo 'alias k=kubectl' >>~/.bashrc
echo 'complete -o default -F __start_kubectl k' >>~/.bashrc
```

To reach the services and running apps, use port forwarding feature in kubectl
```
kubectl port-forward svc/SVC_NAME HOST_PORT:SVC_PORT
```

