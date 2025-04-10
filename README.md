# Configuring-Autoscaling
Here’s the updated README file, personalized with your experience using "I":

---

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

## Challenges Faced and Solutions

### Challenge 1: **Port-Forwarding Issue**
When testing the service, I encountered an issue where it was only accessible locally within the EC2 instance. To make the service accessible externally, I explored two solutions:

1. **SSH Tunnel**:
   I used this command to forward the port from my local machine to the EC2 instance:

   ```bash
   ssh -i funmi.pem -L 8080:localhost:8080 ubuntu@52.91.104.233
   ```

   This allowed me to access the service via `http://localhost:8080` from my local machine.

2. **Bind to 0.0.0.0**:
   I ran the following command to bind the port to all interfaces, allowing access via the EC2 public IP:

   ```bash
   kubectl port-forward --address 0.0.0.0 service/web-app-service 8080:80
   ```

   This made the service available at `http://52.91.104.233:8080`.

---

### Challenge 2: **Pod Restarts Due to Liveness Probe Failures**
I noticed that the pods were restarting frequently due to liveness probe failures. After troubleshooting, I found that the pod was not being given enough time to start before the liveness probe checked its health. To fix this, I:

1. Adjusted the `initialDelaySeconds` and `periodSeconds` in the liveness probe configuration to better match the pod startup time.
2. This helped to prevent unnecessary restarts, ensuring a smoother pod lifecycle.

---

## Testing and Verification

1. **Load Testing with Apache Benchmark**:
   I simulated traffic using Apache Benchmark and monitored the scaling behavior:

```bash
ab -n 1000 -c 100 http://52.91.104.233:8080/
```

2. **Monitoring**:
   I observed the scaling of pods with the following commands:

```bash
kubectl get hpa
kubectl get pods
```

---

## Conclusion
By following the steps outlined in this guide, I successfully deployed a high-traffic web application on Kubernetes, configured autoscaling, load balancing, and health probes to ensure high availability and optimal performance. Through troubleshooting port-forwarding issues and adjusting the liveness probe configuration, I overcame key challenges and gained valuable hands-on experience with Kubernetes.

--- 
![Screenshot (184)](https://github.com/user-attachments/assets/56eb1acd-d1b5-43e6-a118-2eec9267fe72)
![Screenshot (185)](https://github.com/user-attachments/assets/62b1f10f-8d09-4df6-a062-325a28e564eb)
![Screenshot (189)](https://github.com/user-attachments/assets/43ec0c43-530e-4816-a91a-4c790289fc09)
![Screenshot (188)](https://github.com/user-attachments/assets/689ee302-08a4-4cb7-a769-e7b4afd2ae96)
![Screenshot (189)](https://github.com/user-attachments/assets/a8d61cb5-e5e9-45b6-8cb4-bb7573d2781d)
![Screenshot (190)](https://github.com/user-attachments/assets/d019b1b4-221a-4b6f-a55e-2b7979feb7ef)
![Screenshot (191)](https://github.com/user-attachments/assets/0c8c14d8-e790-49f9-8710-8cae60c1217b)
![Screenshot (192)](https://github.com/user-attachments/assets/d4e51dc0-de12-49d4-b559-159356c1dcd0)
![Screenshot (193)](https://github.com/user-attachments/assets/ff5a7a68-4dcb-464a-9d57-e0cb1ea3b1c9)
![Screenshot (197)](https://github.com/user-attachments/assets/7bfb9adc-1cdd-49f4-b368-bebf75dac58f)
![Screenshot (198)](https://github.com/user-attachments/assets/3daa9d59-c59e-4536-975e-52ca6eced013)

