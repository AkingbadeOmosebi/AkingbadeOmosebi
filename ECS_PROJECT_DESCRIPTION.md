# AWS ECS Multi-Environment DevSecOps Platform

## Project Overview

Build a production-grade, multi-environment infrastructure platform on AWS using ECS Fargate to deploy a containerized 3-tier application. The platform demonstrates enterprise-level DevSecOps practices, Blue/Green deployments, Web Application Firewall (WAF) security, and complete CI/CD automation with environment promotion workflows.

---

## Architecture Summary

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              AWS CLOUD                                       │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐                      │
│  │     DEV     │    │   STAGING   │    │    PROD     │                      │
│  │ Environment │───▶│ Environment │───▶│ Environment │                      │
│  └─────────────┘    └─────────────┘    └─────────────┘                      │
│         │                  │                  │                              │
│         ▼                  ▼                  ▼                              │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │                         VPC (Multi-AZ)                               │    │
│  │  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────────┐  │    │
│  │  │  Public Subnet  │  │  Public Subnet  │  │   Public Subnet     │  │    │
│  │  │   (ALB, NAT)    │  │   (ALB, NAT)    │  │    (ALB, NAT)       │  │    │
│  │  └────────┬────────┘  └────────┬────────┘  └─────────┬───────────┘  │    │
│  │           │                    │                      │              │    │
│  │  ┌────────▼────────┐  ┌────────▼────────┐  ┌─────────▼───────────┐  │    │
│  │  │ Private Subnet  │  │ Private Subnet  │  │  Private Subnet     │  │    │
│  │  │  (ECS Tasks)    │  │  (ECS Tasks)    │  │   (ECS Tasks)       │  │    │
│  │  └────────┬────────┘  └────────┬────────┘  └─────────┬───────────┘  │    │
│  │           │                    │                      │              │    │
│  │  ┌────────▼────────┐  ┌────────▼────────┐  ┌─────────▼───────────┐  │    │
│  │  │ Private Subnet  │  │ Private Subnet  │  │  Private Subnet     │  │    │
│  │  │   (Database)    │  │   (Database)    │  │    (Database)       │  │    │
│  │  └─────────────────┘  └─────────────────┘  └─────────────────────┘  │    │
│  └─────────────────────────────────────────────────────────────────────┘    │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Components to Build

### 1. Application Layer

| Component | Description |
|-----------|-------------|
| **Frontend** | Containerized web application (React/static site) served via Nginx |
| **Backend** | Containerized API service (Node.js or Python) |
| **Database** | RDS instance or DynamoDB (optional, can be mocked for demo) |
| **Container Registry** | AWS ECR with lifecycle policies (retain last 10 images) |

### 2. Infrastructure Layer (Terraform)

| Component | Description |
|-----------|-------------|
| **VPC** | Multi-AZ (2-3 availability zones), public/private subnets, NAT Gateway, Internet Gateway |
| **ECS Cluster** | Fargate capacity provider, Container Insights enabled |
| **ECS Services** | Task definitions with Blue/Green deployment controller |
| **Application Load Balancer** | HTTPS listener, target groups for Blue/Green, health checks |
| **Auto Scaling** | Target tracking policies for CPU (70%) and Memory (80%) |
| **ECR Repositories** | Separate repos for frontend and backend, image scanning enabled |

### 3. Security Layer

| Component | Description |
|-----------|-------------|
| **AWS WAF** | Attached to ALB with custom rule sets |
| **ACM** | TLS certificates with DNS validation via Route53 |
| **Route53** | Hosted zone with environment-specific subdomains |
| **Security Groups** | Least-privilege access between tiers |
| **IAM Roles** | Task execution roles, task roles, OIDC federation |

### 4. CI/CD Layer

| Component | Description |
|-----------|-------------|
| **GitHub Actions** | Multi-stage DevSecOps pipeline |
| **OIDC Federation** | Keyless authentication to AWS (no stored credentials) |
| **CodeDeploy** | Blue/Green deployment orchestration for ECS |
| **Infracost** | Cost estimation on every pull request |

### 5. Observability Layer

| Component | Description |
|-----------|-------------|
| **CloudWatch Logs** | Centralized logging for ECS tasks |
| **CloudWatch Metrics** | Container Insights, custom dashboards |
| **CloudWatch Alarms** | Task failures, high CPU/memory, 5xx errors, WAF blocks |
| **ALB Access Logs** | Stored in S3 for audit trail |
| **WAF Logs** | CloudWatch log group for security analysis |

---

## Multi-Environment Strategy

### Environment Specifications

| Configuration | Dev | Staging | Prod |
|---------------|-----|---------|------|
| **ECS Task Count** | 1 | 2 | 3+ |
| **Task Size (CPU/Memory)** | 256/512 | 512/1024 | 1024/2048 |
| **Auto Scaling** | Disabled | Min 1, Max 3 | Min 2, Max 10 |
| **WAF Mode** | Count (log only) | Count (log only) | Block |
| **Deletion Protection** | None | Enabled | Strict |
| **Deployment** | Auto on push | Auto after dev | Manual approval |
| **Domain** | dev.example.com | staging.example.com | example.com |

### State Management

- Separate Terraform state files per environment
- S3 backend with DynamoDB state locking
- State file path: `s3://tfstate-bucket/{env}/terraform.tfstate`

---

## WAF Rules Specification

### Rule Set

| Rule | Type | Action | Purpose |
|------|------|--------|---------|
| **Rate Limiting** | Custom | Block | 2000 requests/5 min per IP — prevents DDoS and brute force |
| **SQL Injection** | AWS Managed | Block | AWSManagedRulesSQLiRuleSet — OWASP protection |
| **XSS Protection** | AWS Managed | Block | AWSManagedRulesCommonRuleSet — cross-site scripting prevention |
| **Bad Bot Control** | AWS Managed | Block | AWSManagedRulesBotControlRuleSet — automated threat protection |
| **IP Whitelist** | Custom | Allow | Staging: restrict to team IPs only |
| **Geo Blocking** | Custom | Block | (Optional) Block requests from specific countries |

### WAF Logging

- All requests logged to CloudWatch Logs
- Retention: 30 days
- Metrics: Blocked requests, allowed requests, rule matches

---

## Blue/Green Deployment Specification

### Deployment Flow

```
┌─────────────────────────────────────────────────────────────────┐
│                    BLUE/GREEN DEPLOYMENT                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  1. New version pushed to ECR                                   │
│              │                                                   │
│              ▼                                                   │
│  2. CodeDeploy creates GREEN task set                           │
│              │                                                   │
│              ▼                                                   │
│  3. Health checks pass on GREEN                                 │
│              │                                                   │
│              ▼                                                   │
│  4. Traffic shift begins:                                       │
│     ├── 10% to GREEN (5 min wait)                               │
│     ├── 50% to GREEN (5 min wait)                               │
│     └── 100% to GREEN                                           │
│              │                                                   │
│              ▼                                                   │
│  5. BLUE task set terminated                                    │
│              │                                                   │
│              ▼                                                   │
│  6. Deployment complete                                         │
│                                                                  │
│  ROLLBACK: Automatic if health checks fail at any stage         │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Configuration

- Deployment controller: `CODE_DEPLOY`
- Deployment config: `CodeDeployDefault.ECSLinear10PercentEvery1Minutes` or custom
- Health check grace period: 60 seconds
- Termination wait time: 5 minutes

---

## CI/CD Pipeline Specification

### Pipeline Stages

```
┌─────────────────────────────────────────────────────────────────┐
│                    DEVSECOPS PIPELINE                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  STAGE 1: Code Quality & Security Scans (Parallel)              │
│  ┌─────────────┬─────────────┬─────────────┬─────────────┐      │
│  │  Gitleaks   │ SonarCloud  │   Trivy     │    OWASP    │      │
│  │  (secrets)  │ (quality)   │ (container) │ (deps)      │      │
│  └─────────────┴─────────────┴─────────────┴─────────────┘      │
│              │                                                   │
│              ▼ (all must pass)                                   │
│  STAGE 2: Build & Push                                          │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │  Docker build → Tag with git SHA → Push to ECR          │    │
│  └─────────────────────────────────────────────────────────┘    │
│              │                                                   │
│              ▼                                                   │
│  STAGE 3: Deploy to Dev (automatic)                             │
│              │                                                   │
│              ▼ (on success)                                      │
│  STAGE 4: Deploy to Staging (automatic)                         │
│              │                                                   │
│              ▼ (manual approval required)                        │
│  STAGE 5: Deploy to Prod                                        │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Infrastructure Pipeline

```
┌─────────────────────────────────────────────────────────────────┐
│                 TERRAFORM PIPELINE                               │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ON PULL REQUEST:                                               │
│  ┌─────────────┬─────────────┬─────────────┬─────────────┐      │
│  │  terraform  │  terraform  │   tfsec     │  Infracost  │      │
│  │    fmt      │   validate  │   (scan)    │  (cost)     │      │
│  └─────────────┴─────────────┴─────────────┴─────────────┘      │
│              │                                                   │
│              ▼                                                   │
│  terraform plan → Comment on PR                                 │
│                                                                  │
│  ON MERGE TO MAIN:                                              │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │  terraform apply (dev) → apply (staging) → apply (prod) │    │
│  │                                    ▲                     │    │
│  │                            manual approval               │    │
│  └─────────────────────────────────────────────────────────┘    │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### OIDC Configuration

- GitHub OIDC Provider registered in AWS IAM
- IAM Role: `github-actions-ecs-deploy-role`
- Trust policy: Scoped to specific repository and branches
- Permissions: ECR push, ECS deploy, CodeDeploy, S3 state access

---

## Terraform Module Structure

```
terraform/
├── modules/
│   ├── vpc/
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   ├── outputs.tf
│   │   └── README.md
│   ├── ecr/
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   └── outputs.tf
│   ├── ecs-cluster/
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   └── outputs.tf
│   ├── ecs-service/
│   │   ├── main.tf          (task definition, service, auto-scaling)
│   │   ├── variables.tf
│   │   └── outputs.tf
│   ├── alb/
│   │   ├── main.tf          (ALB, listeners, target groups)
│   │   ├── variables.tf
│   │   └── outputs.tf
│   ├── waf/
│   │   ├── main.tf          (WAF ACL, rules, logging)
│   │   ├── rules.tf         (individual rule definitions)
│   │   ├── variables.tf
│   │   └── outputs.tf
│   ├── acm/
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   └── outputs.tf
│   ├── route53/
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   └── outputs.tf
│   ├── codedeploy/
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   └── outputs.tf
│   ├── oidc/
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   └── outputs.tf
│   └── monitoring/
│       ├── main.tf          (CloudWatch dashboards, alarms)
│       ├── variables.tf
│       └── outputs.tf
│
└── environments/
    ├── dev/
    │   ├── main.tf          (calls modules with dev values)
    │   ├── terraform.tfvars
    │   ├── backend.tf
    │   └── outputs.tf
    ├── staging/
    │   ├── main.tf
    │   ├── terraform.tfvars
    │   ├── backend.tf
    │   └── outputs.tf
    └── prod/
        ├── main.tf
        ├── terraform.tfvars
        ├── backend.tf
        └── outputs.tf
```

---

## Auto Scaling Specification

### ECS Service Auto Scaling

| Metric | Target | Scale Out | Scale In |
|--------|--------|-----------|----------|
| CPU Utilization | 70% | +1 task | -1 task |
| Memory Utilization | 80% | +1 task | -1 task |
| Cooldown Period | - | 60 seconds | 120 seconds |

### Environment Limits

| Environment | Min Tasks | Max Tasks |
|-------------|-----------|-----------|
| Dev | 1 | 1 |
| Staging | 1 | 3 |
| Prod | 2 | 10 |

---

## Documentation Requirements

### Required Documentation Files

| File | Content |
|------|---------|
| `README.md` | Project overview, architecture diagram, quick start, badges, cost estimate |
| `docs/ARCHITECTURE.md` | Detailed system design, data flow, security architecture |
| `docs/DEPLOYMENT.md` | Step-by-step deployment guide, environment promotion, rollback |
| `docs/WAF_RULES.md` | Each WAF rule explained with rationale |
| `docs/RUNBOOK.md` | Incident response procedures, common issues, troubleshooting |

### Required Screenshots

| Screenshot | Purpose |
|------------|---------|
| Pipeline run (all stages green) | Prove CI/CD works |
| Blue/Green deployment in progress | Prove deployment strategy |
| WAF dashboard with blocked requests | Prove security rules work |
| CloudWatch metrics/dashboard | Prove observability |
| All 3 environments running | Prove multi-env works |
| Infracost PR comment | Prove cost awareness |

---

## Success Criteria

- [ ] All 3 environments deploy successfully via Terraform
- [ ] Application accessible via HTTPS on all environment domains
- [ ] WAF blocks simulated SQL injection and XSS attempts
- [ ] Blue/Green deployment completes with traffic shifting
- [ ] Auto-scaling triggers on load (can be simulated)
- [ ] Pipeline runs end-to-end with all security scans passing
- [ ] Prod deployment requires manual approval
- [ ] Infracost comments appear on Terraform PRs
- [ ] CloudWatch alarms configured and functional
- [ ] All documentation complete with diagrams
- [ ] Screenshots prove working system
- [ ] 25-40 meaningful commits showing iteration

---

## Technology Stack

| Category | Technology |
|----------|------------|
| **Cloud** | AWS |
| **Container Orchestration** | ECS Fargate |
| **Container Registry** | ECR |
| **Infrastructure as Code** | Terraform |
| **CI/CD** | GitHub Actions |
| **Deployment Strategy** | Blue/Green via CodeDeploy |
| **Load Balancing** | Application Load Balancer |
| **DNS** | Route53 |
| **TLS Certificates** | AWS Certificate Manager |
| **Web Application Firewall** | AWS WAF |
| **Authentication** | OIDC Federation |
| **Cost Management** | Infracost |
| **Security Scanning** | Gitleaks, SonarCloud, Trivy, OWASP, tfsec |
| **Monitoring** | CloudWatch (Logs, Metrics, Alarms, Container Insights) |

---

## Target Outcome

Upon completion, this project demonstrates:

1. **Multi-environment infrastructure management** — Dev, Staging, Prod with proper isolation
2. **Enterprise security practices** — WAF, TLS, OIDC, least-privilege IAM
3. **Zero-downtime deployments** — Blue/Green with automatic rollback
4. **DevSecOps maturity** — Security scanning at every pipeline stage
5. **Cost awareness** — Infracost integration, right-sized resources
6. **Operational readiness** — Monitoring, alerting, logging, documentation

This platform is designed to signal **Mid-Senior DevOps/Platform Engineer** level capabilities to hiring managers, particularly in the German market where security, stability, and documentation quality are highly valued.
