# Kubernetes Deployment for ML Model

This repository contains Kubernetes configuration files to deploy a machine learning model as a service using Kubernetes.

## Overview

The deployment consists of:
- A **Deployment** (`mlmodel-deployment.yaml`) to manage the ML model pods.
- A **Service** (`mlmodel-service.yaml`) to expose the model inside and outside the cluster.

## Deployment Details

### **Deployment (`mlmodel-deployment.yaml`):**
- Deploys **2 replicas** of the container `fredyball/mlmodel:latest`.
- Each container runs on **port 8000**.
- Resource limits are set to **512Mi memory** and **500m CPU**.
- Requests for resources are set to **256Mi memory** and **250m CPU**.

### **Service (`mlmodel-service.yaml`):**
- Exposes the ML model service via **port 8000**.
- Uses a **LoadBalancer** type for external access.
- Internally directs traffic to pods matching the `app: mlmodel` label.

## Deployment Instructions

### **1. Apply Kubernetes YAML files**
Run the following command to deploy the service:
```sh
kubectl apply -f mlmodel.yaml
```

### **2. Verify that the pods are running**
```sh
kubectl get pods -o wide
```
Expected output:
```
NAME                                READY   STATUS    RESTARTS   AGE   IP           NODE
mlmodel-deployment-xxxxx            1/1     Running   0          2m    10.244.x.x   worker-node
```

### **3. Check if the service is created**
```sh
kubectl get svc mlmodel-service
```
Expected output:
```
NAME               TYPE           CLUSTER-IP      EXTERNAL-IP     PORT(S)        AGE
mlmodel-service   LoadBalancer   10.96.52.212    <external-ip>   8000:30008/TCP  2m
```

### **4. Test the service inside the cluster**
Run a temporary pod with `curl`:
```sh
kubectl run curl-test --rm -it --image=curlimages/curl -- sh
```
Inside the pod, test the API:
```sh
curl -L -X POST http://mlmodel-service:8000/prediction -H "Content-Type: application/json" -d '{"day":5,"hour":12}'
```

### **5. Test the service from outside the cluster**
If the `LoadBalancer` has assigned an external IP, test with:
```sh
curl -L -X POST http://<external-ip>:8000/prediction -H "Content-Type: application/json" -d '{"day":5,"hour":12}'
```

## Debugging Issues

### **Check logs of a pod**
```sh
kubectl logs <pod-name>
```

### **Check service endpoints**
```sh
kubectl get endpoints mlmodel-service
```

### **Delete and redeploy the application**
```sh
kubectl delete deployment mlmodel-deployment
kubectl apply -f mlmodel.yaml
```

## Conclusion
This setup allows you to deploy and expose a machine learning model via Kubernetes. The `LoadBalancer` service ensures external access, and you can verify functionality using `curl` commands inside and outside the cluster.

