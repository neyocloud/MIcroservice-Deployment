# MIcroservice-Deployment
CI/CD with GitHub Actions: Deploying a Docker Compose App on AWS EC2



# What You’ll Learn

1) What are micro-services?

2) How to deploy a micro-service app using Docker (Docker Compose) locally

3) Automating the deployment process to an EC2 instance using CI/CD with GitHub Actions



# Project Goals

Containerize a multi‑service web app (Vote → Redis → Worker → Postgres → Result).

Run locally via Docker Compose.

Push images to GitHub Container Registry (GHCR) or Docker Hub.

Auto‑deploy to AWS EC2 using GitHub Actions over SSH.


# 🧱 Architecture

[ vote (Flask) ] -> [ redis ]

[ worker (.NET) ] -> [ redis, db ]

[ result (Node) ] -> [ db ]

vote writes to redis

worker drains redis → persists to postgres

result reads tallies from postgres


<img width="860" height="800" alt="image" src="https://github.com/user-attachments/assets/b6dfbc7e-f598-4f80-8375-0dba6e04ce9b" />




# Repo structure
<img width="1512" height="982" alt="image" src="https://github.com/user-attachments/assets/c8720fbb-52bf-43de-8415-dfd9d172fd1c" />



# DOCKER  ENGINE RUNNING 

docker compose -f compose/docker-compose.yml up -d --build

open http://localhost:80 # vote UI

open http://localhost:8091 # result UI

<img width="1512" height="982" alt="image" src="https://github.com/user-attachments/assets/601952fc-0f68-41ee-a2da-79b8407b74be" />





# LOGS & STATUS

docker ps

docker compose up

docker compose -d (running in the background )

docker logs ########
Docker images (published)

neyocicd/vote:latest

neyocicd/result:latest

neyocicd/worker:latest

# These are built by the CI pipeline on pushes to main.

CI/CD (GitHub Actions)

Workflow file: .github/workflows/deploy.yml

Pipeline stages:

Build & Push Images – builds vote, result, worker and pushes to Docker Hub.

Deploy to EC2 – (optional) copies docker-compose.yaml to EC2 and runs docker compose up -d.

Required repository secrets (Settings → Secrets and variables → Actions):

DOCKERHUB_USERNAME = neyocicd

DOCKERHUB_TOKEN = Docker Hub Access Token (write access)

EC2_HOST = EC2 public DNS/IP

EC2_USER = ubuntu (for Ubuntu AMIs)

EC2_SSH_KEY = private key contents used to SSH to EC2

-----BEGIN OPENSSH PRIVATE KEY-----
...
-----END OPENSSH PRIVATE KEY-----


Tip: The workflow can be set to skip deploy automatically if EC2 secrets aren’t present, so the run stays green for documentation.

First-time EC2 setup (one-off)

SSH into the instance and install Docker:

# on EC2
curl -fsSL https://get.docker.com | sh
sudo usermod -aG docker $USER
newgrp docker
docker --version
docker compose version || sudo apt-get install -y docker-compose-plugin
sudo mkdir -p /opt/app


Security Group:

Inbound: 22 (SSH) from your IP

Inbound: 80 (HTTP) from 0.0.0.0/0



Production compose (snippet)

docker-compose.yaml (at repo root) should reference the published images:

services:
  vote:
    image: neyocicd/vote:latest
    ports: ["80:80"]
    depends_on:
      redis:
        condition: service_healthy
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost"]
      interval: 30s
      timeout: 10s
      retries: 5
      start_period: 10s

  result:
    image: neyocicd/result:latest
    ports: ["8091:80"]
    depends_on:
      db:
        condition: service_healthy

  worker:
    image: neyocicd/worker:latest
    depends_on:
      db:
        condition: service_healthy
      redis:
        condition: service_healthy

  redis:
    image: redis:6-alpine
    volumes: [ "redis-data:/data" ]
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 30s
      timeout: 10s
      retries: 5
      start_period: 10s

  db:
    image: postgres:15-alpine
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
    volumes: [ "db-data:/var/lib/postgresql/data" ]
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres -d postgres || exit 1"]
      interval: 10s
      timeout: 5s
      retries: 10
      start_period: 30s

volumes:
  db-data:
  redis-data:

Manual deploy (fallback)
# copy compose to server
scp -i ~/.ssh/<your-key> docker-compose.yaml ubuntu@<EC2_HOST>:/opt/app/

# SSH and bring up the stack
ssh -i ~/.ssh/<your-key> ubuntu@<EC2_HOST> <<'EOF'
cd /opt/app
docker login -u neyocicd -p '<DOCKER_HUB_TOKEN>'
docker compose pull
docker compose up -d
docker compose ps
EOF

Security notes

Never commit private keys. Store them only in GitHub Secrets and on your machine with chmod 600.

Rotate tokens/keys if accidentally exposed.

Enable MFA on AWS, GitHub, and Docker Hub.

Restrict Security Groups (SSH from your IP only).

Troubleshooting

Actions “Login to Docker Hub” fails → verify DOCKERHUB_USERNAME/DOCKERHUB_TOKEN secrets; token must have write access.

Build & push fails → re-run; ensure Dockerfiles exist at vote/, result/, worker/.

Deploy step fails → confirm EC2_HOST, EC2_USER, EC2_SSH_KEY secrets and that you can SSH:

ssh -i ~/.ssh/<your-key> ubuntu@<EC2_HOST> 'uname -a && whoami'


EC2 containers unhealthy → check logs: docker compose logs -f <service>.
