# 1. Build and push Docker images:
# Build frontend
cd frontend
docker build -t your-registry/k8s-demo-frontend:latest .
docker push your-registry/k8s-demo-frontend:latest

# Build API
cd ../api
docker build -t your-registry/k8s-demo-api:latest .
docker push your-registry/k8s-demo-api:latest

# 2. Deploy to Kubernetes:

# Create namespace
kubectl apply -f k8s/namespace.yaml

# Deploy database
kubectl apply -f k8s/database/

# Deploy API
kubectl apply -f k8s/api/

# Deploy frontend
kubectl apply -f k8s/frontend/
# Step 1: Create namespace first
kubectl apply -f k8s/namespace.yaml

# Step 2: Create ConfigMap and Secrets (must be created before deployments)
kubectl apply -f k8s/database/postgres-configmap.yaml
kubectl apply -f k8s/database/postgres-secret.yaml

# Step 3: Deploy database
kubectl apply -f k8s/database/postgres-deployment.yaml
kubectl apply -f k8s/database/postgres-service.yaml

# Step 4: Wait for database to be ready
kubectl wait --for=condition=available --timeout=300s deployment/postgres -n demo-app

# Step 5: Deploy API (depends on database)
kubectl apply -f k8s/api/api-deployment.yaml
kubectl apply -f k8s/api/api-service.yaml

# Step 6: Wait for API to be ready
kubectl wait --for=condition=available --timeout=300s deployment/api -n demo-app

# Step 7: Deploy frontend
kubectl apply -f k8s/frontend/frontend-deployment.yaml
kubectl apply -f k8s/frontend/frontend-service.yaml

# 3. Check deployment status:
# Check if ConfigMap and Secret were created
kubectl get configmap -n demo-app
kubectl get secret -n demo-app

# Check deployment status
kubectl get pods -n demo-app
kubectl get services -n demo-app

# Check logs if needed
kubectl logs -l app=postgres -n demo-app
kubectl logs -l app=api -n demo-app
