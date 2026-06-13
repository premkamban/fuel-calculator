
# Full Sample CI/CD App Blueprint

## Architecture

Frontend (Next.js) -> Backend (NestJS) -> PostgreSQL

CI/CD:
GitHub -> Jenkins -> Docker -> Local Registry -> Kubernetes -> Helm

---

# 1. Frontend (Next.js)

## frontend/package.json

```json
{
  "name": "frontend",
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "next start"
  },
  "dependencies": {
    "next": "14.2.0",
    "react": "18.2.0",
    "react-dom": "18.2.0"
  }
}
```

## frontend/pages/index.js

```javascript
export default function Home() {
  return (
    <div>
      <h1>Loan Eligibility Platform</h1>
    </div>
  );
}
```

## frontend/Dockerfile

```dockerfile
FROM node:20-alpine

WORKDIR /app

COPY package*.json ./

RUN npm install

COPY . .

RUN npm run build

EXPOSE 3000

CMD ["npm", "start"]
```

---

# 2. Backend (NestJS)

## backend/package.json

```json
{
  "name": "backend",
  "scripts": {
    "start": "nest start"
  },
  "dependencies": {
    "@nestjs/common": "^10.0.0",
    "@nestjs/core": "^10.0.0",
    "@nestjs/platform-express": "^10.0.0",
    "pg": "^8.11.0"
  }
}
```

## backend/src/main.ts

```typescript
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  app.enableCors();
  await app.listen(4000);
}

bootstrap();
```

## backend/src/app.controller.ts

```typescript
import { Controller, Get } from '@nestjs/common';

@Controller()
export class AppController {

  @Get()
  getHealth() {
    return {
      status: 'Backend Running'
    };
  }
}
```

## backend/src/app.module.ts

```typescript
import { Module } from '@nestjs/common';
import { AppController } from './app.controller';

@Module({
  controllers: [AppController]
})
export class AppModule {}
```

## backend/Dockerfile

```dockerfile
FROM node:20-alpine

WORKDIR /app

COPY package*.json ./

RUN npm install

COPY . .

EXPOSE 4000

CMD ["npm", "run", "start"]
```

---

# 3. Docker Compose

## docker-compose.yml

```yaml
version: '3'

services:

  frontend:
    build: ./frontend
    ports:
      - "3000:3000"

  backend:
    build: ./backend
    ports:
      - "4000:4000"

  postgres:
    image: postgres:15
    environment:
      POSTGRES_USER: admin
      POSTGRES_PASSWORD: admin
      POSTGRES_DB: loanapp
    ports:
      - "5432:5432"
```

Run:

```bash
docker-compose up --build
```

---

# 4. Jenkins Pipeline

## Jenkinsfile

```groovy
pipeline {

  agent any

  stages {

    stage('Frontend Build') {
      steps {
        dir('frontend') {
          sh 'docker build -t localhost:5000/frontend:v1 .'
        }
      }
    }

    stage('Backend Build') {
      steps {
        dir('backend') {
          sh 'docker build -t localhost:5000/backend:v1 .'
        }
      }
    }

    stage('Push Images') {
      steps {
        sh 'docker push localhost:5000/frontend:v1'
        sh 'docker push localhost:5000/backend:v1'
      }
    }

    stage('Deploy') {
      steps {
        sh 'kubectl apply -f k8s/'
      }
    }
  }
}
```

---

# 5. Kubernetes

## k8s/backend-deployment.yaml

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: backend

spec:
  replicas: 1

  selector:
    matchLabels:
      app: backend

  template:
    metadata:
      labels:
        app: backend

    spec:
      containers:
      - name: backend
        image: localhost:5000/backend:v1

        ports:
        - containerPort: 4000
```

## k8s/backend-service.yaml

```yaml
apiVersion: v1
kind: Service

metadata:
  name: backend-service

spec:
  selector:
    app: backend

  ports:
    - protocol: TCP
      port: 4000
      targetPort: 4000

  type: ClusterIP
```

## k8s/frontend-deployment.yaml

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: frontend

spec:
  replicas: 1

  selector:
    matchLabels:
      app: frontend

  template:
    metadata:
      labels:
        app: frontend

    spec:
      containers:
      - name: frontend
        image: localhost:5000/frontend:v1

        ports:
        - containerPort: 3000
```

## k8s/frontend-service.yaml

```yaml
apiVersion: v1
kind: Service

metadata:
  name: frontend-service

spec:
  selector:
    app: frontend

  ports:
    - protocol: TCP
      port: 3000
      targetPort: 3000

  type: LoadBalancer
```

Deploy:

```bash
kubectl apply -f k8s/
```

---

# 6. Helm

## Create Helm Chart

```bash
helm create loan-platform
```

## Important Files

### values.yaml

```yaml
image:
  repository: localhost:5000/backend
  tag: v1
```

### templates/deployment.yaml

Update image values from Helm variables.

Deploy:

```bash
helm install loan-app ./loan-platform
```

---

# 7. Local Docker Registry

Run registry:

```bash
docker run -d -p 5000:5000 registry:2
```

Verify:

```bash
curl http://localhost:5000/v2/_catalog
```

---

# 8. Local Kubernetes

Enable:
- Docker Desktop Kubernetes
OR
- Minikube

Commands:

```bash
kubectl get pods
kubectl get svc
kubectl logs <pod-name>
```

---

# Topics You MUST Practice After This

## Mandatory

### Docker
- Multi-stage builds
- Volumes
- Networking
- Optimization

### Kubernetes
- Pods
- Services
- Deployments
- Ingress
- ConfigMaps
- Secrets
- Autoscaling

### Helm
- values.yaml
- Templates
- Environment configs

### Jenkins
- Credentials
- GitHub webhooks
- Parallel stages

### AWS
- EKS
- ECS
- IAM
- VPC
- CloudFront
- Route53

### Performance
- SSR vs ISR
- CDN
- Redis caching
- Core Web Vitals

### Security
- JWT
- OAuth
- RBAC
- Secrets management

### System Design
- Kafka
- Queues
- Scaling
- High availability
- Replication

---

# Final Goal

Once this project works fully on your machine:

You will understand:
- CI/CD flow
- Deployment architecture
- Container orchestration
- Frontend/backend separation
- Registry lifecycle
- Infra basics

This is enough to become confident discussing enterprise deployment systems.
