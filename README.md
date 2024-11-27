# Environment-Configuration-Lab1

# 1 Create dev and prod environment and configurations
  # 1.1 Define the development and production environments using namespaces.
  kubectl create namespace production
  # 1. 2 Configure Environment-Specific ConfigMaps
  kubectl create configmap app-config \
    --from-literal=APP_ENV=production \
    --namespace=production

# 2 Define sensitive information (like database passwords) as Secrets
kubectl create secret generic app-secret \
  --from-literal=DB_PASSWORD=prodpassword123 \
  --namespace=production

# 3 View all secrets per environment
kubectl get secrets -n production

# 4 View the metadata of the secret
kubectl describe secret app-secret -n production

# 5 Decode secret
kubectl get secret app-secret -n production -o jsonpath="{.data.DB_PASSWORD}" | base64 --decode

# 6 Create a generic deployment YAML template named app-deployment-template.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-app
  namespace: PLACEHOLDER_NAMESPACE
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx-app
  template:
    metadata:
      labels:
        app: nginx-app
    spec:
      containers:
      - name: nginx
        image: nginx
        env:
        - name: APP_ENV
          valueFrom:
            configMapKeyRef:
              name: app-config
              key: APP_ENV
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: app-secret
              key: DB_PASSWORD

# 7 Replace the generic yaml template and deploy + Set Replica to 3
sed -e 's/PLACEHOLDER_NAMESPACE/production/‘ \
    -e 's/replicas: .*$/replicas: 3/‘ app-deployment-template.yaml > production-deployment.yaml

kubectl apply -f production-deployment.yaml

# 8 Expose the production environment
kubectl expose deployment nginx-app \
  --type=NodePort \
  --name=nginx-service \
  --port=80 \
  --target-port=80 \
  --namespace=production

# 9 Verify the deployment and access
kubectl get pods -n production
kubectl get service -n production

kubectl exec -it nginx-app-65968468b6-46pgj -n production -- /bin/bash
echo $APP_ENV
echo $DB_PASSWORD
