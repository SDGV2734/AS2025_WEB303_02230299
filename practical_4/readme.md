# Practical 4 Report — WEB303: Microservices & Serverless Applications

Student: Sonam Dorji

Module: WEB303 — Microservices & Serverless Applications

Date: 26 November 2025

Repository: https://github.com/Kinley-pal8/Web303_p4

Overview

This practical implements a cloud-native, containerised microservices system for a student cafe ordering application. The implementation uses Go for backend services, React for the frontend, Kubernetes for orchestration, and Kong as an API gateway. Key concerns addressed are service discovery, traffic routing, health checks, and autoscaling.

Learning outcomes

- Explain and apply Kubernetes deployment concepts and declarative manifests.
- Implement containerised microservices and deploy them to a Kubernetes cluster.
- Use Kubernetes DNS for service discovery between services.
- Configure Kong API Gateway to expose and route external traffic.
- Integrate a React frontend with backend services via the API gateway.
- Configure horizontal pod autoscaling and liveness/readiness probes.

Architecture

Services (with typical ports used during development):

- Food Catalog Service — Go, HTTP API, port 8080: provides menu items and health endpoints.
- Order Service — Go, HTTP API, port 8081: handles order creation and retrieval.
- Cafe UI — React single-page application, served on port 80.
- Kong API Gateway — routes external requests to the backend services (ports 80/443 externally, internal proxy to service ports).

Kubernetes components and patterns used

- Namespace isolation (namespace: student-cafe)
- Deployments and Services for each microservice
- Kong configured via Kubernetes ingress or custom manifests
- Readiness and liveness probes to ensure pod health
- Horizontal Pod Autoscaler (HPA) to scale services based on resource metrics
- Rolling updates to enable zero-downtime deployments

Implementation notes

- Manifests are written in declarative YAML (Deployments, Services, HPAs, and ingress configuration for Kong).
- Services communicate using Kubernetes DNS names (e.g., catalog-service.student-cafe.svc.cluster.local).
- The React frontend calls the Kong gateway which forwards requests to the appropriate backend service.

Quick start (local development with Minikube)

1. Start Minikube and configure Docker to build images inside the Minikube VM:

```bash
minikube start
eval $(minikube docker-env)
```

2. Create the project namespace:

```bash
kubectl create namespace student-cafe
```

3. Deploy the application manifests (example filenames used in this repo):

```bash
kubectl apply -f app-deployment.yaml -n student-cafe
kubectl apply -f kong-ingress.yaml -n student-cafe
```

4. Retrieve the external URL for the Kong proxy (Minikube provides a local access URL):

```bash
minikube service kong-kong-proxy -n student-cafe --url
```

Testing

Use the Kong proxy or direct service endpoints (for local testing) to exercise the APIs. Example calls:

```bash
# List menu items (catalog service)
curl http://localhost:8080/api/catalog/items

# Create an order (example payload)
curl -X POST http://localhost:8080/api/orders/orders \
  -H "Content-Type: application/json" \
  -d '{"item_ids": ["1","2"]}'

# Health check for catalog service
curl http://localhost:8080/api/catalog/health
```

Notes and verification checklist

- Confirm namespace exists: `kubectl get ns student-cafe`
- Confirm pods are running: `kubectl get pods -n student-cafe`
- Confirm Kong proxy URL and that routes are reachable via the proxy
- Verify HPA status: `kubectl get hpa -n student-cafe`

Conclusion

This practical demonstrates the deployment of a small microservices application using Kubernetes and Kong. It covers declarative manifests, service discovery, API gateway configuration, health probes, and autoscaling. The setup enables development and experimentation with cloud-native patterns for a student cafe ordering system.
