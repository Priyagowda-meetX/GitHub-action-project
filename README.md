# GitHub Actions CI/CD Pipeline Documentation
# 1. Introduction
This document explains the setup, workflow, and functioning of a CI/CD pipeline using GitHub Actions for deploying containerized applications to AWS ECS (Elastic Container Service).
# The process involves:
  * Continuous Integration (CI): Building, testing, and preparing the application.
  * Continuous Deployment (CD): Pushing Docker images to AWS ECR and updating ECS tasks automatically after a pull request (PR) is merged.
# 2. System Overview
  * Source Code Repository: GitHub
  * CI/CD Engine: GitHub Actions
  * Cloud Provider: Amazon Web Services (AWS)
  * Container Registry: Amazon Elastic Container Registry (ECR)
  * Compute Platform: Amazon Elastic Container Service (ECS)
  * Notification Service: Outlook (for email notifications)

## Flow-of-GitHub-Action
![Flow of GitHub Action Diagram](Flow-of-GitHub-Actions.png) 
# 3. Workflow Trigger
  * The workflow is triggered automatically upon PR Merge into the main branch (or a specified branch).
   
# 4. Secrets Management
In GitHub Repository → Settings → Secrets → Actions:

* AWS_ACCESS_KEY_ID: Your AWS IAM User Access Key ID
* AWS_SECRET_ACCESS_KEY: Your AWS IAM User Secret Access Key

These secrets are retrieved in the pipeline to authenticate against AWS services.

# 5. GitHub Actions Workflow Stages
 The CI/CD pipeline is split into two main stages:
 ## 5.1 Continuous Integration (CI)
 
 # Step 1: Check Status
  *  Action: GitHub Action
  *  Purpose: Verify Git commit status.
  *  Details: Ensure all previous tests (unit tests, integration tests) pass before proceeding.
  *  Outcome: Pipeline continues only if status is successful.
    
 # Step 2: Checkout
  * Action: GitHub Action
 * Purpose: Checkout source code into the GitHub runner.
 * Details: Allows workflow to access the repository content.
   
---
<pre> '''yaml 
 - name: Checkout Repository
   uses: actions/checkout@v3  ''' </pre>


