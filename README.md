# Kubernetes Multi-Tier Web Application

## Prerequisites
- minikube installed
- kubectl installed
- minikube cluster running (`minikube start`)

## Deploy the stack

bash
kubectl apply -f manifests/namespace.yaml
kubectl apply -f manifests/

## Verify deployment

kubectl get all -n webapp

Wait until all pods are in `Running` state.

## Access the application

minikube service webapp-svc -n webapp


This opens the web app in your browser. Screenshot of the running app is in `screenshot.png`.

## Tear down

kubectl delete namespace webapp





Written Questions



Q1: Why is a StatefulSet used for PostgreSQL instead of a Deployment? What would break if you used a Deployment?

StatefulSets give pods a stable network identity and ensure each pod binds to the same PersistentVolume across restarts. A Deployment creates pods with random names and no guaranteed storage attachment — if the pod restarts, it would come back with an empty disk, losing all database data. Additionally, StatefulSets guarantee ordered startup and shutdown, which databases depend on.

-----------------------------------------------------------------------------------------------

Q2: What is the difference between a ConfigMap and a Secret in Kubernetes? Why should
credentials not be stored in a ConfigMap?

onfigMaps store non-sensitive configuration data in plain text. Secrets are designed specifically for sensitive data (like passwords) and are base64 encoded. Credentials should not be stored in ConfigMaps because they are visible in plain text to anyone with read access to the cluster, creating a security vulnerability.

-----------------------------------------------------------------------------------------------

Q3: Your web app Deployment has 2 replicas. A customer reports that after running kubectl apply with a new image, they briefly saw errors. What Kubernetes settings could prevent this?

Use a RollingUpdate with maxUnavailable: 0 and a proper readiness probe. This ensures new pods are ready before old pods are removed, preventing request failures during deployment.

-----------------------------------------------------------------------------------------------

Q4: Explain what happens at the network level when the web app pod connects to
postgres-svc:5432. How does Kubernetes resolve that hostname?

Kubernetes runs an internal DNS server (CoreDNS). When a webapp pod connects to postgres-svc:5432, CoreDNS resolves postgres-svc to the ClusterIP assigned to that Service. The packet is then intercepted by kube-proxy (via iptables rules) on the node, which forwards it to one of the healthy postgres pod IPs on port 5432. The full DNS name would be postgres-svc.webapp.svc.cluster.local.

-----------------------------------------------------------------------------------------------

Q5: A pod is stuck in CrashLoopBackOff. Walk through the kubectl commands you would run to
diagnose the issue.

# 1. See which pod is crashing and its restart count
kubectl get pods -n webapp

# 2. Read the current logs
kubectl logs <pod-name> -n webapp

# 3. Read logs from the previous crashed container
kubectl logs <pod-name> -n webapp --previous

# 4. Get detailed events and state (look at "Events" section at the bottom)
kubectl describe pod <pod-name> -n webapp

# 5. If the container starts briefly, exec into it to investigate
kubectl exec -it <pod-name> -n webapp -- /bin/sh


Common causes: wrong environment variable, missing Secret/ConfigMap, image that doesn't exist, or a failing liveness probe that keeps killing the container."# kubernetis-home-assignment" 
