# AWS 3-Tier Application Deployment Project
## Realised april 2025 after completing the AWS Cloud Foundations course by AWS ACADEMY 
A comprehensive cloud infrastructure project focused on deploying a secure, scalable 3-tier web application architecture on Amazon Web Services (AWS). This project demonstrates enterprise-level cloud architecture design, implementation, and management skills.

# Architecture Overview 

<img width="790" height="606" alt="image" src="https://github.com/user-attachments/assets/48d293e6-b9a0-412b-908f-e8a7611a70e8" />

## 3-Tier Application Stack 
* Frontend : JavaScript
* Backend : Python flask 
* Database : PostgreSQL(managed with Amazon RDS)

# 🛠️ Technologies Used
## AWS Services
* Compute: EC2, Auto Scaling Groups, Application Load Balancer
* Network: VPC, Subnets, Internet Gateway, NAT Gateway, Security Groups
* Database: Amazon RDS (Multi-AZ)
* Storage & CDN: S3, CloudFront
* Security: Certificate Manager, IAM
* Monitoring: CloudWatch, CloudTrail, SNS
## Deployement steps 
# Step 1: VPC and Network Components Configuration

* Created a VPC with CIDR range (10.0.0.0/16)
* Deployed 8 subnets across 2 availability zones:

  2 public subnets (one in each AZ)
  
  6 private subnets (3 per AZ: frontend/backend/RDS)
  
<img width="900" height="385" alt="image" src="https://github.com/user-attachments/assets/13150fce-bacc-4645-b32c-0c53c80d4828" />



* Attached Internet Gateway (IGW) to VPC
* Deployed 2 NAT Gateways (one per AZ) in public subnets
* Configured route tables for public and private subnets

<img width="883" height="410" alt="image" src="https://github.com/user-attachments/assets/cfbf215c-59bb-42ad-ad35-b912b4238ee2" />



# Step 2: Security Groups Configuration

* SG-LB: HTTP/HTTPS access from internet (0.0.0.0/0)
* SG-FE: Traffic from SG-LB only (ports 80/443)
* SG-BE: Traffic from SG-FE only (port 8080)
* SG-DB: Traffic from SG-BE only (port 3306)
* SG-Bastion: SSH access from fixed IP (port 22)

<img width="916" height="418" alt="image" src="https://github.com/user-attachments/assets/411f3c35-a1ae-429f-9260-d64a0fe076a8" />

# Step 3: EC2 Resources Deployment

1) Frontend EC2 Instances: Deployed 2 instances in separate private subnets
* EC2-Frontend-A in Availability Zone A private subnet
* EC2-Frontend-B in Availability Zone B private subnet

2) Backend EC2 Instances: Deployed 2 instances in separate private subnets
* EC2-Backend-A in Availability Zone A private subnet
* EC2-Backend-B in Availability Zone B private subnet
  
<img width="887" height="407" alt="image" src="https://github.com/user-attachments/assets/728a9173-9974-437e-bdfa-da3da19a86b6" />


3) Bastion Host: Deployed 1 EC2 instance in public subnet to connect to private subnet instances
   
4) Application Load Balancers Configuration:
* ALB for frontend traffic management
* ALB for backend traffic management

<img width="918" height="427" alt="image" src="https://github.com/user-attachments/assets/f2038bfa-8976-4233-ac88-4d7c77a202da" />


5) Auto Scaling Groups Configuration:
* Frontend ASG in frontend private subnets
* Backend ASG in backend private subnets

<img width="925" height="424" alt="image" src="https://github.com/user-attachments/assets/ba5be7f5-b0c8-4546-a022-4ec104a8ab15" />

6) Scaling Rules: Defined scaling policies based on metrics:
* CPU Utilization thresholds
* Request count metrics

1 ASG for the frontend with :  
scaling limite = 3 
> 50% CPU usage for the Scale In 
< 50% CPU usage for the Scale Out 

1 ASG for the backend with : 
scaling limite = 3 
> 50% CPU usage for the Scale In
< 50% CPU usage for the Scale Out 

7) Application Load Balancers:
* ALB-Frontend → Target Group TG-Frontend
* ALB-Backend → Target Group TG-Backend

# Step 4: Amazon RDS Database Deployment

* MySQL/PostgreSQL RDS instance in private subnets
* Multi-AZ configuration for high availability
  
<img width="934" height="422" alt="image" src="https://github.com/user-attachments/assets/48b1790f-219f-49bf-b2d6-e8a85a9e9fc7" />

* SG-DB security group attached
* Database 'mydb' created and configured

SSH to the database instance to add your database using the generated key and putty (or any other alternative)

<img width="832" height="561" alt="image" src="https://github.com/user-attachments/assets/1e179759-bac5-4b07-b660-39f47082760c" />


# Step 5: S3 and CloudFront Configuration

* S3 bucket for static file storage
* CloudFront distribution for optimized content delivery

<img width="872" height="198" alt="image" src="https://github.com/user-attachments/assets/1edeba57-ed4d-4254-b6d5-f66dff254bf9" />

UserData script for the frontend to install all requirements and get the frontend files from the S3 bucket

```c
#!/bin/bash

# Met à jour le système et installer nginx
sudo yum update -y
sudo yum install -y nginx

# Crée le dossier web si il n'existe pas
sudo mkdir -p /usr/share/nginx/html

# Télécharge le fichier index.html depuis S3 (bucket public)
sudo wget https://mybucket2025front.s3.amazonaws.com/frontend/index.html -O /usr/share/nginx/html/index.html

# Crée le fichier de configuration nginx
sudo bash -c 'cat > /etc/nginx/conf.d/default.conf <<EOF
server {
    listen 80 default_server;
    listen [::]:80 default_server;

    root /usr/share/nginx/html;
    index index.html;

    location = /health {
        default_type text/plain;
        return 200 "OK";
        access_log off;
    }

    location / {
        try_files \$uri \$uri/ =404;
    }
}
EOF'

# Démarre nginx
sudo systemctl start nginx
sudo systemctl enable nginx
```
UserData script for the backend to install all requirements and get the backend files from the S3 bucket
```c
#!/bin/bash

# 1. Mettre à jour le système
sudo yum update -y

# 2. Installer Python, MySQL client, et outils nécessaires
sudo yum install -y python3 python3-pip mysql wget gcc openssl-devel bzip2-devel libffi-devel python3-devel

# 3. Créer le dossier de l'application backend
sudo mkdir -p /var/www/backend
cd /var/www/backend

# 4. Télécharger app.py depuis ton bucket S3
sudo wget https://mybucket2025front.s3.amazonaws.com/backend/app.py -O app.py

# 5. Créer un environnement virtuel Python
python3 -m venv venv

# 6. Installer les dépendances Flask
source venv/bin/activate
pip install flask pymysql Flask-SQLAlchemy flask-cors python-dotenv cryptography
deactivate

# 7. Créer le fichier .env avec ta configuration MySQL locale
sudo bash -c 'echo "DATABASE_URL=mysql+pymysql://admin:mahdi123@cloudbd.c4dfg7klvxyz.us-east-1.rds.amazonaws.com:3306/myapp" > /var/www/backend/.env'

# 8. Protéger le fichier d'environnement
sudo chmod 600 /var/www/backend/.env
sudo chown ec2-user:ec2-user /var/www/backend/.env

# 9. Créer un service systemd pour lancer Flask avec le serveur intégré
sudo bash -c 'cat > /etc/systemd/system/flask-app.service << EOL
[Unit]
Description=Flask Application
After=network.target

[Service]
User=ec2-user
Group=ec2-user
WorkingDirectory=/var/www/backend
EnvironmentFile=/var/www/backend/.env
ExecStart=/var/www/backend/venv/bin/python3 -m flask run --host=0.0.0.0 --port=8080
Restart=always

[Install]
WantedBy=multi-user.target
EOL'

# 10. Donner les bons droits
sudo chown -R ec2-user:ec2-user /var/www/backend
sudo chmod -R 755 /var/www/backend

# 11. Activer et lancer le service
sudo systemctl daemon-reexec
sudo systemctl daemon-reload
sudo systemctl start flask-app
sudo systemctl enable flask-app
```

# Step 6: Advanced Security Implementation

* Amazon CloudWatch for metrics monitoring and log aggregation
* CloudWatch alarms for critical thresholds (CPU > 70%)

<img width="952" height="451" alt="image" src="https://github.com/user-attachments/assets/c4f5aef7-7da2-4126-ac6b-70ace540da66" />

* RDS automated backups enabled
* AWS CloudTrail for infrastructure action logging
  
<img width="595" height="517" alt="image" src="https://github.com/user-attachments/assets/472c6d81-95df-4c92-ae28-2612e3b11651" />

* AWS Certificate Manager (ACM) for SSL certificates
* SNS
  
<img width="1535" height="570" alt="image" src="https://github.com/user-attachments/assets/2ffc67d7-8161-425b-b15d-895a8fbe0116" />
  
* HTTPS encryption applied to load balancers

# Testing 
<img width="837" height="134" alt="image" src="https://github.com/user-attachments/assets/f1b08637-d83a-4881-8a16-bcea5026b07a" />

<img width="650" height="382" alt="image" src="https://github.com/user-attachments/assets/6bcd5be3-08a2-45bf-a118-687970eeeb45" />

# PS : You can find more details about the deployement in the report PDF added to this project  
