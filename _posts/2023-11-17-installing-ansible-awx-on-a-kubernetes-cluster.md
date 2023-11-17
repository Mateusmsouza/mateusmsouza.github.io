---
layout: post
title: Installing Ansible AWX on a Kubernetes Cluster
categories: [DevOps, Kubernetes, Ansible, AWX]
---
On this quick post I will show how to setup an AWX installation on a Kubernetes Cluster using AWX Operator.

## Provisioning a local Kubernetes cluster

For this we will be using Minikube as our cluster, you can check my tutorial on [Creating your local Kubernetes cluster](https://mateusmsouza.github.io/Creating-your-local-Kubernetes-cluster/) for instructions on how to install and setup Minikube.

```bash
$ minikube start
😄  minikube v1.30.1 en Ubuntu 22.04
    ▪ KUBECONFIG=/home/mateus/Projects/kube-configs/mgc-ipet-produtos-mgl1-dev.yaml
✨  Using the docker driver based on existing profile
👍  Starting control plane node minikube in cluster minikube
🚜  Pulling base image ...
🎉  minikube 1.32.0 is available! Download it: https://github.com/kubernetes/minikube/releases/tag/v1.32.0
💡  To disable this notice, run: 'minikube config set WantUpdateNotification false'

🔄  Restarting existing docker container for "minikube" ...
🐳  Preparando Kubernetes v1.26.3 en Docker 23.0.2...
🔗  Configurando CNI bridge CNI ...
🔎  Verifying Kubernetes components...
    ▪ Using image docker.io/kubernetesui/dashboard:v2.7.0
    ▪ Using image docker.io/kubernetesui/metrics-scraper:v1.0.8
    ▪ Using image registry.k8s.io/metrics-server/metrics-server:v0.6.3
    ▪ Using image gcr.io/k8s-minikube/storage-provisioner:v5
💡  Some dashboard features require the metrics-server addon. To enable all features please run:

	minikube addons enable metrics-server	

🌟  Complementos habilitados: default-storageclass, storage-provisioner, metrics-server, dashboard

❗  /snap/bin/kubectl is version 1.28.4, which may have incompatibilities with Kubernetes 1.26.3.
    ▪ Want kubectl v1.26.3? Try 'minikube kubectl -- get pods -A'
🏄  Done! kubectl is now configured to use "minikube" cluster and "default" namespace by default
```

## Install Helm

Helm makes a lot easier to install applications on a Kubernetes cluster:

```bash
$ curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
$ chmod +x get_helm.sh &&./get_helm.sh
$ helm version
```

## Install AWX Chart

```bash
$ helm repo add awx-operator https://ansible.github.io/awx-operator/
"awx-operator" has been added to your repositories
$ helm repo update
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "awx-operator" chart repository
Update Complete. ⎈Happy Helming!⎈
$ helm install ansible-awx-operator awx-operator/awx-operator \
			-n awx-operator --create-namespace
NAME: ansible-awx-operator
LAST DEPLOYED: Fri Nov 17 16:02:19 2023
NAMESPACE: awx-operator
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
AWX Operator installed with Helm Chart version 2.7.2
```

## Deploying AWX

Create your AWX Deployment, create a file:

```yaml
# awx.yaml
apiVersion: awx.ansible.com/v1beta1
kind: AWX
metadata:
  name: awx
  namespace: awx-operator
spec:
  service_type: nodeport
  ingress_type: none
  hostname: awx.awx-operator.com
```

Apply it to the cluster:

```bash
$ kubectl apply -f awx.yaml 
awx.awx.ansible.com/awx created
```

Check if everything is up and running

```bash
$ kubectl get pods -n awx-operator
NAME                                               READY   STATUS    RESTARTS   AGE
awx-operator-controller-manager-774d74ddd7-p2bgx   2/2     Running   0          14m
awx-postgres-13-0                                  1/1     Running   0          5m12s
awx-task-754fdf8699-975f9                          4/4     Running   0          4m21s
awx-web-7ddfb6549f-qs756                           3/3     Running   0          2m47s
```

## Acessing the GUI

The quickest way to login is using a port foward command:

```bash
$ kubectl port-forward svc/awx-service -n awx-operator 8080:80
Forwarding from 127.0.0.1:8080 -> 8052
Forwarding from [::1]:8080 -> 8052
Handling connection for 8080
Handling connection for 8080
Handling connection for 8080
Handling connection for 8080
```

You will be able to access the GUI on [`http://127.0.0.1:8080/#/login`](http://127.0.0.1:8080/#/login). 

The user is admin and the password is stored in a Kubernetes Secret.

```bash
$ kubectl get secrets -n awx-operator                         
NAME                                         TYPE                 DATA   AGE
awx-admin-password                           Opaque               1      9m59s
.......
.......
........
sh.helm.release.v1.ansible-awx-operator.v1   helm.sh/release.v1   1      18m

$ kubectl get secret awx-admin-password -o jsonpath="{.data.password}" -n awx-operator | base64 --decode
3qVpAN54Rj68K5UXVwdKux5HXHkwmb0n
```

![Untitled](Installing%20Ansible%20AWX%20on%20a%20Kubernetes%20Cluster%20ebd328ad24ab40a4b9bf0b49fee58066/Untitled.png)