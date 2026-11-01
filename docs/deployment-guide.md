# TravelMemory AWS Deployment Guide

## 1. Project Overview

This project demonstrates the deployment of the TravelMemory MERN application on Amazon Web Services (AWS).

The application consists of a React frontend, a Node.js and Express backend, and MongoDB Atlas for database storage.

**Application source:** [TravelMemory — GitHub]
(https://github.com/UnpredictablePrashant/TravelMemory)
**Deployment repository:** [TravelMemory AWS Deployment — GitHub](https://github.com/ananyahs681/TravelMemory-AWS-Deployment)

## 2. Deployed Architecture

The application was deployed using separate frontend and backend Application Load Balancers, with two EC2 instances for each application tier.

```text
                         Users / Browser
                               |
                               v
                     Frontend Application
                       Load Balancer
                               |
                  +------------+------------+
                  |                         |
                  v                         v
            Frontend EC2 1            Frontend EC2 2
                  |                         |
                  +------------+------------+
                               |
                      React application
                      runs in browser
                               |
                        API requests
                               |
                               v
                     Backend Application
                       Load Balancer
                               |
                  +------------+------------+
                  |                         |
                  v                         v
             Backend EC2 1            Backend EC2 2
                  |                         |
                  +------------+------------+
                               |
                               v
                         MongoDB Atlas
```

### Architecture notes

- The frontend ALB distributes incoming web requests across the frontend EC2 instances.
- Nginx serves the production React build on each frontend instance.
- The React application runs in the user's browser.
- Browser API requests are sent to the backend ALB.
- The backend ALB distributes requests across the backend EC2 instances.
- Nginx on each backend instance forwards requests to Node.js on port `3001`.
- The Node.js application connects to MongoDB Atlas.
- Frontend and backend use separate target groups and load balancers.

The architecture diagram is available at [`docs/TravelMemory-AWS-Architecture.drawio.png`](TravelMemory-AWS-Architecture.drawio.png), with the editable file at [`docs/architecture.drawio`](architecture.drawio).

## 3. Deployment Environment

| Component | Configuration |
|---|---|
| Cloud provider | Amazon Web Services |
| AWS Region | Asia Pacific (Mumbai) — `ap-south-1` |
| Operating system | Ubuntu 24.04 LTS |
| Frontend | React.js |
| Frontend web server | Nginx |
| Backend | Node.js and Express |
| Backend process management | systemd |
| Database | MongoDB Atlas |
| Load balancing | Application Load Balancer |
| Compute | Amazon EC2 |

## 4. Backend Deployment

### 4.1 Application and Dependencies

The backend application was deployed on Ubuntu EC2 instances.

Application directory:

```text
/home/ubuntu/TravelMemory/backend
```

Application entry point:

```text
backend/index.js
```

The backend dependencies are defined in `backend/package.json`.

### 4.2 Environment Configuration

The server-side environment file is located at:

```text
/home/ubuntu/TravelMemory/backend/.env
```

The configuration includes:

| Variable | Purpose |
|---|---|
| `MONGO_URI` | MongoDB Atlas connection string |
| `PORT` | Node.js application port (`3001`) |
| `CORS_ORIGIN` | Allowed browser origins |
| `UPLOAD_DIR` | Upload directory |
| `MAX_UPLOAD_MB` | Maximum upload size |
| `TRUST_PROXY` | Express proxy configuration |
| `NODE_ENV` | Application environment |

**Credentials and secret values are intentionally excluded from this guide.**

### 4.3 Backend Endpoints

| Method | Endpoint | Purpose |
|---|---|---|
| GET | `/hello` | Basic endpoint check |
| GET | `/health` | Application and database health check |
| GET | `/trip` | Retrieve trips |
| GET | `/trip/:id` | Retrieve a specific trip |
| POST | `/trip` | Create a trip |

The `/health` endpoint was verified to return a successful response with the database connected.

### 4.4 Nginx Reverse Proxy

Nginx listens on HTTP port `80` and forwards requests to the Node.js application on port `3001`.

Backend Nginx configuration:

```text
/etc/nginx/sites-available/travelmemory-backend
```

Enabled site:

```text
/etc/nginx/sites-enabled/travelmemory-backend
```

The reverse proxy forwards requests to:

```text
http://127.0.0.1:3001
```

The default Nginx site was disabled to prevent conflicts with the TravelMemory backend configuration.

### 4.5 systemd Service

The backend service is named:

```text
travelmemory-backend
```

Service definition:

```text
/etc/systemd/system/travelmemory-backend.service
```

The service runs the Node.js application from the backend directory, loads the server-side environment file, starts automatically, and is configured to restart if the process exits unexpectedly.

### 4.6 Backend Verification

The following checks were completed during deployment:

```bash
sudo systemctl status travelmemory-backend --no-pager
```

```bash
curl -i http://localhost:3001/health
```

```bash
curl -i http://localhost/hello
```

**Verified results:**

- The backend systemd service was running.
- The health endpoint reported a successful database connection.
- The `/hello` endpoint responded successfully.
- Both backend target instances reported healthy status.

## 5. Frontend Deployment

The React frontend was deployed to Ubuntu EC2 instances and served using Nginx.

### 5.1 API Configuration

The frontend API URL is configured in:

```text
frontend/src/url.js
```

The production build was configured to use the backend Application Load Balancer as its API endpoint.

Because the API URL is included in the React production build, changing it requires rebuilding and redeploying the frontend.

### 5.2 Production Build

The production build was generated using the backend ALB URL:

```bash
REACT_APP_BACKEND_URL=http://<backend-alb-dns-name> npm run build
```

The build output was copied to the Nginx web directory:

```bash
sudo cp -r build/. /var/www/travelmemory/
sudo chown -R www-data:www-data /var/www/travelmemory
```

The frontend Nginx configuration was tested, and the application returned a successful HTTP response.

### 5.3 Frontend Verification

The following checks were completed:

- The frontend application loaded through the frontend ALB.
- Existing trips were retrieved successfully.
- New trips could be created through the application UI.
- Newly created trips appeared on the homepage after refreshing.
- Both frontend target instances reported healthy status.

## 6. Load Balancer Configuration

Two internet-facing Application Load Balancers were configured.

### 6.1 Backend ALB

| Setting | Configuration |
|---|---|
| Listener | HTTP `80` |
| Target group protocol | HTTP |
| Target group port | `80` |
| Health check path | `/hello` |
| Targets | Two backend EC2 instances |

Nginx on the backend instances receives traffic on port `80` and proxies it to Node.js on port `3001`.

Both backend targets were verified healthy.

### 6.2 Frontend ALB

| Setting | Configuration |
|---|---|
| Listener | HTTP `80` |
| Target group protocol | HTTP |
| Target group port | `80` |
| Health check path | `/` |
| Targets | Two frontend EC2 instances |

Nginx serves the production React application on each frontend instance.

Both frontend targets were verified healthy.

### 6.3 Network Access

Security groups were configured to support the required application traffic:

- SSH access was restricted to the required administrative source.
- HTTP access was enabled for the internet-facing load balancers.
- Application instances accepted traffic from the appropriate load balancer security groups.
- MongoDB Atlas network access was configured to permit backend connectivity.

## 7. Acceptance Tests

The following acceptance tests were completed:

| Test | Result |
|---|---|
| Frontend loads through the frontend ALB | Passed |
| Existing trips are retrieved | Passed |
| Backend `/health` reports database connectivity | Passed |
| Backend `/hello` responds successfully | Passed |
| New trip can be created through the UI | Passed |
| Newly created trip appears after refresh | Passed |
| Both frontend targets report healthy | Passed |
| Both backend targets report healthy | Passed |

## 8. Screenshots and Evidence

Deployment screenshots are stored in:

```text
docs/screenshots/
```

The evidence includes:

- Frontend application homepage
- Frontend and backend target group health
- Backend health check
- Backend and frontend ALB details
- Running EC2 instances
- MongoDB Atlas database configuration
- Saved trips in the application
- Backend systemd service status
- Successful frontend production build

The screenshots provide evidence of the deployment and verification completed during the assignment.

## 9. Security Considerations

- Do not commit database credentials, `.env` files, AWS access keys, private SSH keys, or other secrets.
- Keep database credentials in server-side environment files.
- Restrict SSH access to trusted administrative IP addresses.
- Configure security groups to allow only required traffic.
- Do not expose sensitive credentials in screenshots or documentation.

## 10. Cost Management and Cleanup

AWS resources may incur charges, including:

- EC2 instances
- EBS volumes and snapshots
- Application Load Balancers
- Public IPv4 addresses and Elastic IPs
- Data transfer

After collecting the required assignment evidence:

1. Identify resources that must remain available for evaluation.
2. Stop or terminate resources that are no longer required.
3. Delete unused load balancers and target groups.
4. Review unused EBS volumes and snapshots.
5. Release unused Elastic IP addresses.
6. Review AWS billing to confirm that unnecessary billable resources have been removed.

Retain any resources required for evaluation until the assignment requirements have been satisfied.
