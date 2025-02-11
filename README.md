Go Web App - Docker & Kubernetes Setup

Overview

This project is a containerized Go web application that utilizes Docker for image creation and Kubernetes for orchestration. The application follows best practices by using a distroless image to minimize the attack surface and reduce image size. Helm is used to manage configurations and enable easy deployment across different environments.

Dockerization with Distroless

To containerize this Go application, we used Docker with a distroless base image. Distroless images contain only the necessary application runtime dependencies, making them:

Smaller in size compared to traditional OS-based images.

More secure due to the absence of unnecessary OS packages.

Faster to load in Kubernetes clusters.

# Dockerfile (Multi-Stage Build)

# First stage: Build the Go binary
FROM golang:1.22 as base

WORKDIR /app

COPY go.mod .

RUN go mod download 

COPY . .

RUN go build -o main .

# Second stage: Use distroless base image
#Distroles image Stage 2
FROM gcr.io/distroless/base

COPY --from=base /app/main .

COPY --from=base /app/static ./static

EXPOSE 8000

CMD ["./main"]

This approach reduces the final image size significantly while maintaining security.

Kubernetes Deployment

We use Kubernetes (K8s) to manage the application's deployment, ensuring scalability and high availability.

1. Deployment

A Deployment manages the pod lifecycle and ensures that the application runs efficiently.

apiVersion: apps/v1
kind: Deployment
metadata:
  name: go-web-app
  labels:
    app: go-web-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: go-web-app
  template:
    metadata:
      labels:
        app: go-web-app
    spec:
      containers:
      - name: go-web-app
        image: mkaran18/go-web-app 
        ports:
        - containerPort: 8080

2. Service

A Service exposes the application internally within the cluster.

apiVersion: v1
kind: Service
metadata:
  name: go-web-app
spec:
  selector:
    app: go-web-app
  ports:
  - name: go-web-app
    protocol: TCP
    port: 80
    targetPort: 8080
  type: ClusterIP

3. Ingress

An Ingress exposes the application externally using the Nginx Ingress Controller.

apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: go-web-app
spec:
  ingressClassName: nginx
  rules:
  - host: go-web-app.local
    http:
      paths:
      - pathType: Prefix
        path: /
        backend:
          service:
            name: go-web-app
            port:
              number: 80
  
![image](https://github.com/user-attachments/assets/9821a18b-4e44-46de-8010-2fd9cd20ac9b)

Helm for Configuration Management

Helm is used to parameterize configurations and simplify deployments.

Helm Chart Structure

/go-web-app-chart
├── charts/
├── templates/
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── ingress.yaml
│   └── _helpers.tpl
├── values.yaml
├── Chart.yaml

Helm Installation Command

helm install go-web-app ./go-web-app-chart

This allows easy customization of environment variables via values.yaml.

Conclusion

This project follows a best-practice containerization approach using:

Docker with a multi-stage build and distroless base image for security and efficiency.

Kubernetes for application orchestration using Deployments, Services, and Ingress.

Helm to parameterize and manage the deployment efficiently.

With this setup, the Go web application is scalable, lightweight, and production-ready.




