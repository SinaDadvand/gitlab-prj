# GitLab with Google Cloud Platform Integration - Step-by-Step Guide

A comprehensive hands-on tutorial for learning GitLab basics and integrating with Google Cloud Platform (GCP) free tier.

## Table of Contents
1. [Prerequisites](#prerequisites)
2. [GitLab Basics](#gitlab-basics)
3. [Setting up Google Cloud Platform](#setting-up-google-cloud-platform)
4. [GitLab CI/CD with GCP](#gitlab-cicd-with-gcp)
5. [Hands-on Projects](#hands-on-projects)
6. [Best Practices](#best-practices)
7. [Troubleshooting](#troubleshooting)

## Prerequisites

Before starting this tutorial, ensure you have:
- A computer with internet access
- A Google account
- Basic knowledge of Git commands
- Text editor or IDE (VS Code recommended)
- Basic understanding of command line/terminal

## GitLab Basics

### Step 1: Create a GitLab Account

1. Visit [GitLab.com](https://gitlab.com)
2. Click "Sign up" and create a free account
3. Verify your email address
4. Complete your profile setup

### Step 2: Understanding GitLab Interface

**Key Components:**
- **Projects**: Your repositories
- **Groups**: Collections of projects
- **Issues**: Bug tracking and feature requests
- **Merge Requests**: Code review and collaboration
- **CI/CD**: Continuous Integration/Deployment
- **Wiki**: Documentation
- **Snippets**: Code sharing

### Step 3: Create Your First Project

1. Click "New project" on GitLab dashboard
2. Choose "Create blank project"
3. Fill in project details:
   - Project name: `my-first-gitlab-project`
   - Description: `Learning GitLab with GCP integration`
   - Visibility: Private (recommended for learning)
4. Initialize with README
5. Click "Create project"

### Step 4: Basic Git Operations with GitLab

**Clone your repository:**
```bash
git clone https://gitlab.com/your-username/my-first-gitlab-project.git
cd my-first-gitlab-project
```

**Create a simple file:**
```bash
echo "Hello GitLab!" > hello.txt
git add hello.txt
git commit -m "Add hello.txt file"
git push origin main
```

**Create a branch:**
```bash
git checkout -b feature/add-documentation
echo "# My Project Documentation" > docs.md
git add docs.md
git commit -m "Add documentation"
git push origin feature/add-documentation
```

### Step 5: Working with Merge Requests

1. Go to your GitLab project
2. Click "Merge Requests" → "New merge request"
3. Select source branch: `feature/add-documentation`
4. Select target branch: `main`
5. Add title and description
6. Click "Create merge request"
7. Review changes and merge

## Setting up Google Cloud Platform

### Step 6: Create GCP Account and Project

1. Visit [Google Cloud Console](https://console.cloud.google.com)
2. Sign in with your Google account
3. Accept terms and activate free tier ($300 credit)
4. Create a new project:
   - Project name: `gitlab-integration-demo`
   - Note down the Project ID

### Step 7: Enable Required APIs

1. Go to "APIs & Services" → "Library"
2. Enable the following APIs:
   - Cloud Build API
   - Cloud Run API
   - Container Registry API
   - Compute Engine API
   - Cloud Storage API

### Step 8: Create Service Account

1. Go to "IAM & Admin" → "Service Accounts"
2. Click "Create Service Account"
3. Fill in details:
   - Name: `gitlab-ci-service-account`
   - Description: `Service account for GitLab CI/CD`
4. Grant roles:
   - Cloud Build Editor
   - Cloud Run Admin
   - Storage Admin
   - Compute Admin
5. Click "Done"
6. Click on the created service account
7. Go to "Keys" tab → "Add Key" → "Create new key"
8. Choose JSON format and download the key file
9. **Important**: Keep this file secure and never commit it to version control

### Step 9: Set up Cloud Storage Bucket

1. Go to "Cloud Storage" → "Buckets"
2. Click "Create bucket"
3. Name: `gitlab-demo-bucket-[your-project-id]`
4. Choose region closest to you
5. Use default settings for other options
6. Click "Create"

## GitLab CI/CD with GCP

### Step 10: Configure GitLab Variables

1. Go to your GitLab project
2. Navigate to "Settings" → "CI/CD"
3. Expand "Variables" section
4. Add the following variables:
   - `GCP_PROJECT_ID`: Your GCP project ID
   - `GCP_SERVICE_ACCOUNT_KEY`: Content of your service account JSON key (mark as protected and masked)
   - `GCP_REGION`: Your preferred GCP region (e.g., `us-central1`)

### Step 11: Create GitLab CI/CD Pipeline

Create `.gitlab-ci.yml` in your project root:

```yaml
# GitLab CI/CD Pipeline for GCP Integration
stages:
  - build
  - test
  - deploy

variables:
  DOCKER_DRIVER: overlay2
  GOOGLE_APPLICATION_CREDENTIALS: /tmp/gcp-key.json

before_script:
  - echo $GCP_SERVICE_ACCOUNT_KEY | base64 -d > ${GOOGLE_APPLICATION_CREDENTIALS}
  - gcloud auth activate-service-account --key-file ${GOOGLE_APPLICATION_CREDENTIALS}
  - gcloud config set project $GCP_PROJECT_ID

build:
  stage: build
  image: google/cloud-sdk:latest
  script:
    - echo "Building application..."
    - echo "Build completed successfully"
  artifacts:
    paths:
      - build/
    expire_in: 1 hour

test:
  stage: test
  image: google/cloud-sdk:latest
  script:
    - echo "Running tests..."
    - echo "All tests passed"

deploy_to_gcs:
  stage: deploy
  image: google/cloud-sdk:latest
  script:
    - echo "Deploying to Google Cloud Storage..."
    - echo "Hello from GitLab CI/CD!" > deploy.txt
    - gsutil cp deploy.txt gs://gitlab-demo-bucket-${GCP_PROJECT_ID}/
    - echo "Deployment completed"
  only:
    - main
```

### Step 12: Create a Simple Web Application

Create a simple Node.js application to deploy:

**package.json:**
```json
{
  "name": "gitlab-gcp-demo",
  "version": "1.0.0",
  "description": "Demo app for GitLab-GCP integration",
  "main": "server.js",
  "scripts": {
    "start": "node server.js",
    "test": "echo \"No tests specified\" && exit 0"
  },
  "dependencies": {
    "express": "^4.18.0"
  }
}
```

**server.js:**
```javascript
const express = require('express');
const app = express();
const port = process.env.PORT || 8080;

app.get('/', (req, res) => {
  res.json({
    message: 'Hello from GitLab CI/CD deployed to Google Cloud!',
    timestamp: new Date().toISOString(),
    environment: process.env.NODE_ENV || 'development'
  });
});

app.get('/health', (req, res) => {
  res.json({ status: 'healthy', uptime: process.uptime() });
});

app.listen(port, () => {
  console.log(`Server running on port ${port}`);
});
```

**Dockerfile:**
```dockerfile
FROM node:16-alpine

WORKDIR /app

COPY package*.json ./
RUN npm ci --only=production

COPY . .

EXPOSE 8080

USER node

CMD ["npm", "start"]
```

### Step 13: Enhanced CI/CD Pipeline for Cloud Run

Update your `.gitlab-ci.yml`:

```yaml
stages:
  - build
  - test
  - deploy

variables:
  DOCKER_DRIVER: overlay2
  GOOGLE_APPLICATION_CREDENTIALS: /tmp/gcp-key.json
  IMAGE_NAME: gcr.io/$GCP_PROJECT_ID/gitlab-demo-app

before_script:
  - echo $GCP_SERVICE_ACCOUNT_KEY | base64 -d > ${GOOGLE_APPLICATION_CREDENTIALS}
  - gcloud auth activate-service-account --key-file ${GOOGLE_APPLICATION_CREDENTIALS}
  - gcloud config set project $GCP_PROJECT_ID
  - gcloud auth configure-docker

build:
  stage: build
  image: google/cloud-sdk:latest
  services:
    - docker:dind
  script:
    - docker build -t $IMAGE_NAME:$CI_COMMIT_SHA .
    - docker push $IMAGE_NAME:$CI_COMMIT_SHA
    - docker tag $IMAGE_NAME:$CI_COMMIT_SHA $IMAGE_NAME:latest
    - docker push $IMAGE_NAME:latest

test:
  stage: test
  image: node:16-alpine
  script:
    - npm ci
    - npm test

deploy_to_cloud_run:
  stage: deploy
  image: google/cloud-sdk:latest
  script:
    - gcloud run deploy gitlab-demo-app
      --image $IMAGE_NAME:$CI_COMMIT_SHA
      --platform managed
      --region $GCP_REGION
      --allow-unauthenticated
      --port 8080
      --memory 512Mi
      --cpu 1
      --max-instances 10
    - echo "Application deployed to Cloud Run"
  only:
    - main
```

## Hands-on Projects

### Project 1: Static Website Deployment

**Objective**: Deploy a static website to Google Cloud Storage with GitLab CI/CD

1. Create `index.html`:
```html
<!DOCTYPE html>
<html>
<head>
    <title>GitLab + GCP Demo</title>
    <style>
        body { font-family: Arial, sans-serif; margin: 40px; }
        .container { max-width: 800px; margin: 0 auto; }
        .header { background: #6366f1; color: white; padding: 20px; border-radius: 8px; }
    </style>
</head>
<body>
    <div class="container">
        <div class="header">
            <h1>GitLab + Google Cloud Platform</h1>
            <p>Successfully deployed via GitLab CI/CD!</p>
        </div>
        <h2>Project Features</h2>
        <ul>
            <li>Automated deployment with GitLab CI/CD</li>
            <li>Static hosting on Google Cloud Storage</li>
            <li>Version control with Git</li>
        </ul>
    </div>
</body>
</html>
```

2. Update `.gitlab-ci.yml` for static site deployment:
```yaml
deploy_static_site:
  stage: deploy
  image: google/cloud-sdk:latest
  script:
    - gsutil -m cp -r *.html gs://gitlab-demo-bucket-${GCP_PROJECT_ID}/
    - gsutil web set -m index.html gs://gitlab-demo-bucket-${GCP_PROJECT_ID}/
    - echo "Static site deployed to https://storage.googleapis.com/gitlab-demo-bucket-${GCP_PROJECT_ID}/index.html"
  only:
    - main
```

### Project 2: Serverless Function with Cloud Functions

**Objective**: Deploy a serverless function to Google Cloud Functions

1. Create `main.py`:
```python
import functions_framework
import json
from datetime import datetime

@functions_framework.http
def hello_gitlab(request):
    """HTTP Cloud Function for GitLab demo"""
    
    if request.method == 'OPTIONS':
        headers = {
            'Access-Control-Allow-Origin': '*',
            'Access-Control-Allow-Methods': 'GET',
            'Access-Control-Allow-Headers': 'Content-Type',
            'Access-Control-Max-Age': '3600'
        }
        return ('', 204, headers)
    
    headers = {
        'Access-Control-Allow-Origin': '*'
    }
    
    data = {
        'message': 'Hello from GitLab CI/CD deployed Cloud Function!',
        'timestamp': datetime.now().isoformat(),
        'source': 'GitLab CI/CD Pipeline'
    }
    
    return (json.dumps(data), 200, headers)
```

2. Create `requirements.txt`:
```
functions-framework==3.4.0
```

3. Add Cloud Functions deployment to `.gitlab-ci.yml`:
```yaml
deploy_cloud_function:
  stage: deploy
  image: google/cloud-sdk:latest
  script:
    - gcloud functions deploy hello-gitlab
      --runtime python39
      --trigger-http
      --allow-unauthenticated
      --region $GCP_REGION
      --source .
      --entry-point hello_gitlab
    - echo "Cloud Function deployed successfully"
  only:
    - main
```

### Project 3: Database Integration with Cloud SQL

**Objective**: Set up a database connection with Cloud SQL

1. Create Cloud SQL instance:
```bash
gcloud sql instances create gitlab-demo-db
  --database-version=MYSQL_8_0
  --tier=db-f1-micro
  --region=$GCP_REGION
```

2. Create database and user:
```bash
gcloud sql databases create appdb --instance=gitlab-demo-db
gcloud sql users create appuser --instance=gitlab-demo-db --password=secure-password
```

3. Update your application to connect to Cloud SQL (example in Node.js):
```javascript
const mysql = require('mysql2/promise');

const dbConfig = {
  host: process.env.DB_HOST,
  user: process.env.DB_USER,
  password: process.env.DB_PASSWORD,
  database: process.env.DB_NAME,
  socketPath: process.env.DB_SOCKET_PATH
};

async function connectToDatabase() {
  try {
    const connection = await mysql.createConnection(dbConfig);
    console.log('Connected to Cloud SQL database');
    return connection;
  } catch (error) {
    console.error('Database connection error:', error);
    throw error;
  }
}
```

## Best Practices

### Security Best Practices

1. **Never commit sensitive data**:
   - Use GitLab CI/CD variables for secrets
   - Mark sensitive variables as "protected" and "masked"
   - Use `.gitignore` to exclude sensitive files

2. **Service Account Security**:
   - Use minimal required permissions
   - Rotate service account keys regularly
   - Monitor service account usage

3. **Network Security**:
   - Use VPC for production deployments
   - Implement proper firewall rules
   - Use Cloud IAM for access control

### CI/CD Best Practices

1. **Pipeline Optimization**:
   - Use caching for dependencies
   - Parallel job execution when possible
   - Fail fast on errors

2. **Environment Management**:
   - Use different environments (dev, staging, prod)
   - Environment-specific configurations
   - Automated testing before deployment

3. **Monitoring and Logging**:
   - Enable Cloud Logging
   - Set up alerts for failures
   - Monitor resource usage

### Example optimized `.gitlab-ci.yml`:
```yaml
# Optimized GitLab CI/CD Pipeline
image: google/cloud-sdk:latest

stages:
  - build
  - test
  - deploy

variables:
  DOCKER_DRIVER: overlay2
  GOOGLE_APPLICATION_CREDENTIALS: /tmp/gcp-key.json

cache:
  paths:
    - node_modules/
    - .npm/

before_script:
  - echo $GCP_SERVICE_ACCOUNT_KEY | base64 -d > ${GOOGLE_APPLICATION_CREDENTIALS}
  - gcloud auth activate-service-account --key-file ${GOOGLE_APPLICATION_CREDENTIALS}
  - gcloud config set project $GCP_PROJECT_ID

build:
  stage: build
  script:
    - npm ci --cache .npm --prefer-offline
    - npm run build
  artifacts:
    paths:
      - dist/
    expire_in: 1 hour

test:
  stage: test
  script:
    - npm ci --cache .npm --prefer-offline
    - npm run test
    - npm run lint
  coverage: '/Lines\s*:\s*(\d+\.\d+)%/'

deploy:
  stage: deploy
  script:
    - gcloud run deploy $CI_PROJECT_NAME
      --image gcr.io/$GCP_PROJECT_ID/$CI_PROJECT_NAME:$CI_COMMIT_SHA
      --platform managed
      --region $GCP_REGION
      --allow-unauthenticated
  only:
    - main
  environment:
    name: production
    url: https://$CI_PROJECT_NAME-$GCP_PROJECT_ID.a.run.app
```

## Troubleshooting

### Common Issues and Solutions

1. **Authentication Errors**:
   ```
   Error: (gcloud.auth.activate-service-account) Invalid key file
   ```
   - Verify service account JSON is correct
   - Check if the key is properly base64 encoded
   - Ensure the service account has required permissions

2. **Build Failures**:
   ```
   Error: Docker build failed
   ```
   - Check Dockerfile syntax
   - Verify base image availability
   - Review dependency installation steps

3. **Deployment Errors**:
   ```
   Error: Cloud Run deployment failed
   ```
   - Check if APIs are enabled
   - Verify region availability
   - Review resource quotas

4. **Permission Denied**:
   ```
   Error: Permission denied on resource
   ```
   - Check IAM roles and permissions
   - Verify service account has required access
   - Review project billing status

### Useful Debugging Commands

```bash
# Check gcloud configuration
gcloud config list

# View service account details
gcloud iam service-accounts list

# Check enabled APIs
gcloud services list --enabled

# View Cloud Run services
gcloud run services list

# Check Cloud Storage buckets
gsutil ls

# View logs
gcloud logging read "resource.type=cloud_run_revision"
```

## Next Steps

1. **Advanced CI/CD**:
   - Implement blue-green deployments
   - Add automated testing with coverage reports
   - Set up multi-environment pipelines

2. **Monitoring and Observability**:
   - Configure Cloud Monitoring
   - Set up alerting policies
   - Implement distributed tracing

3. **Infrastructure as Code**:
   - Use Terraform for GCP resources
   - Implement GitOps practices
   - Automate resource provisioning

4. **Security Enhancements**:
   - Implement container image scanning
   - Add secret management with Secret Manager
   - Configure network security policies

## Resources

- [GitLab CI/CD Documentation](https://docs.gitlab.com/ee/ci/)
- [Google Cloud Platform Documentation](https://cloud.google.com/docs)
- [Cloud Run Documentation](https://cloud.google.com/run/docs)
- [Cloud Build Documentation](https://cloud.google.com/build/docs)
- [GitLab-GCP Integration Guide](https://about.gitlab.com/partners/technology-partners/gcp/)

## Conclusion

This guide provides a comprehensive introduction to GitLab and its integration with Google Cloud Platform. By following these steps and working through the hands-on projects, you'll gain practical experience with:

- GitLab project management and CI/CD pipelines
- Google Cloud Platform services and deployment
- DevOps best practices and automation
- Security and monitoring considerations

Continue practicing with these concepts and explore advanced features as you build more complex applications!