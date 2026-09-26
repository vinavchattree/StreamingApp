# StreamingApp

Stream premium video content, host live watch parties, and manage your catalogue with a modern microservice architecture. The platform now ships with a production-ready admin portal, real-time chat, S3-backed adaptive streaming, and a redesigned cinematic frontend experience.

## Architecture

| Service | Port | Description |
| --- | --- | --- |
| `authService` | 3001 | User authentication, registration, JWT issuance |
| `streamingService` | 3002 | Video catalogue, S3 playback endpoints, public APIs |
| `adminService` | 3003 | Dedicated admin microservice for asset management and uploads |
| `chatService` | 3004 | Websocket + REST chat for live watch parties |
| `frontend` | 3000 | React SPA with revamped UI and integrated chat |
| `mongo` | 27017 | Shared MongoDB instance |

All backend services share common database models and utilities through `backend/common`.

## Environment Configuration

Create an `.env` for each service (or export variables before running). All services accept the standard AWS credentials for S3 access.

### Auth Service (`backend/authService/.env`)
```ini
PORT=3001
MONGO_URI=mongodb://localhost:27017/streamingapp
JWT_SECRET=changeme
CLIENT_URLS=http://localhost:3000
AWS_ACCESS_KEY_ID=
AWS_SECRET_ACCESS_KEY=
AWS_REGION=ap-south-1
AWS_S3_BUCKET=
```

### Streaming Service (`backend/streamingService/.env`)
```ini
PORT=3002
MONGO_URI=mongodb://localhost:27017/streamingapp
JWT_SECRET=changeme
CLIENT_URLS=http://localhost:3000
AWS_ACCESS_KEY_ID=
AWS_SECRET_ACCESS_KEY=
AWS_REGION=ap-south-1
AWS_S3_BUCKET=
AWS_CDN_URL=
STREAMING_PUBLIC_URL=http://localhost:3002
```

### Admin Service (`backend/adminService/.env`)
```ini
PORT=3003
MONGO_URI=mongodb://localhost:27017/streamingapp
JWT_SECRET=changeme
CLIENT_URLS=http://localhost:3000
AWS_ACCESS_KEY_ID=
AWS_SECRET_ACCESS_KEY=
AWS_REGION=ap-south-1
AWS_S3_BUCKET=
```

### Chat Service (`backend/chatService/.env`)
```ini
PORT=3004
MONGO_URI=mongodb://localhost:27017/streamingapp
JWT_SECRET=changeme
CLIENT_URLS=http://localhost:3000
```

### Frontend build variables (`frontend/.env` or Docker build args)
```ini
REACT_APP_AUTH_API_URL=http://localhost:3001/api
REACT_APP_STREAMING_API_URL=http://localhost:3002/api
REACT_APP_STREAMING_PUBLIC_URL=http://localhost:3002
REACT_APP_ADMIN_API_URL=http://localhost:3003/api/admin
REACT_APP_CHAT_API_URL=http://localhost:3004/api/chat
REACT_APP_CHAT_SOCKET_URL=http://localhost:3004
```

## Running with Docker Compose

1. Populate the environment variables above (or rely on the defaults baked into `docker-compose.yml`).
2. Build and start the stack:
   ```bash
   docker-compose up --build
   ```
3. Navigate to `http://localhost:3000` for the web app.

The compose file provisions MongoDB plus all four Node.js microservices. S3 credentials are optional for local testing—you can still browse seeded metadata, but streaming requires valid S3 objects.

## Local Development

Install dependencies for each service:

```bash
# auth service
cd backend/authService && npm install

# streaming service
cd ../streamingService && npm install

# admin service
cd ../adminService && npm install

# chat service
cd ../chatService && npm install

# frontend
cd ../../frontend && npm install
```

Run the services (in separate terminals) after starting MongoDB:

```bash
cd backend/authService && npm run dev
cd backend/streamingService && npm run dev
cd backend/adminService && npm run dev
cd backend/chatService && npm run dev
cd frontend && npm start
```

## Feature Highlights

- **S3-backed adaptive streaming** with secure signed uploads for admins.
- **Dedicated admin microservice** for video ingestion, metadata management, and featured curation.
- **Real-time chat** overlay in the player (Socket.IO + persistent message history).
- **Modern React experience** featuring cinematic hero sections, dynamic carousels, and responsive design.
- **Role-aware access control** across frontend routes and backend microservices.

## Testing

Automated tests are not yet included. Recommended smoke checks:

1. Register and log in through the web UI.
2. Upload a small video + thumbnail via the admin dashboard (requires valid S3 credentials).
3. Confirm playback from the browse page and verify that chat messages broadcast between multiple browser tabs.

## License

MIT © StreamFlix Team

# EKS Deployment

## Architecture

Amazon EKS & Helm.

Components:

- authService —  3001
- streamingService —  3002
- adminService —  3003
- chatService — 3004
- frontend —  80
- MongoDB 27017


Helm :

The Helm chart includes:
•	Auth service
•	Streaming service
•	Admin service
•	Chat service
•	Frontend
•	MongoDB StatefulSet
•	ConfigMap and Secret
•	MongoDB persistent storage
•	AWS ALB Ingress
•	WebSocket routing for Socket.IO
•	Kubernetes service accounts
•	Health checks and resource limits



2. Create the Helm Chart Directory
From the StreamingApp project directory:
cd /home/ec2-user/streaming-app/StreamingApp
Create the Helm chart structure:
mkdir -p helm/streamingapp/templates
cd helm/streamingapp



3. Create Chart.yaml
Create the Helm chart definition:
cat > Chart.yaml <<'EOF'
apiVersion: v2
name: streamingapp
description: Helm chart for StreamingApp MERN platform
type: application
version: 1.0.0
appVersion: "1.0.0"
EOF



4.Create values.yaml
values.yaml contains the configurable values used by the Kubernetes templates.
cat > values.yaml <<'EOF'
namespace: streamingapp

registry: 691317217805.dkr.ecr.ap-south-1.amazonaws.com

images:
  auth:
    repository: streaming-auth
    tag: "1.0.0"
  streaming:
    repository: streaming-service
    tag: "1.0.0"
  admin:
    repository: streaming-admin
    tag: "1.0.0"
  chat:
    repository: streaming-chat
    tag: "1.0.0"
  frontend:
    repository: streaming-frontend
    tag: "1.0.1"

replicas:
  auth: 2
  streaming: 2
  admin: 1
  chat: 2
  frontend: 2

mongo:
  image: mongo:6
  storage: 5Gi
  database: streamingapp

config:
  mongoUri: mongodb://mongo:27017/streamingapp
  awsRegion: ap-south-1
  awsS3Bucket: test-bucket-vin2
  clientUrls: http://localhost

secret:
  jwtSecret: streamingapp-k8s-secret

serviceAccounts:
  admin: admin-service-account

resources:
  requests:
         cpu: 100m
    memory: 128Mi
  limits:
    cpu: 500m
    memory: 512Mi

mongoResources:
  requests:
    cpu: 100m
    memory: 256Mi
  limits:
    cpu: 500m
    memory: 512Mi

ingress:
  enabled: true
  scheme: internet-facing
  targetType: ip
EOF


5. ConfigMap
   cat > templates/configmap.yaml <<'EOF'
apiVersion: v1
kind: ConfigMap
metadata:
  name: streamingapp-config
data:
  MONGO_URI: {{ .Values.config.mongoUri | quote }}
  AWS_REGION: {{ .Values.config.awsRegion | quote }}
  AWS_S3_BUCKET: {{ .Values.config.awsS3Bucket | quote }}
  CLIENT_URLS: {{ .Values.config.clientUrls | quote }}
EOF


6. Create secret
   cat > templates/secret.yaml <<'EOF'
apiVersion: v1
kind: Secret
metadata:
  name: streamingapp-secret
type: Opaque
stringData:
  JWT_SECRET: {{ .Values.secret.jwtSecret | quote }}
EOF



7. The application is exposed externally through an AWS Application Load Balancer (ALB).
alb.ingress.kubernetes.io/scheme: internet-facing
alb.ingress.kubernetes.io/target-type: ip
alb.ingress.kubernetes.io/listen-ports: '[{"HTTP":80}]'
The Ingress uses:
ingressClassName: alb


    Request Routing
Path	Kubernetes Service	Port
/api/admin	admin	3003
/api/streaming	streaming	3002
/api/chat	chat	3004
/socket.io	chat	3004
/api	auth	3001
/	frontend	80



8. Verify the Helm Chart Structure
find . -maxdepth 2 -type f | sort
Expected structure:
./Chart.yaml
./values.yaml
./templates/admin-deployment.yaml
./templates/admin-service.yaml
./templates/auth-deployment.yaml
./templates/auth-service.yaml
./templates/chat-deployment.yaml
./templates/chat-service.yaml
./templates/configmap.yaml
./templates/frontend-deployment.yaml
./templates/frontend-service.yaml
./templates/ingress.yaml
./templates/mongo-service.yaml
./templates/mongo-statefulset.yaml
./templates/secret.yaml
./templates/streaming-deployment.yaml
./templates/streaming-service.yaml


9. Validate the Helm Chart
Do not install the chart yet.
First run Helm lint:
helm lint .
Then render the Kubernetes manifests locally:
helm template streamingapp . > /tmp/streamingapp-rendered.yaml


10. Install the Helm Release
Once validation succeeds, install the application into EKS:
helm install streamingapp . \
  --namespace streamingapp \
  --create-namespace
Verify the Helm release:
helm list -n streamingapp
Check the deployed Kubernetes resources:
kubectl get all -n streamingapp



11. Verify the Ingress
kubectl get ingress -n streamingapp
The ALB hostname appear in the Ingress output under ADDRESS


12. Access the Application
Get the ALB address:
kubectl get ingress streamingapp -n streamingapp
Look for the ADDRESS value. It will be an AWS ALB DNS name similar to:
xxxxx.ap-south-1.elb.amazonaws.com
Open the application in a browser:
http://<ALB-DNS-NAME>


13. Final Verification
kubectl get pods -n streamingapp
kubectl get svc -n streamingapp
kubectl get ingress -n streamingapp
helm list -n streamingapp
      └── ALB DNS





