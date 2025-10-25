# ☸️ Kubernetes kubectl Cheatsheet

[⬅️ Back to Root](../../README.md)

## Core Concepts
- Cluster, Node, Pod, ReplicaSet, Deployment
- Service (ClusterIP, NodePort, LoadBalancer), Ingress
- Namespace, Context, Kubeconfig
- ConfigMap, Secret, ServiceAccount

## Context and Namespace
```bash
kubectl config get-contexts
kubectl config use-context myctx
kubectl config set-context --current --namespace dev
kubectl get ns
```

## CRUD Basics
```bash
# List
kubectl get pods,deploy,svc,cm,secret -A
kubectl get po -n dev -o wide

# Describe/Logs/Exec
kubectl describe po mypod -n dev
kubectl logs -f deploy/myapp -n dev
kubectl exec -it deploy/myapp -- sh

# Create/Apply/Delete
kubectl apply -f k8s/
kubectl delete -f k8s/
```

## Pods and Deployments
```bash
kubectl run nginx --image=nginx:alpine -n dev
kubectl create deploy web --image=nginx:alpine -n dev
kubectl scale deploy/web --replicas=3 -n dev
kubectl set image deploy/web nginx=nginx:1.27 -n dev
kubectl rollout status deploy/web -n dev
kubectl rollout undo deploy/web -n dev
```

## Services and Ingress
```bash
kubectl expose deploy/web --port 80 --target-port 80 --type ClusterIP -n dev
kubectl get svc -n dev
# Ingress usually via manifest
```

## ConfigMaps and Secrets
```bash
kubectl create configmap app-config --from-literal=MODE=prod -n dev
kubectl create secret generic app-secret --from-literal=API_KEY=abcd -n dev
kubectl get cm,secret -n dev -o yaml
```

## Context Shortcuts
- -n: namespace
- -o json|yaml|wide|name
- -l key=value: label selector
- --field-selector key=value

## Useful Examples
```yaml
apiVersion: apps/v1
kind: Deployment
metadata: { name: web, namespace: dev }
spec:
  replicas: 3
  selector: { matchLabels: { app: web } }
  template:
    metadata: { labels: { app: web } }
    spec:
      containers:
      - name: nginx
        image: nginx:1.27
        ports: [{ containerPort: 80 }]
```

## Best Practices
- Use namespaces per env/team
- Version manifests with GitOps
- Use readiness/liveness probes
- Request/limit resources
- RBAC: least privilege
