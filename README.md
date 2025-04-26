# GitHub Actions CI/CD Pipeline Documentation
## 1. Introduction
This document explains the setup, workflow, and functioning of a CI/CD pipeline using GitHub Actions for deploying containerized applications to AWS ECS (Elastic Container Service).
## The process involves:
  * Continuous Integration (CI): Building, testing, and preparing the application.
  * Continuous Deployment (CD): Pushing Docker images to AWS ECR and updating ECS tasks automatically after a pull request (PR) is merged.
## 2. System Overview
  * Source Code Repository: GitHub
  * CI/CD Engine: GitHub Actions
  * Cloud Provider: Amazon Web Services (AWS)
  * Container Registry: Amazon Elastic Container Registry (ECR)
  * Compute Platform: Amazon Elastic Container Service (ECS)
  * Notification Service: Outlook (for email notifications)

## Flow-of-GitHub-Action
![Flow of GitHub Action Diagram](Flow-of-GitHub-Actions.png) 
## 3. Workflow Trigger
  * The workflow is triggered automatically upon PR Merge into the main branch (or a specified branch).
   
## 4. Secrets Management
In GitHub Repository → Settings → Secrets → Actions:

* AWS_ACCESS_KEY_ID: Your AWS IAM User Access Key ID
* AWS_SECRET_ACCESS_KEY: Your AWS IAM User Secret Access Key

These secrets are retrieved in the pipeline to authenticate against AWS services.

## 5. GitHub Actions Workflow Stages
 The CI/CD pipeline is split into two main stages:
 ## 5.1 Continuous Integration (CI)
 
 ### Step 1: Check Status
  *  Action: GitHub Action
  *  Purpose: Verify Git commit status.
  *  Details: Ensure all previous tests (unit tests, integration tests) pass before proceeding.
  *  Outcome: Pipeline continues only if status is successful.
    
 ### Step 2: Checkout
  * Action: GitHub Action
 * Purpose: Checkout source code into the GitHub runner.
 * Details: Allows workflow to access the repository content.
   

 ``` yaml 
 - name: Checkout Repository
   uses: actions/checkout@v3
   ```
### Step 3: Configure
 * Action: GitHub Action
 * Purpose: Configure AWS CLI inside the GitHub runner.
 * Details: Uses AWS credentials stored in GitHub secrets.

``` yaml
- name: Configure AWS Credentials
  uses: aws-actions/configure-aws-credentials@v2
  with:
    aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
    aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
    aws-region: your-aws-region
```
### Step 4: Login to ECR
  * Action: GitHub Action
  * Purpose: Authenticate Docker to AWS Elastic Container Registry (ECR).
  * Details: Allow GitHub runner to push images to private ECR repository.

``` yaml
- name: Login to Amazon ECR
  id: login-ecr
  uses: aws-actions/amazon-ecr-login@v2
```

## 5.2 Continuous Deployment (CD)
### Step 5: Build Image

  * Action: GitHub Action
  * Purpose: Build Docker image and push to Amazon ECR.
---
    ``` yaml
    - name: Build, Tag, and Push Docker Image to ECR
      run: |
        docker build -t ${{ env.ECR_REPOSITORY }}:${{ github.sha }} .
        docker tag ${{ env.ECR_REPOSITORY }}:${{ github.sha }} ${{ env.ECR_REGISTRY }}/${{ env.ECR_REPOSITORY }}:${{ github.sha }}
        docker push ${{ env.ECR_REGISTRY }}/${{ env.ECR_REPOSITORY }}:${{ github.sha }}
    
    ```
##### Variables:
   *  ${{ github.sha }} → Commit SHA used for image tagging.
  
### Step 6: Update ECS Task Definition
  
  * Action: GitHub Action
  *  Purpose: Update ECS task definition with the new Docker image tag.
``` yaml
- name: Update ECS Task Definition
  uses: aws-actions/amazon-ecs-render-task-definition@v1
  with:
    task-definition: task-definition.json
    container-name: your-container-name
    image: ${{ env.ECR_REGISTRY }}/${{ env.ECR_REPOSITORY }}:${{ github.sha }}
```
### Step 7: Deploy
  * Action: GitHub Action
  * Purpose: Update ECS service with the new task definition.
``` yaml
- name: Deploy ECS Service
  uses: aws-actions/amazon-ecs-deploy-task-definition@v1
  with:
    service: your-ecs-service-name
    cluster: your-ecs-cluster-name
    task-definition: your-updated-task-definition.json
```

## 6. Notifications
 * After deployment, an email notification is sent through Outlook integration to the stakeholders/developers to inform them about the deployment status.

## 7. Visual Representation (from the provided diagram)
``` sql
GitHub Repository
        |
      PR Merge
        |
    GitHub Workflow
        |
        |--- Check Status (Tests Passed?)
        |--- Checkout Repo
        |--- Configure AWS CLI
        |--- Login to ECR
        |--- Build and Push Docker Image
        |--- Update ECS Task Definition
        |--- Deploy to ECS Service
        |
      Send Outlook Notification
```

## 8. Requirements
* AWS Account (IAM User with permissions for ECR, ECS)
* GitHub Repository with code and Dockerfile
* GitHub Secrets configured
* ECS Cluster and Service pre-created
* ECR Repository pre-created
* Task Definition file template available (JSON)

## 9. Possible Enhancements
* Add Slack/MS Teams notifications.
* Add Rollback strategy if deployment fails.
* Add more environments (Dev, QA, Prod) using branches or environments.
* Use GitHub Environments protection rules.
* Add security scanning before pushing the image.
* Add automated testing step after deployment (Health checks).

# ✅ Conclusion
This GitHub Actions workflow provides a fully automated CI/CD pipeline from source code merge to AWS ECS deployment, along with email notifications for full visibility.







