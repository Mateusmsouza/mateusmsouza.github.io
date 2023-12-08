---
layout: post
title: 'Creating Mutating Web Hooks for Kubernetes: A Step-by-Step Guide'
categories: [DevOps, Kubernetes]
---

In the ever-evolving landscape of Kubernetes orchestration, extending and customizing functionality is crucial. Mutating web hooks offer a powerful mechanism for dynamically altering objects before they are persisted in the cluster. In this guide, we'll walk through the process of creating a mutating web hook for Kubernetes, from generating SSL certificates to deploying and testing the webhook service.

# Requirements

For this guide I will be using Minikube, If you need some guidance on how to setup a local cluster check [this](https://mateusmsouza.github.io/Creating-your-local-Kubernetes-cluster/).

# Generating SSL certificate

Before diving into the coding and deployment of the mutating webhook, let's secure our communication with an SSL certificate. Execute the following commands to generate the required certificate and create a Kubernetes secret to store it:

```bash
$ openssl req -x509 -sha256 -newkey rsa:2048 -keyout webhook.key -out \
		webhook.crt -days 1024 \
		-nodes -addext "subjectAltName = DNS.1:<SERVICE_NAME>.<NAMESPACE>.svc"
```

Create a secret to store the certificate:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: webhook-cert
type: Opaque
data:
  webhook.crt: <CRT IN BASE64>
  webhook.key: <KEY IN BASE64>
```

Remember to base64 encode the certificate and keys using the **`cat <FILE_NAME> | base64 | tr -d '\n'`** command on Linux.

After these configurations you are already good to go:

```bash
$ kubectl apply -f secret-certificate.yaml
secret/webhook-cert created
$ kubectl get secrets
NAME           TYPE     DATA   AGE
webhook-cert   Opaque   2      58s
```

# Configuring the Webhook Service

Now, let's delve into the implementation of the webhook service using FastAPI and Docker. The Python code snippet below demonstrates a basic implementation of a mutating webhook service:

```python
from fastapi import FastAPI, Body

app = FastAPI()

@app.post("/collect")
def collect_data(request: dict = Body(...)):
    uid = request["request"]["uid"]
    object_in = request["request"]["object"]
    # Modify the object as needed
    print(f"Received object {uid}")
    return {
        "apiVersion": "admission.k8s.io/v1",
        "kind": "AdmissionReview",
        "response": {
            "uid": uid,
            "allowed": True,
            "patchType": "JSONPatch",
            "status": {"message": "You are good to go now."}
        },
    }
```

The accompanying Dockerfile ensures the correct setup for our Python application within a Kubernetes environment:

```docker
# Use the official Python image as the base image
FROM python:3.9

# Set the working directory to /app
WORKDIR /app

# Copy the current directory contents into the container at /app
COPY . /app

# Install any needed packages specified in requirements.txt
RUN pip install --no-cache-dir -r requirements.txt

# Expose port 5000 to the outside world
EXPOSE 5000

# Command to run the application
CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "5000", "--ssl-keyfile=/ssl/webhook.key", "--ssl-certfile=/ssl/webhook.crt"]
```

Build the Docker image using **`minikube image build . -t mutator-svc:0.1`**.

# ****Webhook Configuration Manifests****

Now, let's configure the necessary Kubernetes manifests for deploying the mutating webhook:

```yaml
# mutator-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mutating-webhook-deployment

spec:
  replicas: 1
  selector:
    matchLabels:
      app: mutator-deployment
  template:
    metadata:
      labels:
        app: mutator-deployment
    spec:
      containers:
        - name: mutator
          image: mutator-svc:0.1
          ports:
          - containerPort: 5000
          resources:
            limits:
              memory: "128Mi"
              cpu: "256m"
          
          volumeMounts:
          - name: certificate-volume
            readOnly: true
            mountPath: "/ssl"
      volumes:
      - name: certificate-volume
        secret:
          secretName: webhook-cert
```

```yaml
# mutator-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: mutating-webhook-service
spec:
  selector:
    app: mutator-deployment
  ports:
    - port: 5000
```

```yaml
# mutator-config.yaml
apiVersion: admissionregistration.k8s.io/v1
kind: MutatingWebhookConfiguration
metadata:
  name: mutating-webhook
webhooks:
- name: mutating-webhook-service.default.svc
  matchPolicy: Equivalent
  admissionReviewVersions: ["v1"]
  sideEffects: None
  rules:
  - operations: ["CREATE"]
    apiGroups: ["apps"]
    apiVersions: ["v1"]
    resources: ["deployments", "statefulsets", "daemonsets"]
    scope: "Namespaced"
  failurePolicy: Ignore
  timeoutSeconds: 20
  clientConfig:
    caBundle: <YOUR BASE64 ENCODED CERT>
    service:
      namespace: default
      name: mutating-webhook-service
      path: /collect
      port: 5000
```

Apply the manifests using **`kubectl apply -f mutator-deployment.yaml -f mutator-service.yaml -f mutator-config.yaml`**.

Check the status of your webhook pod using:

```yaml
kubectl get pods 
NAME                                           READY   STATUS    RESTARTS   AGE
mutating-webhook-deployment-76b9f4b578-vmfzj   1/1     Running   0          32m
```

# Testing the Webhook

For testing purposes, create a dummy deployment:

```yaml
#busybox.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: busybox-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: busybox-app
  template:
    metadata:
      labels:
        app: busybox-app
    spec:
      containers:
      - name: busybox-container
        image: busybox:latest
        command: ["sleep", "3600"]  # Keeps the container running for testing
```

Apply to the cluster `kubectl apply -f busybox.yaml`.

Check the logs of your webhook using:

```bash
$ kubectl logs -f mutating-webhook-deployment-<pod-id>
INFO:     Started server process [1]
INFO:     Waiting for application startup.
INFO:     Application startup complete.
INFO:     Uvicorn running on https://0.0.0.0:5000 (Press CTRL+C to quit)
Received object 3c36d97d-b33c-4c93-be16-07e6c932e751
INFO:     10.244.0.1:47039 - "POST /collect?timeout=20s HTTP/1.1" 200 OK
```

You should see logs indicating the reception and processing of objects.

This comprehensive guide equips you with the knowledge to create and deploy mutating web hooks in Kubernetes. As you explore further, consider customizing the implementation to suit your specific use cases and enhance the security and efficiency of your Kubernetes cluster. Happy coding!

## References

[https://kmitevski.com/kubernetes-mutating-webhook-with-python-and-fastapi/](https://kmitevski.com/kubernetes-mutating-webhook-with-python-and-fastapi/)
[https://kubernetes.io/docs/reference/access-authn-authz/extensible-admission-controllers/](https://kubernetes.io/docs/reference/access-authn-authz/extensible-admission-controllers/)
[https://medium.com/ovni/writing-a-very-basic-kubernetes-mutating-admission-webhook-398dbbcb63ec](https://medium.com/ovni/writing-a-very-basic-kubernetes-mutating-admission-webhook-398dbbcb63ec)