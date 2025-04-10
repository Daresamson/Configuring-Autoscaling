
# Kubernetes Web Application Deployment with Autoscaling, Load Balancing, and Health Probes

## Project Overview
In this project, I deployed a high-traffic web application on a Kubernetes cluster, configured autoscaling, load balancing, and health probes to ensure high availability and performance during peak traffic periods. The deployment was done on AWS EKS, and I faced several challenges along the way that required troubleshooting and adjustments.

## Tools Used
- **Kubernetes (EKS)**: For managing the containerized web application.
- **kubectl**: Command-line tool for interacting with the Kubernetes cluster.
- **Minikube**: For local Kubernetes cluster simulation (for testing).
- **NGINX**: As the web server for the application.
- **Horizontal Pod Autoscaler (HPA)**: To scale pods based on CPU usage.
- **LoadBalancer Service**: To distribute traffic across application pods.
- **Apache Benchmark (ab)**: For load testing the web application.

---

## Setup Instructions

### Step 1: Set Up the Kubernetes Cluster
I started by setting up an AWS EKS cluster and verified the setup using the following command:

```bash
kubectl get nodes
```
![Screenshot (202)](https://github.com/user-attachments/assets/8e5c2f2e-e605-4267-a292-59dff39c731d)

### Step 2: Deploy the Web Application
1. **Create a deployment.yaml file**:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web-app
  template:
    metadata:
      labels:
        app: web-app
    spec:
      containers:
      - name: web-app
        image: nginx:latest
        ports:
        - containerPort: 80
        readinessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 10
        livenessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 15
          periodSeconds: 20
```

2. I applied the deployment:

```bash
kubectl apply -f deployment.yaml
kubectl get pods
```
![Screenshot (203)](https://github.com/user-attachments/assets/b61d661b-6572-4875-87a8-d9a6364864f4)
![Screenshot (204)](https://github.com/user-attachments/assets/22a57e24-8838-43ec-8c35-fa064b3830b5)

### Step 3: Configure Horizontal Pod Autoscaler (HPA)
1. **Create an hpa.yaml file**:

```yaml
apiVersion: autoscaling/v2beta2
kind: HorizontalPodAutoscaler
metadata:
  name: web-app-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: web-app
  minReplicas: 3
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 50
```

2. I applied the HPA configuration:

```bash
kubectl apply -f hpa.yaml
kubectl get hpa
```
![Screenshot (205)](https://github.com/user-attachments/assets/b39bb6bf-2424-4f7d-9bba-818031d5d854)
![Screenshot (206)](https://github.com/user-attachments/assets/04cab64e-4181-4e39-82ce-5c5bab42f390)

### Step 4: Configure Load Balancing
1. **Create a service.yaml file**:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-app-service
spec:
  selector:
    app: web-app
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
  type: LoadBalancer
```

2. I applied the service:

```bash
kubectl apply -f service.yaml
kubectl get svc
```
![Screenshot (208)](https://github.com/user-attachments/assets/f31a0dd6-293a-43c8-b50f-a29df1477e12)
![Screenshot (209)](https://github.com/user-attachments/assets/fa7b1033-f02f-4d89-857c-0d4d4dcc2163)

### Step 5: Test the Setup
To test the setup and simulate traffic, I used Apache Benchmark:

1. **Port-Forwarding for Local Testing (Minikube)**:
   I ran the following command to get the service URL:

```bash
minikube service web-app-service --url
```

This returned a URL like `http://192.168.49.2:31887`.

2. **Run Load Test**:

```bash
ab -n 1000 -c 100 http://192.168.49.2:31887/
```

---
![Screenshot (210)](https://github.com/user-attachments/assets/18cd59c3-807d-42a9-a161-c75075c02274)
![Screenshot (211)](https://github.com/user-attachments/assets/f03c1da8-afa7-423f-85f8-07752066f79e)
![Screenshot (212)](https://github.com/user-attachments/assets/afe352b1-69bd-48ae-b477-12e9071fc129)


## ✅ Challenges Faced and Final Solutions

### **Challenge 1: External Access to the Web App**

**Problem**:  
The application was accessible only from within the EC2 instance when using `kubectl port-forward`. Direct access via public IP did not work, even with `--address 0.0.0.0`.

**Solution**:
I used **SSH tunneling** in combination with `kubectl port-forward` for seamless local access:

1. **SSH Port Forwarding from Local Machine**:
   ```bash
   ssh -i funmi.pem -L 8080:localhost:8080 ubuntu@52.91.104.233
   ```

2. **Inside the EC2 Instance**, forward the Kubernetes service:
   ```bash
   kubectl port-forward service/web-app-service 8080:80
   ```

3. **Access the app from local machine**:
   ```bash
   curl http://localhost:8080
   ```
   ![Screenshot (213)](https://github.com/user-attachments/assets/ad9d0ad2-1c37-4075-9561-65d4cd2f4c05)
   ![Screenshot (214)](https://github.com/user-attachments/assets/1330e191-bdc0-4590-87b0-c454302b1d31)


This approach securely exposed the service to my local environment using `localhost`, solving the issue without needing to alter cluster networking or public firewall rules.

---

### **Challenge 2: Frequent Pod Restarts from Liveness Probe Failures**

**Problem**:  
Pods restarted repeatedly due to failing the liveness probe, likely because the NGINX container hadn't fully started by the time the probe was triggered.

**Solution**:
I tuned the **liveness probe settings** by increasing `initialDelaySeconds` and `periodSeconds`:

```yaml
livenessProbe:
  httpGet:
    path: /
    port: 80
  initialDelaySeconds: 15  # Increased from default
  periodSeconds: 20        # Gave more room between checks
```

This allowed the pods enough time to start and stabilize before being probed, eliminating unnecessary restarts and improving overall pod lifecycle management.

---

## ✅ Conclusion

This project solidified my understanding of deploying scalable and resilient applications on Kubernetes. By configuring **autoscaling, load balancing**, and **health checks**, and overcoming real-world challenges like **port-forwarding and probe tuning**, I gained valuable hands-on experience with production-grade Kubernetes workflows on AWS EKS.

---
