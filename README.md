# Two-Tier Flask + MySQL Application (Dockerized)

![Python](https://img.shields.io/badge/-Python-3776AB?style=flat&logo=python&logoColor=white)
![Flask](https://img.shields.io/badge/-Flask-000000?style=flat&logo=flask&logoColor=white)
![MySQL](https://img.shields.io/badge/-MySQL-4479A1?style=flat&logo=mysql&logoColor=white)
![Docker](https://img.shields.io/badge/-Docker-2496ED?style=flat&logo=docker&logoColor=white)
![AWS EC2](https://img.shields.io/badge/-AWS%20EC2-FF9900?style=flat&logo=amazon-aws&logoColor=white)

A **two-tier containerized application** — a Flask message board backed by MySQL — demonstrating multi-container Docker networking, environment-based configuration, and deployment on an AWS EC2 instance.

---

## Table of Contents

- [About the Project](#about-the-project)
- [Architecture](#architecture)
- [Tech Stack](#tech-stack)
- [Project Structure](#project-structure)
- [Deployment Guide](#deployment-guide)
- [Accessing the MySQL Container](#accessing-the-mysql-container)
- [Known Issues / To-Do](#known-issues--to-do)
- [Future Enhancements](#future-enhancements)
- [Author](#author)

---

## About the Project

This project is a simple Flask-based message board where users can submit messages that are stored in a MySQL database and displayed on the front end. The application is split into two independently containerized tiers:

- **App tier** — a Flask web server handling requests and business logic
- **Data tier** — a MySQL database storing submitted messages

Both containers run independently and communicate over a **custom Docker bridge network**, mirroring how multi-tier applications are structured in real production environments.

---

## Architecture

```
                    ┌──────────────────────┐
                    │        User          │
                    └──────────┬───────────┘
                               │  HTTP :5000
                               ▼
                    ┌──────────────────────┐
                    │   AWS EC2 Instance    │
                    │  (Security Group:     │
                    │   port 5000 open)     │
                    └──────────┬───────────┘
                               │
                 Docker Bridge Network: "two-tier"
                               │
        ┌──────────────────────┼──────────────────────┐
        ▼                                              ▼
┌──────────────────┐                          ┌──────────────────┐
│  Flask App        │  MYSQL_HOST=mysql        │  MySQL Container  │
│  Container        │ ───────────────────────▶ │  (mysql image)     │
│  (two-tier-       │                          │                    │
│   backend:latest) │                          │  DB: devops        │
└──────────────────┘                          └──────────────────┘
```

The two containers are attached to the same Docker network (`two-tier`), so the Flask app can reach the database simply by its container name (`mysql`) rather than an IP address.

---

## Tech Stack

| Category | Technology |
|---|---|
| Backend | Python, Flask |
| Database | MySQL |
| Containerization | Docker, Docker Networking (bridge) |
| Cloud Hosting | AWS EC2 |

---

## Project Structure

```
two-tier-flask-app-docker/
│
├── app.py                  # Flask application (routes, MySQL connection)
├── requirements.txt        # Python dependencies
├── Dockerfile              # Docker build instructions for the app tier
├── message.sql             # SQL schema for the messages table
│
├── templates/
│   └── index.html          # Front-end message board UI
│
└── README.md
```

---

## Deployment Guide

These are the exact steps used to deploy this two-tier application on an AWS EC2 instance using Docker networking.

### 1. Pull the MySQL image
```bash
docker pull mysql
```

### 2. Pull the application code
```bash
git clone <this-repo-url>
cd two-tier-flask-app-docker
```

### 3. Build the Flask app image
```bash
docker build -t two-tier-backend:latest .
```

### 4. (Optional) Quick standalone test run
Run the app on its own first to confirm it builds and starts correctly:
```bash
docker run -d -p 5000:5000 \
  -e MYSQL_HOST=mysql \
  -e MYSQL_USER=root \
  -e MYSQL_PASSWORD=root \
  -e MYSQL_DB=devops \
  two-tier-backend:latest
```

### 5. Create a dedicated Docker network
So the app and database containers can discover each other by name:
```bash
docker network create two-tier -d bridge
```

### 6. Run the MySQL container on that network
```bash
docker run -d --name mysql --network two-tier \
  -e MYSQL_ROOT_PASSWORD=root \
  -e MYSQL_DATABASE=devops \
  mysql
```

### 7. Run the Flask app container on the same network
```bash
docker run -d -p 5000:5000 --network two-tier \
  -e MYSQL_HOST=mysql \
  -e MYSQL_USER=root \
  -e MYSQL_PASSWORD=root \
  -e MYSQL_DB=devops \
  two-tier-backend:latest
```

### 8. Open port 5000 on the EC2 security group
In the AWS Console → EC2 → Security Groups → Inbound Rules, add a rule allowing **TCP port 5000** from your IP (or `0.0.0.0/0` for public testing).

### 9. Access the app
Copy the EC2 instance's public IP and visit:
```
http://<ec2-public-ip>:5000
```

---

## Accessing the MySQL Container

To inspect the database directly from inside its container:

```bash
# Find the running MySQL container's ID
docker ps

# Open a shell inside the container
docker exec -it <mysql-container-id> bash

# Log in to the MySQL monitor
mysql -u root -p
# Enter the MYSQL_ROOT_PASSWORD you set (e.g. "root")
```

Once inside the MySQL monitor, you can inspect the data directly:
```sql
USE devops;
SELECT * FROM messages;
```

---

## Future Enhancements

- [ ] Add Kubernetes manifests (Deployment, Service, ConfigMap, Secret) under `k8s/`
- [ ] Add AWS EKS deployment manifests under `eks-manifests/`
- [ ] Replace manual `docker run` commands with a `docker-compose.yml` for one-command local setup
- [ ] Add persistent volume for MySQL data so messages survive container restarts
- [ ] Add HTTPS via a reverse proxy (e.g. Nginx) or an Application Load Balancer

---

## Author

**Md Farhan Kalim**

[![GitHub](https://img.shields.io/badge/-GitHub-181717?style=flat&logo=github&logoColor=white)](https://github.com/Mcadefarhan)
[![LinkedIn](https://img.shields.io/badge/-LinkedIn-0077B5?style=flat&logo=linkedin&logoColor=white)](https://linkedin.com/in/md-farhan-kalim-9a8823325)

---

⭐ If you found this project useful, consider giving it a star!
THANK YOU !
