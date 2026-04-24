# Kubernetes Web Deployment

A production-grade Kubernetes deployment built as part of my #100DaysOfDevOps challenge (Days 32-35).

## Architecture
Browser (Host: web.local)
    │
    ▼
Ingress Controller (nginx)
    │
    └── hostname-based routing
            │
            ▼
      Service (NodePort :31437)
            │
            └── load balances across pods
                    │
                    ├── Pod 1
                    ├── Pod 2
                    └── Pod 3
                         │
                         └── ConfigMap (HTML content)

## AWS to Kubernetes mapping

| AWS | Kubernetes | What it does |
|-----|-----------|--------------|
| Auto Scaling Group | Deployment | Keeps N pods running always |
| EC2 instance | Pod | Runs the app |
| Target Group | Service | Stable endpoint, load balancing |
| ALB | Ingress | Hostname routing, single entry point |
| user_data | ConfigMap | App content, separate from image |

## Features

- Self-healing — delete a pod, a new one appears in seconds
- Health checks — liveness and readiness probes on every pod
- Resource limits — CPU and memory enforced at the kernel level
- Hostname routing — Ingress routes by Host header
- Zero downtime — rolling updates by default

## Prerequisites

- Kubernetes cluster (tested on v1.35)
- nginx Ingress controller

## Install Ingress controller

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.10.1/deploy/static/provider/baremetal/deploy.yaml
```

## Deploy

```bash
kubectl apply -f configmap.yaml
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
kubectl apply -f ingress.yaml
```

## Verify

```bash
kubectl get pods
kubectl get svc web
kubectl get ingress web

INGRESS_PORT=$(kubectl get svc -n ingress-nginx \
  ingress-nginx-controller \
  -o jsonpath='{.spec.ports[?(@.port==80)].nodePort}')

curl -H "Host: web.local" http://<node-ip>:$INGRESS_PORT
```

## Self-healing demo

```bash
kubectl delete pods --all
kubectl get pods -w
```

## Key things I learned

**Pods are ephemeral.** They die and come back with different IPs. Never rely on a pod IP directly.

**Services solve the IP problem.** Stable virtual IP that always finds healthy pods using label selectors.

**Deployments solve the reliability problem.** Declare the desired state, Kubernetes enforces it forever.

**Ingress solves the routing problem.** One entry point, route to many services by hostname or path.

**Resource limits go all the way to the kernel.** Setting `memory: 64Mi` writes `67108864` to a cgroup file. I proved this live by reading `/sys/fs/cgroup` directly on the node.

## Related projects

- [30 Days Terraform Challenge](https://github.com/elorm116) — includes Day 27 multi-region HA on AWS. This Kubernetes project is the containerised version of that infrastructure.

## Stack

- Kubernetes v1.35.1
- nginx:alpine
- nginx Ingress Controller v1.10.1
- Cilium CNI
