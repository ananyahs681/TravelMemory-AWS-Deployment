# TravelMemory AWS Deployment Guide

## 1. Project Overview

Deploy the TravelMemory application using AWS EC2, Nginx,
Application Load Balancers, and MongoDB Atlas.

Source repository:
https://github.com/UnpredictablePrashant/TravelMemory

## 2. Architecture

Browser
  -> Frontend Application Load Balancer
  -> Frontend EC2 instance 1
  -> Frontend EC2 instance 2
  -> Backend Application Load Balancer
  -> Backend EC2 instance 1
  -> Backend EC2 instance 2
  -> MongoDB Atlas

Note: The frontend and backend each have their own target group.
The two frontend EC2 instances are targets of the frontend load
balancer; the two backend EC2 instances are targets of the backend
load balancer. The frontend calls the backend through its load
balancer. Cloudflare and a custom domain are optional.

## 3. Prerequisites

- AWS account and access to the EC2 console.
- MongoDB Atlas cluster and database user.
- Git and a local development environment.
- Ubuntu EC2 instances.
- Security groups configured for the required traffic.

## 4. Deployment Status

Deployment has not started yet. Record each step only after
performing and verifying it.

- [ ] Verify AWS billing and available credits.
- [ ] Configure MongoDB Atlas network access and credentials.
- [ ] Launch the initial backend EC2 instance.
- [ ] Install Node.js, npm, Git, and Nginx on the backend.
- [ ] Configure the backend environment variables.
- [ ] Start the backend and verify the health endpoint.
- [ ] Configure a systemd service for the backend.
- [ ] Launch the initial frontend EC2 instance.
- [ ] Configure the frontend API URL and build the React app.
- [ ] Serve the frontend through Nginx.
- [ ] Verify that trips load from MongoDB Atlas.
- [ ] Create backend and frontend AMIs.
- [ ] Launch the additional EC2 instances.
- [ ] Configure both target groups and load balancers.
- [ ] Update the frontend to use the backend load balancer.
- [ ] Rebuild and deploy the updated frontend.
- [ ] Test trip retrieval and creation through the UI.
- [ ] Capture screenshots and complete the architecture diagram.
- [ ] Review and clean up billable resources after submission.

## 5. Backend Configuration

The backend entry point is `backend/index.js`.
It reads the `PORT` and `MONGO_URI` environment variables.

The application exposes these endpoints:

- `GET /hello` - health check.
- `GET /trip` - retrieve all trips.
- `GET /trip/:id` - retrieve one trip.
- `POST /trip` - create a trip.

The intended deployment port is 3000. Verify the configured port
and actual service behavior during deployment.

Keep database credentials in a server-side `.env` file.
Never commit secrets to GitHub.

## 6. Frontend Configuration

The React frontend reads its backend address from
`REACT_APP_BACKEND_URL` during the build, with a local fallback
defined in `frontend/src/url.js`.

Configure the production backend URL before building the frontend.
The production URL must point to the backend load balancer.

## 7. Load Balancers

Backend:
- Target group protocol: HTTP.
- Target port: 80 when Nginx proxies traffic to the Node.js service.
- Health check path: `/hello`.

Frontend:
- Target group protocol: HTTP.
- Target port: 80.
- Health check path: `/`.

Verify that both target groups report healthy targets before
performing the final application tests.

## 8. Acceptance Tests

- Open the frontend using the frontend load balancer DNS name.
- Confirm that existing trips load.
- Confirm that browser requests reach the backend through its load
  balancer.
- Add a new travel experience through the UI.
- Confirm that the new experience appears in the application.
- Refresh the page and confirm the saved record remains available.

Record the test date, results, and supporting screenshots.

## 9. Screenshots and Evidence

Store screenshots under `docs/screenshots/`:

- `01-aws-setup/` - billing and initial AWS configuration.
- `02-backend/` - backend service and health checks.
- `03-frontend/` - frontend deployment and UI.
- `04-load-balancers/` - listeners, target groups, and target health.
- `05-final-verification/` - trip retrieval and trip creation.

Redact account identifiers and never capture passwords, private keys,
or database connection strings in screenshots.

## 10. Troubleshooting Notes

Document each issue, its root cause, the fix, and the verification
result after troubleshooting during the actual deployment.

## 11. Cost and Cleanup

Review AWS billing before and during deployment. EC2 instances,
EBS volumes, public IPv4 addresses, Application Load Balancers,
and data transfer may incur charges.

After collecting the required evidence, remove resources that are
no longer needed. Verify that unused load balancers, volumes, and
other billable resources have been removed.
