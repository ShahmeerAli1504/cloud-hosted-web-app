# Full Stack web Hosted on Cloud(AWS)

This guide walks you through deploying a **SERN (SQL, Express, React, Node.js)** stack application on **AWS**, using:

- **EC2** for backend (Dockerized Node.js/Express)
- **RDS (PostgreSQL)** for database
- **S3** for frontend hosting (React build)
- **IAM** for managing permissions

---

## Project Structure

full-stack-crud-auth-web-app.sern/
├── frontend/ # React frontend
├── backend/ # Express backend (Node.js)

## Prerequisites

- AWS account with access to EC2, RDS, S3, and IAM
- Git, Docker, Node.js, NPM installed on EC2
- PostgreSQL credentials ready

---

## Step-by-Step Deployment

---

### 1. Set Up RDS (PostgreSQL)

- Go to **RDS > Create Database**

  - Engine: **PostgreSQL**
  - DB instance identifier: `sern_db`
  - Username: `root`
  - Password: `yourpassword`
  - DB class: `db.t3.micro`
  - Enable **Public Access**
  - In **VPC Security Groups**, add EC2’s security group, and allow **port 5432**

- After creation, copy the **Endpoint** (e.g., `serndb.xxxxxx.rds.amazonaws.com`)

---

### 2. Launch EC2 Instance (Amazon Linux 2)

- Open EC2 dashboard > Launch instance
- Choose **Amazon Linux 2**
- Allow inbound traffic on:
  - **Port 22** (SSH)
  - **Port 3000** (Backend)
  - Optional: **Port 80/443** (if serving frontend via EC2 later)

---

### 3. Backend Setup (Dockerized on EC2)

SSH into EC2:

```bash
ssh -i Fatima.pem ec2-user@<EC2-PUBLIC-IP>
```

### Install Git & Docker:

sudo yum update -y
sudo yum install git -y
sudo yum install docker -y
sudo service docker start
sudo usermod -aG docker ec2-user

## Log out and back in for Docker permissions to apply.

### Clone and navigate to server:

git clone https://github.com/Refaat0/full-stack-crud-auth-web-app.sern.git
cd full-stack-crud-auth-web-app.sern/server

---

### Create .env in backend/:

DB_HOST=serndb.xxxxxx.rds.amazonaws.com
DB_USER=root
DB_PASSWORD=yourpassword
DB_DATABASE=sern_db
PORT=3333

---

### Create Dockerfile:

FROM node:16
WORKDIR /app
COPY . .
RUN npm install
CMD ["npm", "start"]

---

### Build & run the Docker container:

docker build -t sern-backend .
docker run -d -p 3000:3333 --env-file .env sern-backend

---

### Check logs:

docker ps
docker logs <container_id>

### Frontend Setup (React on S3):

Navigate to frontend/ folder on your local machine
Update .env
Build the React app
Zip and upload:

### Create an S3 Bucket:

        Enable static website hosting
        Upload the build folder or build.zip
        Set index document: index.html
        Make files public

### EC2 Security Group:

#### Inbound Rules:

TCP port 22 open to My IP – For SSH access to EC2.

TCP port 3000 open to 0.0.0.0/0 – For accessing the backend from the browser or frontend.

TCP port 80/443 open to 0.0.0.0/0 – For general web traffic (optional if you are hosting frontend from S3).

#### Outbound Rules:

All traffic (default) – So the EC2 instance can send requests to the internet, RDS, etc.
