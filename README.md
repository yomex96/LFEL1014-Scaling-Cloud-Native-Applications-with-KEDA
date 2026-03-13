## Kubernetes Autoscaling Labs – HPA & KEDA

This repository contains hands-on lab exercises completed as part of the **Scaling Cloud Native Applications with KEDA (LFEL1014)** course by the **The Linux Foundation**.

The labs demonstrate how to configure a Kubernetes environment, implement **Horizontal Pod Autoscaling (HPA)**, and deploy **KEDA** to enable event-driven autoscaling.

---

# Lab 1 – Setting Up the Course Lab Environment

## Overview

In this lab, the Kubernetes environment was prepared with the essential tools required to run and manage a cluster locally. The environment was built using **kind (Kubernetes in Docker)** and includes monitoring capabilities through the **Metrics Server**.

## Tools Installed

The following tools were installed and configured:

* Docker – Container runtime used by kind
* kind – Tool for running local Kubernetes clusters using Docker
* kubectl – Kubernetes command-line interface
* Helm – Kubernetes package manager
* Siege – Load testing and benchmarking tool

## Key Setup Steps

### Install Docker

```bash
curl -fsSL https://get.docker.com/ | sh
sudo systemctl enable --now docker
sudo usermod -aG docker $USER
```

### Install kubectl

```bash
curl -sSL -O "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl
sudo mv kubectl /usr/local/bin
```

### Install Helm

```bash
curl -sSL -O https://get.helm.sh/helm-v3.19.0-linux-amd64.tar.gz
tar -zxf helm-v3.19.0-linux-amd64.tar.gz
sudo mv linux-amd64/helm /usr/local/bin/helm
```

### Install Siege

```bash
sudo apt-get install siege
```

### Install kind

```bash
curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.30.0/kind-linux-amd64
chmod +x ./kind
sudo mv ./kind /usr/local/bin/kind
```

### Create Kubernetes Cluster

```bash
kind create cluster
kubectl get ns
```

### Install Metrics Server

```bash
helm repo add metrics-server https://kubernetes-sigs.github.io/metrics-server/
helm upgrade --install metrics-server metrics-server/metrics-server -n kube-system --set args[0]=--kubelet-insecure-tls
```

This enables Kubernetes to collect CPU and memory metrics used by autoscaling features.

---

# Lab 2 – Implementing Horizontal Pod Autoscaler (HPA)

## Overview

This lab demonstrates how to configure **Horizontal Pod Autoscaling (HPA)** in Kubernetes. The HPA automatically adjusts the number of pods in a deployment based on CPU utilization.

## Prerequisites

* Kubernetes cluster
* Metrics Server installed
* kubectl configured

## Deploy Sample Application

Create `webapp.yaml`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webapp
spec:
  replicas: 2
  selector:
    matchLabels:
      app: webapp
  template:
    metadata:
      labels:
        app: webapp
    spec:
      containers:
      - name: nginx
        image: nginx
        resources:
          limits:
            cpu: "10m"
          requests:
            cpu: "10m"
```

Deploy the application:

```bash
kubectl apply -f webapp.yaml
kubectl get deployments
```

### Access the Application

```bash
kubectl port-forward deploy/webapp 8080:80
```

Test with:

```bash
curl http://localhost:8080
```

---

## Configure Horizontal Pod Autoscaler

Create HPA:

```bash
kubectl autoscale deployment webapp --min=2 --max=5 --cpu-percent=20
```

Verify HPA:

```bash
kubectl get hpa
```

---

## Generate Load

Use Siege to simulate traffic:

```bash
siege -q -c 2 -t 1m http://localhost:8080
```

Monitor autoscaling:

```bash
kubectl get hpa -w
```

When CPU utilization exceeds the threshold, Kubernetes automatically scales the deployment.

Example output:

```
NAME    REFERENCE          TARGETS   MINPODS   MAXPODS   REPLICAS
webapp  Deployment/webapp  20%/20%   2         5         3
```

---

## Cleanup

```bash
kubectl delete hpa webapp
kubectl delete deploy webapp
```

---

# Lab 3 – Installing KEDA and Configuring Cron Scaler

## Overview

This lab introduces **KEDA (Kubernetes Event-Driven Autoscaling)** and demonstrates how to scale applications based on a **Cron schedule** instead of CPU metrics.

## Install KEDA

```bash
helm repo add kedacore https://kedacore.github.io/charts
helm repo update
helm upgrade -i keda kedacore/keda --namespace keda --create-namespace
```

Verify installation:

```bash
kubectl get deployment -n keda
```

---

## Deploy Sample Application

```bash
kubectl create deploy myapp --image nginx --replicas=2
kubectl get deployments
```

---

## Create Cron ScaledObject

Create `cron.yaml`:

```yaml
apiVersion: keda.sh/v1alpha1
kind: ScaledObject
metadata:
  name: cron-scaledobject
  namespace: default
spec:
  scaleTargetRef:
    name: myapp
  triggers:
  - type: cron
    metadata:
      timezone: Asia/Kolkata
      start: 30 * * * *
      end: 45 * * * *
      desiredReplicas: "10"
```

Apply configuration:

```bash
kubectl apply -f cron.yaml
```

Verify ScaledObject:

```bash
kubectl get scaledobject.keda.sh
```

---

## Monitor Scaling

```bash
watch kubectl get all
```

During the defined cron schedule, the deployment scales automatically to the configured number of replicas.

Example:

```
deployment.apps/myapp 10/10 10 10
```

---

# Key Concepts Learned

* Kubernetes cluster setup with **kind**
* Kubernetes monitoring using **Metrics Server**
* **Horizontal Pod Autoscaling (HPA)** based on CPU metrics
* Load testing using **Siege**
* Installing and managing applications using **Helm**
* Event-driven autoscaling using **KEDA**
* Scheduled scaling with **Cron ScaledObjects**

---

✅ These labs demonstrate **how modern cloud-native applications scale automatically based on metrics and events**, improving performance and cost efficiency in Kubernetes environments.

