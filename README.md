# Kubernetes Web Deployment

A production-grade Kubernetes deployment built as part of my #100DaysOfDevOps challenge.

## What this deploys

A self-healing web application with:
- **3 replicas** — always 3 pods running, automatically replaced if one dies
- **Health checks** — liveness and readiness probes on every pod
- **Resource limits** — CPU and memory limits enforced at the kernel level
- **ConfigMap** — content separated from the container image
- **NodePort Service** — externally accessible on port 31437

## How to deploy

```bash
kubectl apply -f configmap.yaml
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
```

## How to verify

```bash
# Check pods are running
kubectl get pods

# Check service
kubectl get svc web

# Hit the app
curl http://<node-ip>:31437
```

## How self-healing works

```bash
# Delete all pods — they come back automatically
kubectl delete pods --all
kubectl get pods -w
```

## What I learned

- Pods are the equivalent of EC2 instances in Kubernetes
- Deployments are the equivalent of Auto Scaling Groups
- Services are the equivalent of Target Groups
- ConfigMaps are the equivalent of user_data scripts
- Resource limits write directly to Linux cgroup files in the kernel

## Part of a larger project

This Kubernetes deployment is the containerised version of my 
[Day 27 multi-region AWS HA project](link-to-your-aws-repo).

Same app. Same concepts. Different infrastructure.

## Stack

- Kubernetes v1.35
- nginx:alpine
- Cilium CNI
