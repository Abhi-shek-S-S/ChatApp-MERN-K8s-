# Production Deployment Guide

This file explains how to deploy your Realtime Chat App in production without using `kubectl port-forward`. In production, traffic is routed through Kubernetes Services and an Ingress controller instead of manual port forwarding.

## Why port-forwarding is only for debugging

`kubectl port-forward` is useful for:
- debugging pod behavior
- temporarily accessing an internal service from your workstation
- testing a deployment before exposing it externally

It is not suitable for production because it:
- only forwards traffic from a single local machine
- is not resilient or scalable
- requires a user to keep the command running

## Production architecture overview

In production, this app should use:
- Deployments for frontend, backend, and MongoDB
- Services to expose pods inside the cluster
- Ingress to expose the frontend and backend to external users
- DNS to map a real domain to the Ingress controller
- TLS for secure HTTPS traffic

Your project already includes the following key manifests:
- `namespace.yml`
- `secrets.yml`
- `frontend-deployment.yml`
- `frontend-service.yml`
- `backend-deployment.yml`
- `backend-serive.yml`
- `mongodb-deployment.yml`
- `mongodb-service.yml`
- `mongodb-pv.yml`
- `mongodb-pvc.yml`
- `ingress.yml`

## Deploying to a production-ready cluster

### 1. Install or enable an Ingress controller

Your `ingress.yml` uses the NGINX Ingress syntax, so install an NGINX Ingress controller in the cluster if it is not already present.

Example for a generic cluster:
```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/cloud/deploy.yaml
```

### 2. Apply namespace and secrets

```bash
kubectl apply -f k8s/namespace.yml
kubectl apply -f k8s/secrets.yml
```

### 3. Apply MongoDB storage and database resources

```bash
kubectl apply -f k8s/mongodb-pv.yml
kubectl apply -f k8s/mongodb-pvc.yml
kubectl apply -f k8s/mongodb-deployment.yml
kubectl apply -f k8s/mongodb-service.yml
```

### 4. Deploy backend and frontend

```bash
kubectl apply -f k8s/backend-deployment.yml
kubectl apply -f k8s/backend-serive.yml
kubectl apply -f k8s/frontend-deployment.yml
kubectl apply -f k8s/frontend-service.yml
```

### 5. Apply the Ingress rule

```bash
kubectl apply -f k8s/ingress.yml
```

## How external traffic reaches your app

Your `ingress.yml` defines a rule for host `chat-tws.com`:

```yaml
rules:
- host: chat-tws.com
  http:
    paths:
    - path: "/"
      backend:
        service:
          name: frontend
          port:
            number: 80
    - path: "/api"
      backend:
        service:
          name: backend
          port:
            number: 5001
```

In production:
- assign `chat-tws.com` to the external IP of the Ingress controller
- use a real DNS A/AAAA record or CNAME record
- prefer HTTPS with a certificate manager such as cert-manager

## Example: production DNS + Ingress setup

1. Find the external IP of the Ingress controller service:

```bash
kubectl get svc -n ingress-nginx
```

2. Create a DNS record for your domain:
- `chat-tws.com` → `<INGRESS_EXTERNAL_IP>`

3. Verify the host resolves:

```bash
nslookup chat-tws.com
```

4. Access the app in a browser:
- `http://chat-tws.com`
- backend requests from the frontend will use `/api`

## Example: using a LoadBalancer service instead of port-forwarding

If your cloud provider supports LoadBalancer services, you can expose the frontend directly with a Service of type `LoadBalancer`.

Example Service manifest:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: frontend-lb
  namespace: chat-app
spec:
  type: LoadBalancer
  selector:
    app: frontend
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
```

After applying it, the cloud provider will allocate an external IP for `frontend-lb`.

## Production best practices

- Use a real domain name instead of localhost
- Use `type: LoadBalancer` or `Ingress` for external exposure
- Configure HTTPS certificates with cert-manager or your cloud provider
- Do not rely on `kubectl port-forward` for real users
- Use managed Kubernetes services for reliability in production (EKS, AKS, GKE, DigitalOcean, etc.)
- Store sensitive values in Kubernetes Secrets, not in plain text
- Monitor pods with `kubectl get pods -n chat-app` and logs with `kubectl logs`

## Common production commands

Check resources:
```bash
kubectl get all -n chat-app
kubectl describe ingress chatapp-ingress -n chat-app
kubectl logs -l app=backend -n chat-app
```

Inspect service endpoints:
```bash
kubectl get svc -n chat-app
kubectl get endpoints -n chat-app
```

## Summary

- `kubectl port-forward` is for local debugging only.
- In production, expose the app through Services and Ingress.
- Use DNS + ingress external IP for the public hostname.
- Use TLS and cloud-native LoadBalancer/Ingress support for real traffic.

This guide is based on your current Kubernetes manifests and shows how to move from port-forwarding to a production deployment.
