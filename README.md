# TravelMemory: High-Availability AWS Deployment

Building and deploying a modern full-stack MERN (MongoDB, Express, React, Node.js) application into a high-availability production environment requires clean architectural decoupling, robust process management, immutable infrastructure backups, and elastic load balancing.

This comprehensive technical guide outlines the complete end-to-end lifecycle deployment of **TravelMemory**—taking the application from source code on dedicated server IPs to a resilient multi-instance AWS cluster managed behind dual Application Load Balancers (ALBs).

**Source Code Repository:** [SarthakNigam8000/Travel_Memory](https://github.com/SarthakNigam8000/Travel_Memory)

**Documentation link:** https://medium.com/@sarthaknigamm/building-deploying-high-availability-mern-applications-on-aws-from-ec2-mongodb-atlas-to-dual-9ae7dc40b5aa

---

## End-to-End System Architecture

Our production topology is partitioned into three distinct tiers across AWS (Asia Pacific - Mumbai `ap-south-1`) and MongoDB Atlas:

1. **Database Tier:** Cloud-managed MongoDB Atlas cluster (`hv.jhwezgk.mongodb.net`) ensuring scalable NoSQL data persistence.
2. **Backend Application Tier:** Multi-instance Node.js & Express REST API (`backend-master` at `13.233.193.0`, plus `backend-2`) managed via systemd process daemons and fronted by the Backend Application Load Balancer (`travelmemory-backend-alb-2126919898.ap-south-1.elb.amazonaws.com`).
3. **Frontend Web Tier:** Multi-instance React SPA (`frontend-master` at `65.0.181.156`, plus `frontend-2`) served via Nginx web servers and exposed to users through the public Frontend Application Load Balancer (`travelmemory-frontend-alb-1871682931.ap-south-1.elb.amazonaws.com`).

### Traffic Flow
`Web Users` -> `Frontend ALB` -> `Frontend EC2 (Nginx/React)` -> `Backend ALB` -> `Backend EC2 (Node/Express)` -> `MongoDB Atlas`

---

## **Step 1: Database Setup via MongoDB Atlas & Compass**

Before spinning up compute instances, we configure our persistent cloud database.

1. Provision a cloud cluster in MongoDB Atlas and create your database user credentials (`sarthaknigamm_db_user`).
2. Configure Network Access rules to allow inbound connections from your server IPs.
3. Verify connectivity using MongoDB Compass. Compass should successfully connect to the Atlas cluster, displaying the target database `travelmemory`.

## Step 2: Provisioning Primary AWS EC2 Instances

In the AWS Console (Region: `ap-south-1`), provision two initial Ubuntu `t2.micro` instances:

* **frontend-master:** Dedicated web server hosting static React production bundles at Public IP `65.0.181.156`.
* **backend-master:** Dedicated application server running the Express REST API at Public IP `13.233.193.0`.

Both instances are deployed in Availability Zone `ap-south-1b`.

## Step 3: Backend Environment & Systemd Daemon Setup

SSH into `backend-master` (`13.233.193.0`), clone the repository, install dependencies, and configure the application:

```bash
git clone [https://github.com/UnpredictablePrashant/TravelMemory.git](https://github.com/UnpredictablePrashant/TravelMemory.git)
cd TravelMemory/backend
npm install
```
Create and populate /home/ubuntu/TravelMemory/backend/.env:
```
MONGO_URI=mongodb+srv://sarthaknigamm_db_user:<password>@hv.jhwezgk.mongodb.net/travelmemory?retryWrites=true&w=majority
PORT=3000
```

To ensure process persistence and automatic restarts on server reboots or crashes, create a systemd service file at /etc/systemd/system/travelmemory-backend.service:
```
[Unit]
Description=TravelMemory Backend
After=network.target

[Service]
Type=simple
User=ubuntu
WorkingDirectory=/home/ubuntu/TravelMemory/backend
EnvironmentFile=/home/ubuntu/TravelMemory/backend/.env
ExecStart=/usr/bin/node index.js
Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target
```

Enable and start the service:
```
sudo systemctl daemon-reload
sudo systemctl enable --now travelmemory-backend
```
## Step 4: Frontend Environment Setup, URL Config & Build Pipeline
On frontend-master (65.0.181.156), configure the React environment variables and URL mapping so that the frontend communicates directly with the backend server IP.

Create a .env file inside the frontend directory (TravelMemory/frontend/.env):
```
REACT_APP_BACKEND_URL=[http://13.233.193.0](http://13.233.193.0)
```
Update src/url.js to point explicitly to the backend server IP address:
```
export const baseUrl = process.env.REACT_APP_BACKEND_URL || "[http://13.233.193.0](http://13.233.193.0)";
```
Configure Nginx for SPA Routing: To prevent 404 errors when refreshing or navigating client-side routes in React, update the Nginx configuration file at /etc/nginx/sites-available/default.

Replace the default location / block with the SPA fallback rule:
```
location / {
    try_files $uri$uri/ /index.html;
}
```
Execute the production build sequence and restart Nginx:
```
npm run build
sudo rm -rf /var/www/html/*
sudo cp -r build/* /var/www/html/
sudo systemctl restart nginx
sudo systemctl status nginx
```
Validation: sudo nginx -t confirms configuration syntax, and systemctl status nginx confirms the web server is active.


## Step 5: Functional UI & Database Integration Testing
Before horizontal scaling, verify full-stack data flow directly through the live IP web interface.

Initial Landing Page & Detail View: Accessing the frontend public IP (http://65.0.181.156) loads the application. Clicking "More Details" loads itinerary details and destination imagery fetched from backend IP 13.233.193.0.

Creating New Travel Experiences: Navigate to /addexperience. A new entry (e.g., "Goa") is filled with hotel details, cost, locations visited, image link, and description.

Database Persistence Verification: Checking MongoDB Compass confirms the newly created document under the tripdetails collection.

Feed & Detail Refresh: Refreshing the web interface confirms live API data fetching, showing the newly added entry in the feed.

## Step 6: Baking Amazon Machine Images (AMIs)
To scale out seamlessly without re-running installation commands, create system AMIs from the configured master instances:

Select backend-master -> Actions -> Image and templates -> Create image (travelmemory-backend-ami).

Select frontend-master -> Actions -> Image and templates -> Create image (travelmemory-frontend-ami).

## Step 7: Launching Scaled Instances from AMIs
Using these AMIs, launch secondary instances to establish a resilient four-node cluster. Four active EC2 instances are now running with individual public IPs:

frontend-master: 65.0.181.156 (Instance ID: i-082f0f6b308241f06)

backend-master: 13.233.193.0 (Instance ID: i-0f7739521479b291a)

backend-2: 43.204.116.244 (Instance ID: i-0850330631a16359a)

frontend-2: 13.203.195.170 (Instance ID: i-0688938d136b447e4)

## Step 8: Configuring Backend Target Group & Application Load Balancer
To distribute REST API requests across backend nodes, set up Elastic Load Balancing.

Create Target Group: Create travelmemory-backend-tg (Port 80) and register backend-master and backend-2.

Create ALB: Launch travelmemory-backend-alb forwarding HTTP traffic to travelmemory-backend-tg.

ALB DNS: travelmemory-backend-alb-2126919898.ap-south-1.elb.amazonaws.com

## Step 9: Configuring Frontend Target Group & Application Load Balancer
Repeat the load-balancing process for the frontend web tier.

Create Target Group: Create travelmemory-frontend-tg (Port 80) and register frontend-master and frontend-2.

Create ALB: Launch travelmemory-frontend-alb forwarding HTTP traffic to travelmemory-frontend-tg.

ALB DNS: travelmemory-frontend-alb-1871682931.ap-south-1.elb.amazonaws.com

## Step 10: Routing Frontend Nodes to Backend Load Balancer
To prevent web requests from hardcoding a single backend IP, update the environment configuration on all frontend servers to point to the Backend ALB DNS.

SSH into frontend-master and frontend-2, update .env, and rebuild:
```
REACT_APP_BACKEND_URL=[http://travelmemory-backend-alb-1480453010.ap-south-1.elb.amazonaws.com](http://travelmemory-backend-alb-1480453010.ap-south-1.elb.amazonaws.com)
```
Rebuild static assets and restart Nginx on both frontend instances:
```
npm run build
sudo cp -r build/* /var/www/html/
sudo systemctl restart nginx
```
## Step 11: Final Cluster Testing via Frontend ALB
Access the production application stack exclusively through the Frontend Application Load Balancer DNS:http://travelmemory-frontend-alb-1871682931.ap-south-1.elb.amazonaws.com

The landing feed renders seamlessly over the Frontend ALB, forms load correctly through load-balanced routing, and full trip details are retrieved and displayed across the dual-ALB topology.
