# TravelMemory AWS Deployment Guide

## 1. Project Overview

This project deploys the TravelMemory MERN application using Amazon EC2, Nginx, Application Load Balancers (ALBs), and MongoDB Atlas.

**Application source repository:**  
https://github.com/UnpredictablePrashant/TravelMemory

**Deployment documentation repository:**  
https://github.com/ananyahs681/TravelMemory-AWS-Deployment

The deployment is being completed incrementally. Each milestone is marked complete only after its configuration and functionality have been verified.

## 2. Target Architecture

The intended final architecture is:

```text
                         Users / Browser
                                |
                                v
                     Frontend Application
                        Load Balancer
                                |
                     +----------+----------+
                     |                     |
                     v                     v
              Frontend EC2 1        Frontend EC2 2
                     |                     |
                     +----------+----------+
                                |
                   Browser-side API requests
                                |
                                v
                      Backend Application
                         Load Balancer
                                |
                     +----------+----------+
                     |                     |
                     v                     v
              Backend EC2 1          Backend EC2 2
                     |                     |
                     +----------+----------+
                                |
                                v
                         MongoDB Atlas
```

**Architecture notes:**

- The frontend ALB distributes incoming web requests across the frontend EC2 instances.
- Nginx on the frontend instances serves the built React application.
- The React application runs in the user's browser.
- Browser-side API requests are sent directly to the backend ALB.
- The backend ALB distributes API requests across the backend EC2 instances.
- Nginx on each backend instance forwards requests to the Node.js application.
- The Node.js application communicates with MongoDB Atlas.
- Frontend and backend use separate target groups and separate load balancers.
- A custom domain and Cloudflare are optional and are not required for the initial deployment.

This is the intended final architecture. Additional instances and load balancers will be added after the initial application deployment has been verified.

## 3. Prerequisites

- AWS account and access to the EC2 console.
- MongoDB Atlas cluster and database user.
- Git and a local development environment.
- Ubuntu EC2 instances.
- Appropriate EC2 security groups.
- A configured SSH key pair.
- AWS billing and available credits reviewed before deploying additional resources.

## 4. Deployment Progress

### 4.1 Backend Infrastructure and Configuration

- [x] Create or resume the MongoDB Atlas cluster.
- [x] Configure the MongoDB Atlas database user.
- [x] Launch the initial backend EC2 instance.
- [x] Install Node.js, npm, Git, and Nginx.
- [x] Clone the TravelMemory source repository.
- [x] Configure backend environment variables in a server-side `.env` file.
- [x] Establish a successful MongoDB connection.
- [x] Configure Nginx as a reverse proxy to the Node.js application.
- [x] Create the `travelmemory-backend` systemd service.
- [ ] Reverify the backend service after the SSH connection timeout.
- [ ] Confirm the Elastic IP association and verify connectivity using the stable address.

### 4.2 Initial Frontend Deployment

- [ ] Launch the initial frontend EC2 instance.
- [ ] Install the required Node.js, npm, Git, and Nginx packages.
- [ ] Clone the TravelMemory source repository.
- [ ] Configure the frontend API URL.
- [ ] Install frontend dependencies and create a production build.
- [ ] Configure Nginx to serve the production build.
- [ ] Verify that the frontend loads in a browser.
- [ ] Verify that existing trips load from MongoDB Atlas through the backend API.

### 4.3 High Availability and Load Balancing

- [ ] Create an AMI of the verified backend instance.
- [ ] Create an AMI of the verified frontend instance.
- [ ] Launch the additional backend EC2 instance.
- [ ] Launch the additional frontend EC2 instance.
- [ ] Create the backend target group.
- [ ] Register both backend instances and verify target health.
- [ ] Create the backend Application Load Balancer.
- [ ] Create the frontend target group.
- [ ] Register both frontend instances and verify target health.
- [ ] Create the frontend Application Load Balancer.
- [ ] Update the frontend API URL to use the backend ALB DNS name.
- [ ] Rebuild and deploy the updated frontend.
- [ ] Verify the final application through the frontend ALB.

### 4.4 Final Verification and Documentation

- [ ] Verify that existing trips can be retrieved.
- [ ] Add a new travel experience through the UI.
- [ ] Confirm that the new record appears in the application.
- [ ] Refresh the application and verify that the record persists.
- [ ] Capture screenshots of the deployment and final verification.
- [ ] Complete the architecture diagram.
- [ ] Update this guide with final configuration details and troubleshooting notes.
- [ ] Review AWS costs and clean up resources that are no longer required.

## 5. Backend Configuration

### 5.1 EC2 Instance

The initial backend instance uses Ubuntu 24.04 LTS in the AWS Mumbai region (`ap-south-1`).

The backend application is located at:

```text
/home/ubuntu/TravelMemory/backend
```

The application entry point is:

```text
backend/index.js
```

The backend dependencies are defined in `backend/package.json`.

### 5.2 Environment Variables

The server-side environment file is located at:

```text
/home/ubuntu/TravelMemory/backend/.env
```

The configuration includes the following variables:

| Variable | Purpose |
|---|---|
| `MONGO_URI` | MongoDB Atlas connection string |
| `PORT` | Node.js application port; currently `3001` |
| `CORS_ORIGIN` | Allowed browser origins |
| `UPLOAD_DIR` | Upload directory |
| `MAX_UPLOAD_MB` | Maximum upload size |
| `TRUST_PROXY` | Express proxy configuration |
| `NODE_ENV` | Application environment |

The actual MongoDB connection string and database credentials must remain private.

**Security requirements:**

- Never commit `.env` files containing secrets.
- Never include database credentials in screenshots or public documentation.
- Restrict access to the environment file.
- Configure MongoDB Atlas network access using the required trusted source addresses rather than allowing unrestricted access.

### 5.3 Backend Endpoints

The application provides the following endpoints:

| Method | Endpoint | Purpose |
|---|---|---|
| GET | `/hello` | Basic endpoint check |
| GET | `/health` | Application and database health check |
| GET | `/trip` | Retrieve trips |
| GET | `/trip/:id` | Retrieve a specific trip |
| POST | `/trip` | Create a trip |

The `/health` endpoint is intended to verify application health and MongoDB connectivity.

### 5.4 Nginx Reverse Proxy

Nginx listens on HTTP port `80` and forwards requests to the Node.js application on port `3001`.

The backend Nginx configuration is located at:

```text
/etc/nginx/sites-available/travelmemory-backend
```

The enabled site is:

```text
/etc/nginx/sites-enabled/travelmemory-backend
```

The proxy forwards incoming requests to:

```text
http://127.0.0.1:3001
```

The default Nginx site was disabled to prevent conflicts with the TravelMemory backend configuration.

### 5.5 systemd Service

The backend service is named:

```text
travelmemory-backend
```

Its service definition is located at:

```text
/etc/systemd/system/travelmemory-backend.service
```

The service runs the Node.js application from the backend directory and uses the server-side environment file.

The service is configured to start automatically and restart if the process exits unexpectedly.

The service's current operational status must be reverified after the SSH timeout.

### 5.6 Backend Verification

Run these commands on the backend EC2 instance:

```bash
sudo systemctl status travelmemory-backend --no-pager
```

```bash
curl -i http://localhost:3001/health
```

```bash
curl -i http://localhost/hello
```

Expected results:

- The systemd service reports `active (running)`.
- The health endpoint returns a successful response and reports a connected database.
- The `/hello` endpoint returns `Hello World!`.

Record the verification date and any issues encountered.

## 6. Frontend Configuration

The React frontend must be configured to use the correct backend API URL.

The API URL configuration is located at:

```text
frontend/src/url.js
```

The production API URL must point to the backend Application Load Balancer after it has been created.

Because the API URL is included in the React production build, changes to the URL require rebuilding and redeploying the frontend.

The initial frontend deployment must first be tested against the currently available backend endpoint. After the backend ALB is created, update the API URL and repeat the frontend build and deployment.

The exact environment-variable behavior and fallback URL will be confirmed against the application's actual source code during frontend deployment.

## 7. Load Balancer Configuration

### 7.1 Backend Load Balancer

Intended configuration:

- Load balancer type: Application Load Balancer.
- Scheme: Internet-facing.
- Listener: HTTP port `80`.
- Target group protocol: HTTP.
- Target group port: `80`.
- Health check path: `/hello`.
- Targets: Both backend EC2 instances.

Nginx receives traffic on port `80` and proxies it to the Node.js application on port `3001`.

Verify that both backend targets report healthy status before final application testing.

### 7.2 Frontend Load Balancer

Intended configuration:

- Load balancer type: Application Load Balancer.
- Scheme: Internet-facing.
- Listener: HTTP port `80`.
- Target group protocol: HTTP.
- Target group port: `80`.
- Health check path: `/`.
- Targets: Both frontend EC2 instances.

Verify that both frontend targets report healthy status before final application testing.

### 7.3 Network Access

Configure security groups so that:

- SSH access is restricted to the required administrative source IP.
- Public HTTP access is allowed where required for the internet-facing load balancers.
- Application instances accept HTTP traffic from the appropriate load balancer security group.
- Backend API access is restricted to the intended traffic paths wherever feasible.
- MongoDB Atlas permits connections from the backend's current stable public IP address or another deliberately configured network path.

Review the effective security group rules before exposing the final application.

## 8. Acceptance Tests

Perform the following tests after completing the load balancer configuration:

1. Open the frontend using the frontend ALB DNS name.
2. Confirm that the React application loads.
3. Confirm that existing trips load successfully.
4. Confirm that browser API requests reach the backend through the backend ALB.
5. Add a new travel experience through the UI.
6. Confirm that the new experience appears in the application.
7. Refresh the page and verify that the record remains available.
8. Confirm that both frontend targets and both backend targets are healthy.
9. Record the test date, results, and supporting screenshots.

## 9. Screenshots and Evidence

Store screenshots in:

```text
docs/screenshots/
```

Suggested folders:

```text
docs/screenshots/
├── 01-aws-setup/
├── 02-backend/
├── 03-frontend/
├── 04-load-balancers/
└── 05-final-verification/
```

Capture evidence of:

- Initial AWS configuration and billing review.
- Backend EC2 configuration and service status.
- Successful backend health checks.
- Frontend deployment and application UI.
- Load balancer listeners and target health.
- Successful trip retrieval and creation.

Redact account identifiers and never capture passwords, private keys, access tokens, or database connection strings.

## 10. Troubleshooting Notes

Record each significant issue using the following format:

| Item | Details |
|---|---|
| Issue | Describe the observed problem |
| Root cause | Record the confirmed cause, if known |
| Resolution | Document the change made |
| Verification | Record the test that confirmed the fix |
| Date | Record when the issue occurred |

For SSH timeouts, check the instance state, public or Elastic IP association, security group rules, and network connectivity before changing application configuration.

For backend failures, inspect the systemd service status and journal logs.

Useful commands:

```bash
sudo systemctl status travelmemory-backend --no-pager
```

```bash
sudo journalctl -u travelmemory-backend -n 100 --no-pager
```

```bash
sudo nginx -t
```

```bash
curl -i http://localhost:3001/health
```

Do not record credentials or sensitive environment-variable values in troubleshooting logs.

## 11. Cost Management and Cleanup

Review AWS billing before and during deployment.

Resources that may incur charges include:

- EC2 instances.
- EBS volumes and snapshots.
- Public IPv4 addresses and Elastic IPs.
- Application Load Balancers.
- Data transfer.

An Elastic IP may incur charges depending on its usage and the applicable AWS pricing rules. Avoid leaving unused addresses allocated.

After completing the assignment and collecting the required evidence:

1. Identify which AWS resources must remain available.
2. Stop or terminate resources that are no longer required, as appropriate.
3. Delete unused load balancers and target groups.
4. Review unused EBS volumes and snapshots.
5. Release unused Elastic IP addresses.
6. Verify that no unnecessary billable resources remain.

Retain any resources needed for evaluation until the assignment requirements have been satisfied.