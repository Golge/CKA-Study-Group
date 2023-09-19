**Name**: Fatih GUMUS

**Slack User ID**: U05SHFLUUNM

**Email**: fatihgumush@gmail.com

---

## Solution
creating deployment manifest files with labels for dev and prod (matchLabels)
### Solution Details

1. I generated 2 manifest files ```deployment-app.yaml``` and ```production-app.yaml``` Then on the terminal run those 2 commands ```kubectl create deploy -f deployment-app.yaml``` and ```kubectl create deploy -f production-app.yaml``` respectively
2. for monitoring logs ```kubectl logs -l app=prod-app```
### Code Snippet

```yaml
# Deployment Manifest File
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deployment-app
  labels:
    app: dev-app
    environment: development
spec:
  replicas: 1
  selector:
    matchLabels:
      app: deployment-app
  template:
    metadata:
      labels:
        app: deployment-app
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        resources:
          limits:
            memory: "128Mi"
            cpu: "500m"
        ports:
        - containerPort: 80
```

**Production Application (`production-app.yaml`):**
```yaml
# Production Manifest File
apiVersion: apps/v1
kind: Deployment
metadata:
  name: production-app
  labels:
    app: prod-app
    environment: production
spec:
  replicas: 3
  selector:
    matchLabels:
      app: prod-app
  template:
    metadata:
      labels:
        app: prod-app
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        resources:
          limits:
            memory: "128Mi"
            cpu: "500m"
        ports:
        - containerPort: 80
```