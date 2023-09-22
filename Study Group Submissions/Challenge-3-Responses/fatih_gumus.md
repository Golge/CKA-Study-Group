**Name**: Fatih GUMUS

**Slack User ID**: U05SHFLUUNM

**Email**: fatihgumush@gmail.com

---

## Solution
creating deployment manifest files with labels for dev and prod (matchLabels)
### Solution Details

1. I generated 3 manifest files ```nginx-conf.yaml```, ```nginx-proxy-deploy.yaml```, and ``` nginx-svc.yaml```
Then on the terminal run ```kubectl apply -f .``` 

3. 

### Code Snippet for ConfigMap
To create the configuration settings on the manifest file, i used the given code snippet. On the data block i used ```default.conf``` as it is the default configuration file for nginx. Then i used | for parsing multiline string.
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-conf # <- volume // configMap pair ->
data:
  default.conf: |
    server {
      listen 80;
      server_name localhost;
      location / {
          proxy_set_header   X-Forwarded-For $remote_addr;
          proxy_set_header   Host $http_host;
          proxy_pass         "http://127.0.0.1:8080";
      }
    }
```
### Code Snippet for Service object
The service type must be ```ClusterIP``` and on labels attribute the given key-value pair must match with the Deployment object's template label.
```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-proxy-deployment
  labels:
    app: nginx-proxy # <- deploy.template.label ->
spec:
  selector:
    app: nginx-proxy
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
  type: ClusterIP
```
### Code Snippet for Deployment object
Deployment object's template field contains label as ```app: nginx-proxy``` This value must match with the Service's metadata labels section.
To define ConfigMap and inject it to the nginx-proxy pod, i used volumes and volumeMounts.
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-proxy-deployment
  labels:
    app: nginx-proxy
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx-proxy
  template:
    metadata:
      labels:
        app: nginx-proxy # <- svc.label.selector ->
    spec:
      volumes:
        - name: nginx-conf # <- volume // volumeMount pair ->
          configMap:
            name: nginx-conf # <- volume // configMap pair ->
            items:
              - key: default.conf
                path: default.conf
      containers:
        - name: nginx-proxy
          image: nginx
          ports:
            - containerPort: 80
              protocol: TCP
          volumeMounts:
            - name: nginx-conf # <- volume // volumeMount pair ->
              readOnly: true
              mountPath: /etc/nginx/conf.d/default.conf
              subPath: default.conf
          resources:
            limits:
              memory: 128Mi
              cpu: 500m
        - name: simple-webapp
          image: kodekloud/simple-webapp
          ports:
            - containerPort: 8080
              protocol: TCP
          resources:
            limits:
              memory: 128Mi
              cpu: 500m
```