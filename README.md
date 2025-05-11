# 🚀 Cloud-Based Web Application Deployment Guide

This guide explains how to deploy a full-stack web application on AWS using:

- **Frontend**: React (or similar) on AWS Elastic Beanstalk
- **Backend**: Node.js in Docker on EC2
- **Database**: Amazon RDS (MySQL)
- **File Storage**: Amazon S3 (for image uploads)

---

## 📝 Prerequisites

- AWS Account
- GitHub repository for frontend and backend
- Node.js and Docker installed locally
- AWS CLI configured (`aws configure`)

---

## 📁 Folder Structure

```
project-root/
├── backend/        # Node.js app
├── frontend/       # React/Vue/Angular app
└── README.md
```

---

## 🐳 Backend Deployment (Node.js in Docker on EC2)

### 1. Dockerize the Backend

In `/backend`, create a `Dockerfile`:

```Dockerfile
FROM node:18
WORKDIR /usr/src/app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 3000
CMD ["node", "index.js"]
```

Build & test locally:

```bash
docker build -t backend-app .
docker run -p 3000:3000 backend-app
```

---

### 2. Launch EC2 & Install Docker

- Instance type: `t2.micro` (Free Tier)
- Security Group: Open ports 22 (SSH), 3000 (API)
- SSH into EC2:

```bash
ssh -i your-key.pem ec2-user@your-ec2-public-ip
```

Install Docker:

```bash
sudo yum update -y
sudo yum install docker -y
sudo service docker start
sudo usermod -aG docker ec2-user
```

Log out and log back in, then:

```bash
docker --version
```

---

### 3. Deploy Backend on EC2

```bash
git clone https://github.com/yourusername/yourrepo.git
cd yourrepo/backend
docker build -t backend-app .
docker run -d -p 3000:3000 backend-app
```

Test: `http://<EC2_PUBLIC_IP>:3000`

---

## 🌐 Frontend Deployment (Elastic Beanstalk)

### 1. Prepare React App for Deployment

```bash
cd frontend
npm run build
```

### 2. Initialize Beanstalk App

Install EB CLI:

```bash
pip install awsebcli
```

Initialize:

```bash
eb init -p node.js frontend-app-name
eb create frontend-env
```

Deploy:

```bash
eb deploy
```

Visit: `http://<elasticbeanstalk-url>.elasticbeanstalk.com`

---

## 🗄️ Database Setup (Amazon RDS MySQL)

### 1. Launch RDS Instance

- Engine: MySQL
- DB Instance: `db.t2.micro` (Free Tier)
- Enable public access
- Security Group: Open port 3306 only to EC2 instance

### 2. Connect & Configure

Use connection string in your backend:

```js
const mysql = require('mysql2');
const connection = mysql.createConnection({
  host: 'your-db-endpoint',
  user: 'admin',
  password: 'yourpassword',
  database: 'yourdbname'
});
```

---

## 📦 File Uploads (Amazon S3)

### 1. Create S3 Bucket

- Name: `your-bucket-name`
- Enable CORS and versioning (optional)

### 2. IAM Role or User Policy

Use the following policy for S3 access:

```json
{
  "Effect": "Allow",
  "Action": ["s3:PutObject", "s3:GetObject"],
  "Resource": "arn:aws:s3:::your-bucket-name/*"
}
```

### 3. Upload Code (Node.js Example)

```js
const AWS = require('aws-sdk');
const s3 = new AWS.S3();

s3.upload({
  Bucket: 'your-bucket-name',
  Key: 'uploads/filename.jpg',
  Body: fileBuffer,
  ACL: 'public-read'
}, (err, data) => {
  if (err) console.error(err);
  else console.log(data.Location);
});
```

---

## 🔐 Security Best Practices

- IAM Roles:
  - EC2 role for accessing S3 and RDS
- Security Groups:
  - EC2: Ports 22, 3000
  - RDS: Port 3306 (from EC2 SG only)
- HTTPS:
  - Use ACM + Load Balancer for HTTPS (optional)
- Least Privilege:
  - Limit IAM roles to specific services/resources

---

## ✅ Final Checklist

- [x] Backend on EC2 (Docker)
- [x] Frontend on Elastic Beanstalk
- [x] RDS MySQL Database connected
- [x] S3 Bucket for image uploads
- [x] IAM Roles and Policies configured
- [x] Security Groups set correctly
- [x] GitHub repo with code
- [x] Live demo URLs

---

## 📎 Useful Links

- [AWS EC2](https://console.aws.amazon.com/ec2/)
- [AWS Elastic Beanstalk](https://console.aws.amazon.com/elasticbeanstalk/)
- [AWS RDS](https://console.aws.amazon.com/rds/)
- [AWS S3](https://s3.console.aws.amazon.com/s3/)
- [IAM Management](https://console.aws.amazon.com/iam/)
