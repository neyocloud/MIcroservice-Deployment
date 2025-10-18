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
