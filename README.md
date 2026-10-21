# TravelMemory — AWS Cloud Deployment

## Project Overview

This project deploys the TravelMemory MERN application on AWS.

The application uses a React frontend, a Node.js and Express backend,
and MongoDB Atlas for database storage.

## Source Application

https://github.com/UnpredictablePrashant/TravelMemory

## Deployment Objectives

- Deploy the React frontend on Ubuntu EC2 instances using Nginx.
- Deploy the Node.js backend on Ubuntu EC2 instances.
- Configure MongoDB Atlas connectivity.
- Configure backend and frontend Application Load Balancers.
- Verify trip retrieval and creation through the application UI.
- Document the deployment with screenshots and an architecture diagram.

## Planned Architecture

Browser → Frontend Application Load Balancer → Frontend EC2 instances
→ Backend Application Load Balancer → Backend EC2 instances
→ MongoDB Atlas

Cloudflare and a custom domain are optional and depend on availability.

## Repository Structure

- `docs/deployment-guide.md` — deployment steps and verification.
- `docs/architecture.drawio` — AWS architecture diagram.
- `docs/screenshots/` — deployment evidence.
- `deployment/nginx/` — Nginx configuration files.
- `deployment/systemd/` — backend service configuration.

## Deployment Status

- [x] Source repository cloned and inspected.
- [x] Local documentation workspace created.
- [x] Deployment repository initialized.
- [ ] AWS infrastructure configured.
- [ ] Backend deployed and verified.
- [ ] Frontend deployed and verified.
- [ ] Load balancers configured and verified.
- [ ] Final application acceptance tests completed.
- [ ] Screenshots and architecture diagram added.

## Security

Never commit database credentials, `.env` files, AWS access keys,
private SSH keys, or other secrets to this repository.

## Cleanup

After completing the assignment, review and remove AWS resources
that are no longer required to avoid unnecessary charges.
