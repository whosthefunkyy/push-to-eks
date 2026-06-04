# Kubernetes Practice

Basic Kubernetes setup with Go API deployed to minikube.

## Stack
- Kubernetes (minikube)
- Go API (whosthefunky/cicd-practice)
- Nginx Ingress Controller
- HPA (Horizontal Pod Autoscaler)

## What's inside
| File | Description |
|------|-------------|
| `deployment.yaml` | Go API deployment, 2 replicas, liveness/readiness probes, resource limits |
| `service.yaml` | NodePort service, port 8080 |
| `configmap.yaml` | App config (APP_ENV, APP_PORT) |
| `secret.yaml` | Sensitive data (DB_PASSWORD) in base64 |
| `ingress.yaml` | Nginx ingress, routes go-api.local to service |
| `hpa.yaml` | Autoscaling: min 2, max 5 pods, CPU threshold 50% |

All resources deployed to `dev` namespace.
## Quick start
```bash
minikube start
kubectl config set-context --current --namespace=dev
kubectl apply -f "file*".yaml

```

## Useful commands

```bash
kubectl get pods
kubectl get services
kubectl get hpa
kubectl describe pod <pod-name>
kubectl logs <pod-name>
```

## Notes
### Ingress on Mac (Docker driver)
Ingress IP is not routable directly from macOS. Use port-forward instead:

```bash
kubectl port-forward -n ingress-nginx service/ingress-nginx-controller 8080:80
curl -H "Host: go-api.local" http://localhost:8080
```

On Linux and EKS this works without port-forward.

### HPA
Autoscaler checks CPU every 15s. Current load ~1%, scales up when CPU > 50%.

### Namespace
Resources are isolated in `dev` namespace. To switch:
```bash
kubectl config set-context --current --namespace=dev
```
