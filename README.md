# Real-Time Chat Application


Welcome to the **Full Stack Realtime Chat App** project, a personal Kubernetes-ready chat application built to showcase a modern full-stack architecture with Docker and Kubernetes. This repository is maintained independently by Abhishek and is intended as a self-owned project for learning, deployment, and development.

## Table of Contents


* [Introduction](#introduction)
* [Features](#features)
* [Tech Stack](#tech-stack)
* [Getting Started](#getting-started)
* [Building the Backend](#building-the-backend)
* [Running the Application](#running-the-application)
* [Contributing](#contributing)
* [Future Plans](#future-plans)
* [License](#license)

## 📝 Introduction

This project aims to provide a real-time chat experience that's both scalable and secure. With a focus on modern technologies, we're building an application that's easy to use and maintain.

## ✨ Features


* **Real-time Messaging**: Send and receive messages instantly using Socket.io 
* **User Authentication & Authorization**: Securely manage user access with JWT 
* **Scalable & Secure Architecture**: Built to handle large volumes of traffic and data 
* **Modern UI Design**: A user-friendly interface crafted with React and TailwindCSS 
* **Profile Management**: Users can upload and update their profile pictures 
* **Online Status**: View real-time online/offline status of users 


## 🛠️ Tech Stack


* **Backend:** Node.js, Express, MongoDB, Socket.io
* **Frontend:** React, TailwindCSS
* **Containerization:** Docker
* **Orchestration:** Kubernetes
* **Ingress & Routing:** NGINX Ingress Controller
* **Web Server:** Nginx
* **State Management:** Zustand
* **Authentication:** JWT
* **Styling Components:** DaisyUI


### 🔧 Prerequisites


* **[Node.js](https://nodejs.org/)** (v14 or higher)
* **[Docker](https://www.docker.com/get-started)** (for containerizing the app)
* **[Git](https://git-scm.com/downloads)** (to clone the repository)
* **[kubectl](https://kubernetes.io/docs/tasks/tools/)** (to manage Kubernetes resources)
* **A Kubernetes cluster** (Minikube, Docker Desktop Kubernetes, or a cloud-managed cluster)


### 📝 Environment Configuration

Create a `.env` file in the root directory with the following configuration:

```env
# Database Configuration
MONGODB_URI=mongodb://root:admin@mongo:27017/chatApp?authSource=admin&retryWrites=true&w=majority

# JWT Configuration
JWT_SECRET=your_jwt_secret_key

# Server Configuration
PORT=5001
NODE_ENV=production
```

> **Note:** 
> - Replace `your_jwt_secret_key` with a strong secret key
> - For local development without Docker, change `MONGODB_URI` to `mongodb://localhost:27017/chatApp`
> - You can use command ```echo "Text what you want" | base64

### Clone the Repository

```bash
git clone https://github.com/iemafzalhassan/full-stack_chatApp.git
```

🏗️ Build and Run the Application

Follow these steps to build and run the application:

1. Build & Run the Containers:

```bash
cd full-stack_chatApp
```
```bash
docker-compose up -d --build
```

2. Access the application in your browser:

```
http://localhost
```
---

## 🛠️ Getting Started

Follow these simple steps to get the project up and running on your local Host using docker.

```bash
git clone https://github.com/iemafzalhassan/full-stack_chatApp.git
```

```bash
cd full-stack_chatApp
```
## Create a Docker network:

```bash
docker network create full-stack
```

## 🛠️ Building the Frontend

```bash
cd frontend
```

```bash
docker build -t full-stack_frontend .
```

### Run the Frontend container:

```bash
docker run -d --network=full-stack  -p 5173:5173 --name frontend full-stack_frontend:latest
```
#### The frontend will now be accessible on port 5173.


## Run the MongoDB Container:

```bash
docker run -d -p 27017:27017 --name mongo mongo:latest
```
---

## 🛠️ Building the Backend

```bash
cd backend
```

### Build the Backend image:

```bash
docker build -t full-stack_backend .
```

### Run the Backend container:

```bash
docker run -d --network=full-stack --add-host=host.docker.internal:host-gateway -p 5001:5001 --env-file .env full-stack_backend
```
#### This will build and run the backend container, exposing the backendAPI on port 5001.

`Backend API: http://localhost:5001`

### To Verify the conncetion between backend and databse:
```bash
docker-compose logs -f
```

### Once the backend and frontend containers are running, you can access the application in your browser:

`Frontend: http://localhost`


You can now interact with the real-time chat app and start messaging!

---

## ☸️ Kubernetes Deployment

This project includes Kubernetes manifests in the [k8s](k8s) folder so the application can run in a cluster instead of only through Docker Compose. Kubernetes helps organize the app into reusable resources such as Deployments, Services, Secrets, PersistentVolumeClaims, and an Ingress rule.

### What is deployed in Kubernetes?

- **Frontend Deployment:** Runs the React/Vite frontend and exposes it through a Service.
- **Backend Deployment:** Runs the Node.js/Express API and exposes it internally through a Service.
- **MongoDB Deployment:** Runs the database with persistent storage so chat data is retained.
- **Ingress:** Routes incoming traffic so requests to `/` go to the frontend and requests to `/api` go to the backend.
- **Secrets:** Stores sensitive values such as the JWT secret securely.

### Deploy to Kubernetes

If you have a Kubernetes cluster available, you can deploy the project with:

```bash
kubectl apply -f k8s/namespace.yml
kubectl apply -f k8s/secrets.yml
kubectl apply -f k8s/mongodb-pv.yml
kubectl apply -f k8s/mongodb-pvc.yml
kubectl apply -f k8s/mongodb-deployment.yml
kubectl apply -f k8s/mongodb-service.yml
kubectl apply -f k8s/backend-deployment.yml
kubectl apply -f k8s/backend-serive.yml
kubectl apply -f k8s/frontend-deployment.yml
kubectl apply -f k8s/frontend-service.yml
kubectl apply -f k8s/ingress.yml
```

### Verify the deployment

```bash
kubectl get pods,svc,ingress -n chat-app
```

> If you are using a local cluster, update the host value in [k8s/ingress.yml](k8s/ingress.yml) to your local domain or add it to your hosts file.

### Architecture Overview

The application is split into four main layers in Kubernetes:

```mermaid
flowchart LR
    User[User] --> Ingress[Ingress / NGINX]
    Ingress --> Frontend[Frontend Service]
    Ingress --> Backend[Backend Service]
    Frontend --> React[React App]
    Backend --> API[Express API]
    API --> DB[(MongoDB)]
```

- **User** accesses the application through the Ingress endpoint.
- **Frontend Service** serves the React UI.
- **Backend Service** handles authentication, chat logic, and API requests.
- **MongoDB** stores users, messages, and chat-related data.

---

### 🤝 Ownership and Maintenance

This repository is maintained as a personal project and is owned by Abhishek. Feel free to use it for learning, experimentation, or as a base for your own implementations.

If you want to suggest improvements, you can open an issue or contact the maintainer directly.

## 🔮 Future Plans


This project is evolving, and here are a few exciting things on the horizon:

* [x] **Kubernetes (K8s):** Add Kubernetes manifests for container orchestration and deployment.
* [ ] **CI/CD Pipelines:** Implement Continuous Integration and Continuous Deployment pipelines to automate testing and deployment.
* [ ] **Helm Charts:** Package the Kubernetes resources for easier environment-based deployment.
* [ ] **Feature Expansion:** Add more features like group chats, media sharing, and user status updates.
* **Stay tuned for updates as we continue to improve and expand this project!**

---

## 📚 Project Snapshots:

![Settings](frontend/public/settings.png)

![chat](frontend/public/chat.png)

![logout](/frontend/public/logout.png)

![Login](/frontend/public/login.png)



## 📜 License


This project is licensed under the MIT License. See the LICENSE file for more details.














