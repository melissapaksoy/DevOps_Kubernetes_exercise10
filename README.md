An **pritamworld/hellodocker:latest Deployment with 3 replicas** and a **NodePort Service** you can hit from local **Minikube**.

Note: Replace ```pritamworld/hellodocker:latest``` with your own docker hub image ```<dockerusername>/hellodocker:latest```

### 1) Manifest

Save as `hellodocker-deploy.yaml`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hellodocker
spec:
  replicas: 3
  selector:
    matchLabels:
      app: hellodocker
  template:
    metadata:
      labels:
        app: hellodocker
    spec:
      containers:
        - name: hellodocker
          image: pritamworld/hellodocker:latest
          ports:
            - containerPort: 80
          readinessProbe:
            httpGet:
              path: /
              port: 80
            initialDelaySeconds: 5
            periodSeconds: 5
          livenessProbe:
            httpGet:
              path: /
              port: 80
            initialDelaySeconds: 10
            periodSeconds: 10
```
Save as `hellodocker-service.yaml`:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: hellodocker-nodeport
spec:
  type: NodePort
  selector:
    app: hellodocker
  ports:
    - name: http
      port: 80
      targetPort: 80
      nodePort: 30080   # optional; pick any free port in 30000–32767
```

Apply & verify:

```bash
kubectl apply -f hellodocker-deploy.yaml
kubectl apply -f hellodocker-service.yaml
kubectl rollout status deployment/hellodocker
kubectl get pods -l app=hellodocker -o wide
kubectl get svc hellodocker-nodeport
```

Access it from Minikube:

```bash
# Easiest (prints a URL and opens a tunnel if needed):
minikube service hellodocker-nodeport --url

# Or hit the NodePort directly:
MINIKUBE_IP=$(minikube ip)
curl http://$MINIKUBE_IP:30080
```

---

### Cleanup

```bash
kubectl delete -f hellodocker-deploy.yaml
kubectl delete -f hellodocker-service.yaml
# or, if you used the one-liners:
kubectl delete svc hellodocker-nodeport
kubectl delete deploy hellodocker
```