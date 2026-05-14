# Server Deploy using GitHub Actions + Docker

## Overview

This project demonstrates a simple CI/CD pipeline using:

* GitHub Actions
* SSH
* Docker
* Ubuntu Server

Whenever code is pushed to the `main` branch, GitHub Actions will:

1. Connect to the server using SSH
2. Clone the repository
3. Build a Docker image
4. Run the Docker container automatically

---

# Project Structure

```text
serverdeploy/
│
├── .github/
│   └── workflows/
│       └── blank.yml
│
├── Dockerfile
├── index.html
└── README.md
```

---

# Technologies Used

* GitHub Actions
* Docker
* Ubuntu Linux
* SSH
* Nginx

---

# Prerequisites

Before using this project, make sure you have:

* Ubuntu server
* Docker installed
* Git installed
* GitHub repository
* SSH access to server

---

# Install Docker on Ubuntu

```bash
sudo apt update

sudo apt install docker.io -y

sudo systemctl start docker

sudo systemctl enable docker
```

Check Docker:

```bash
docker --version
```

---

# Generate SSH Key

Run:

```bash
ssh-keygen -t ed25519
```

Files created:

```text
~/.ssh/id_ed25519
~/.ssh/id_ed25519.pub
```

---

# Add Public Key to Server

View public key:

```bash
cat ~/.ssh/id_ed25519.pub
```

Copy the output.

Open server:

```bash
nano ~/.ssh/authorized_keys
```

Paste the public key.

Set permissions:

```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
```

Restart SSH:

```bash
systemctl restart ssh
```

---

# Add GitHub Secrets

Go to:

```text
Repository
→ Settings
→ Secrets and variables
→ Actions
```

Create secrets:

| Secret Name     | Value                 |
| --------------- | --------------------- |
| HOST            | Server Public IP      |
| SSH_PRIVATE_KEY | Content of id_ed25519 |

---

# GitHub Actions Workflow

Create file:

```text
.github/workflows/blank.yml
```

Add:

```yaml
name: Deploy Pipeline

on:
  push:
    branches: [ main ]

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Setup SSH
        run: |
          mkdir -p ~/.ssh
          echo "${{ secrets.SSH_PRIVATE_KEY }}" > ~/.ssh/id_ed25519
          chmod 600 ~/.ssh/id_ed25519

          ssh-keyscan -H ${{ secrets.HOST }} >> ~/.ssh/known_hosts

      - name: Deploy to server
        run: |
          ssh -i ~/.ssh/id_ed25519 -o StrictHostKeyChecking=no root@${{ secrets.HOST }} << EOF

          set -euxo pipefail

          rm -rf /tmp/serverdeploy

          mkdir -p /tmp/serverdeploy

          cd /tmp/serverdeploy

          git clone https://github.com/Kavin-dev54/serverdeploy.git

          cd serverdeploy

          docker build -t serverdeploy .

          docker stop serverdeploy || true
          docker rm serverdeploy || true

          docker run -d -p 1111:80 --name serverdeploy serverdeploy

          echo "Deployment completed successfully"

          EOF
```

---

# Docker Commands Used

Build image:

```bash
docker build -t serverdeploy .
```

Run container:

```bash
docker run -d -p 1111:80 --name serverdeploy serverdeploy
```

List containers:

```bash
docker ps
```

Stop container:

```bash
docker stop serverdeploy
```

Remove container:

```bash
docker rm serverdeploy
```

---

# Access Application

Open browser:

```text
http://SERVER_IP:1111
```

Example:

```text
http://65.1.xx.xx:1111
```

---

# Common Errors

## Permission denied (publickey)

Cause:

* Wrong SSH key
* Public key not added
* Wrong GitHub secret

Fix:

* Add correct public key to `authorized_keys`
* Add correct private key to GitHub secrets

---

## Git clone asking username/password

Cause:

* Private repository

Fix:

* Make repository public
  OR
* Configure GitHub SSH deploy keys

---

## Docker command not found

Install Docker:

```bash
sudo apt install docker.io -y
```

---

# Useful Commands

Check Docker containers:

```bash
docker ps -a
```

Check logs:

```bash
docker logs serverdeploy
```

Connect to server:

```bash
ssh root@SERVER_IP
```

---

# CI/CD Flow

```text
Developer Push Code
        ↓
GitHub Actions Trigger
        ↓
SSH Connect to Server
        ↓
Clone Repository
        ↓
Docker Build
        ↓
Docker Run
        ↓
Application Live
```

---

# Author

Kavin
Junior DevOps Engineer

