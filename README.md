# ECS Deployment via CodePipeline (Rolling & Blue/Green)

## 📋 Prerequisites

- **ECR repository**: `$ECR_REPO_NAME`  
- **ECS Cluster + Service** (Fargate or EC2) behind an **ALB**  
- **(For Blue/Green)**:  
  - CodeDeploy application of type `ECS`  
  - Deployment group wired to your ECS service & ALB target groups  
- **IAM Roles**:
  - **CodeBuild**:  
    - ECR (push/pull)  
    - ECS (Describe/Register)  
    - S3 artifacts  
    - CloudWatch Logs  
  - **CodeDeploy**: permissions to update ECS service / target groups  
  - **Task roles**:  
    - `executionRoleArn` → ECR/Logs access  
    - `taskRoleArn` → your app’s AWS access  

---

## 🚀 Pipeline Stages

### 1. Source
- Pulls code from your Git provider.

### 2. Build (CodeBuild)
- Runs `buildspec.yml`  
- Logs into ECR  
- Builds & pushes Docker image (`latest` + short `SHA`)  
- Resolves the current Task Definition ARN  
- Generates:
  - `appspec.yaml` (for CodeDeploy ECS)  
  - `imagedefinitions.json` (for ECS rolling)  
  - `taskdef.json` (templated Task Definition)  

### 3. Deploy
- **Rolling update (ECS)** → uses `imagedefinitions.json` artifact  
- **Blue/Green/Canary (CodeDeploy)** → uses `appspec.yaml` + TaskDef ARN  

---

## ⚙️ Environment Variables (CodeBuild)

| Name | Example | Purpose |
|------|---------|---------|
| `AWS_ACCOUNT_ID` | `123456789012` | ECR registry/account |
| `AWS_REGION` | `us-east-1` | Region for AWS calls |
| `ECR_REPO_NAME` | `my-service` | ECR repo name |
| `TASK_DEFINITION_FAMILY` | `my-svc-task` | ECS task family name |
| `CONTAINER_NAME` | `web` | Container name in TaskDef |
| `SERVICE_PORT` | `80` | Container/ALB port |
| `ECS_ROLE` | `arn:aws:iam::…:role/ecsTaskExecutionRole` | Execution role ARN |
| `ECS_TASK_ROLE` | `arn:aws:iam::…:role/myAppTaskRole` | App task role ARN |
| `CPU_UNITS` | `512` | Fargate CPU |
| `MEMORY_UNITS` | `1024` | Fargate memory (MiB) |

---

## 🔄 Optional – Register New Task Definition

If you want CodeDeploy to deploy a brand-new Task Definition that includes the **new image**:

```bash
aws ecs register-task-definition --cli-input-json file://taskdef.json
