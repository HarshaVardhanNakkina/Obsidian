## Before you begin
This tutorial assumes that you have already set up `minikube`. See **Step 1** in [minikube start](https://minikube.sigs.k8s.io/docs/start/) for installation instructions.

>Note: only execute the instruction in Step 1, Installation. The rest is covered in this page.

You also need to install `kubectl`. See [Install tools](https://kubernetes.io/docs/tasks/tools/#kubectl) for installation instructions.

## Cluster Diagram
![[module_01_cluster.svg]]
**The Control Plane is responsible for managing the cluster.** The Control Plane coordinates all activities in your cluster, such as scheduling applications, maintaining applications' desired state, scaling applications, and rolling out new updates.

> Control Planes manage the cluster and the nodes that are used to host the running applications.

**A node is a VM or a physical computer that serves as a worker machine in a Kubernetes cluster.** Each node has a Kubelet, which is an agent for managing the node and communicating with the Kubernetes control plane. The node should also have tools for handling container operations, such as [containerd](https://containerd.io/docs/) or [CRI-O](https://cri-o.io/#what-is-cri-o). A Kubernetes cluster that handles production traffic should have a minimum of three nodes because if one node goes down, both an [etcd](https://kubernetes.io/docs/concepts/overview/components/#etcd) member and a control plane instance are lost, and redundancy is compromised. You can mitigate this risk by adding more control plane nodes.

When you deploy applications on Kubernetes, you tell the control plane to start the application containers. The control plane schedules the containers to run on the cluster's nodes. **Node-level components, such as the kubelet, communicate with the control plane using the [Kubernetes API](https://kubernetes.io/docs/concepts/overview/kubernetes-api/)**, which the control plane exposes. End users can also use the Kubernetes API directly to interact with the cluster.

A Kubernetes cluster can be deployed on either physical or virtual machines. To get started with Kubernetes development, you can use Minikube. Minikube is a lightweight Kubernetes implementation that creates a VM on your local machine and deploys a simple cluster containing only one node. Minikube is available for Linux, macOS, and Windows systems. The Minikube CLI provides basic bootstrapping operations for working with your cluster, including start, stop, status, and delete.
## Create a minikube cluster
```powershell
minikube start
```

## Open the Dashboard

Open new terminal, and run:
```shell
# Start a new terminal, and leave this running
minikube dashboard
```

If you don't want minikube to open a web browser for you, run the `dashboard` command with the `--url` flag. `minikube` outputs a URL that you can open in the browser you prefer.
```shell
minikube dashboard --url
```

> **Note:** 
> The `dashboard` command enables the dashboard add-on and opens the proxy in the default web browser. You can create Kubernetes resources on the dashboard such as Deployment and Service. 
> To find out how to avoid directly invoking the browser from the terminal and get a URL for the web dashboard, see the "URL copy and paste" tab.
> By default, the dashboard is only accessible from within the internal Kubernetes virtual network. The `dashboard` command creates a temporary proxy to make the dashboard accessible from outside the Kubernetes virtual network.
> To stop the proxy, run `Ctrl+C` to exit the process. After the command exits, the dashboard remains running in the Kubernetes cluster. You can run the `dashboard` command again to create another proxy to access the dashboard.

## Create a Deployment

A Kubernetes [_Pod_](https://kubernetes.io/docs/concepts/workloads/pods/) is a group of one or more Containers, tied together for the purposes of administration and networking. The Pod in this tutorial has only one Container. A Kubernetes [_Deployment_](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/) checks on the health of your Pod and restarts the Pod's Container if it terminates. Deployments are the recommended way to manage the creation and scaling of Pods.

1. Use the `kubectl create` command to create a Deployment that manages a Pod. The pod runs a Container based on the provided Docker image.
   ```shell
# Run a test container image that includes a webserver
kubectl create deployment hello-node --image=registry.k8s.io/e2e-test-images/agnhost:2.39 -- /agnhost netexec --http-port=8080
```
2. View the Deployment
```shell
kubectl get deployments
```
The output is similar to:
```
NAME         READY   UP-TO-DATE   AVAILABLE   AGE
hello-node   1/1     1            1           1m
```
3. View the Pod:
```shell
kubectl get pods
```
The output is similar to:
```
NAME                          READY     STATUS    RESTARTS   AGE
hello-node-5f76cf6ccf-br9b5   1/1       Running   0          1m
```
4. View cluster events:
```shell
kubectl get events
```
5. View the `kubectl` configuration:
```shell
kubectl config view
```
6. View application logs for a container in a pod.
```shell
kubectl logs hello-node-5f76cf6ccf-br9b5
```
The output is similar to:
```
I0911 09:19:26.677397       1 log.go:195] Started HTTP server on port 8080
I0911 09:19:26.677586       1 log.go:195] Started UDP server on port  8081
```

## Create a service

By default, the Pod is only accessible by its internal IP address within the Kubernetes cluster. To make the `hello-node` Container accessible from outside the Kubernetes virtual network, you have to expose the Pod as Kubernetes [_Service_](https://kubernetes.io/docs/concepts/services-networking/service/).

1. Expose the Pod to the public internet using the `kubectl expose` command:
```shell
kubectl expose deployment hello-node --type=LoadBalancer --port=8080
```

The `--type=LoadBalancer` flag indicates that you want to expose your Service outside of the cluster.
The application code inside the test image only listens on TCP port 8080. If you used `kubectl expose` to expose a different port, clients could not connect to that other port.

2. View the Service you created:
```shell
kubectl get services
```

The output is similar to:
```
NAME         TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
hello-node   LoadBalancer   10.108.144.78   <pending>     8080:30369/TCP   21s
kubernetes   ClusterIP      10.96.0.1       <none>        443/TCP          23m
```

On cloud providers that support load balancers, an external IP address would be provisioned to access the Service. On minikube, the `LoadBalancer` type makes the Service accessible through the `minikube service` command.

3. Run the following command:
```
minikube service hello-node
```

This opens up a browser that serves your app and shows the app's response.

## Clean up

Now you can clean up the resources you created in your cluster:
```shell
kubectl delete service hello-node
kubectl delete deployment hello-node
```

Stop the Minikube cluster
```shell
minikube stop
```

Optionally, delete the Minikube VM:
```shell
# Optional
minikube delete
```
