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

**What we're doing**: Setting up your GitLab account to start version controlling your projects and preparing for CI/CD automation. GitLab provides a complete DevOps platform with integrated version control, CI/CD, and project management tools.

1. Visit [GitLab.com](https://gitlab.com)
2. Click "Sign up" and create a free account
3. Verify your email address
4. Complete your profile setup

### Step 2: Understanding GitLab Interface

**What we're doing**: Learning the key components of GitLab that we'll use throughout this tutorial for project management, code collaboration, and automated deployments. Understanding these components will help you navigate GitLab effectively and use its features to their full potential.

**Key Components:**
- **Projects**: Your repositories - where your code lives and is version controlled
- **Groups**: Collections of projects - useful for organizing related projects and managing team permissions
- **Issues**: Bug tracking and feature requests - project management tool for tracking work
- **Merge Requests**: Code review and collaboration - how teams review code changes before merging
- **CI/CD**: Continuous Integration/Deployment - automated testing and deployment pipelines
- **Wiki**: Documentation - project documentation and knowledge sharing platform
- **Snippets**: Code sharing - small code examples and utilities that can be shared across projects

### Step 3: Create Your First Project

**What we're doing**: Creating a GitLab project that will serve as our learning playground and eventually connect to Google Cloud Platform for automated deployments. This project will demonstrate the entire DevOps workflow from code creation to cloud deployment.

1. Click "New project" on GitLab dashboard
2. Choose "Create blank project"
3. Fill in project details:
   - Project name: `my-first-gitlab-project`
   - Description: `Learning GitLab with GCP integration`
   - Visibility: Private (recommended for learning to keep your experiments private)
4. Initialize with README (this creates an initial commit and main branch)
5. Click "Create project"

### Step 4: Basic Git Operations with GitLab

**What we're doing**: Learning the fundamental Git operations that form the foundation of version control. These commands will be essential for managing code changes, collaborating with others, and triggering automated deployments.

**Clone your repository:**
```bash
# Download the repository from GitLab to your local machine
# This creates a local copy of your remote repository
git clone https://gitlab.com/your-username/my-first-gitlab-project.git

# Navigate into the project directory
# This is where all your project files will be stored locally
cd my-first-gitlab-project
```

**Create a simple file:**
```bash
# Create a simple text file with content
# The ">" operator creates a new file and writes content to it
echo "Hello GitLab!" > hello.txt

# Stage the file for commit
# This tells Git to track changes to this file
git add hello.txt

# Create a commit with a descriptive message
# This saves a snapshot of your changes with a message explaining what you did
git commit -m "Add hello.txt file"

# Push the changes to the remote repository on GitLab
# This uploads your local changes to GitLab so others can see them
git push origin main
```

**Create a branch:**
```bash
# Create a new branch and switch to it in one command
# Branches allow you to work on features without affecting the main code
git checkout -b feature/add-documentation

# Create a markdown documentation file
# The "#" creates a heading in markdown format
echo "# My Project Documentation" > docs.md

# Stage the new documentation file
# This prepares the file to be committed
git add docs.md

# Commit the documentation with a clear message
# This saves the changes to the branch history
git commit -m "Add documentation"

# Push the new branch to GitLab
# This creates the branch on GitLab and uploads your changes
git push origin feature/add-documentation
```

### Step 5: Working with Merge Requests

**What we're doing**: Learning how to use merge requests for code review and collaboration. Merge requests are GitLab's way of proposing changes to be merged into the main branch after review. This is a critical part of the collaborative development workflow.

1. Go to your GitLab project
2. Click "Merge Requests" → "New merge request"
3. Select source branch: `feature/add-documentation`
4. Select target branch: `main`
5. Add title and description
6. Click "Create merge request"
7. Review changes and merge

## Setting up Google Cloud Platform

### Step 6: Create GCP Account and Project

**What we're doing**: Setting up Google Cloud Platform to host our applications and services. We'll use the free tier which provides $300 in credits and access to many services without cost. This will be the foundation for our cloud infrastructure.

1. Visit [Google Cloud Console](https://console.cloud.google.com)
2. Sign in with your Google account
3. Accept terms and activate free tier ($300 credit)
4. Create a new project:
   - Project name: `gitlab-integration-demo`
   - Note down the Project ID (this will be unique and used in configurations)

### Step 7: Enable Required APIs

**What we're doing**: Enabling the Google Cloud APIs that our CI/CD pipeline will use. APIs in Google Cloud must be explicitly enabled before they can be used, which helps with security and cost management.

1. Go to "APIs & Services" → "Library"
2. Enable the following APIs:
   - Cloud Build API (for building Docker containers)
   - Cloud Run API (for running containerized applications)
   - Container Registry API (for storing Docker images)
   - Compute Engine API (for virtual machines if needed)
   - Cloud Storage API (for file storage and static website hosting)

### Step 8: Create Service Account

**What we're doing**: Creating a service account that GitLab will use to authenticate with Google Cloud. Service accounts are special accounts used by applications (not humans) to access Google Cloud resources securely.

1. Go to "IAM & Admin" → "Service Accounts"
2. Click "Create Service Account"
3. Fill in details:
   - Name: `gitlab-ci-service-account`
   - Description: `Service account for GitLab CI/CD`
4. Grant roles (these determine what the service account can do):
   - Cloud Build Editor (can create and manage builds)
   - Cloud Run Admin (can deploy and manage Cloud Run services)
   - Storage Admin (can manage Cloud Storage buckets and files)
   - Compute Admin (can manage compute resources)
5. Click "Done"
6. Click on the created service account
7. Go to "Keys" tab → "Add Key" → "Create new key"
8. Choose JSON format and download the key file
9. **Important**: Keep this file secure and never commit it to version control

### Step 9: Set up Cloud Storage Bucket

**What we're doing**: Creating a Cloud Storage bucket that will store our application files and can serve static websites. Buckets are containers for files in Google Cloud Storage.

1. Go to "Cloud Storage" → "Buckets"
2. Click "Create bucket"
3. Name: `gitlab-demo-bucket-[your-project-id]` (must be globally unique)
4. Choose region closest to you (reduces latency and costs)
5. Use default settings for other options
6. Click "Create"

## GitLab CI/CD with GCP

### Step 10: Configure GitLab Variables

**What we're doing**: Setting up secure environment variables in GitLab that our CI/CD pipeline will use to authenticate with Google Cloud. These variables keep sensitive information secure and allow our pipeline to access Google Cloud services.

1. Go to your GitLab project
2. Navigate to "Settings" → "CI/CD"
3. Expand "Variables" section
4. Add the following variables:
   - `GCP_PROJECT_ID`: Your GCP project ID (not sensitive)
   - `GCP_SERVICE_ACCOUNT_KEY`: Content of your service account JSON key (mark as protected and masked for security)
   - `GCP_REGION`: Your preferred GCP region (e.g., `us-central1`)

### Step 11: Create GitLab CI/CD Pipeline

**What we're doing**: Creating our first CI/CD pipeline configuration file. This YAML file tells GitLab how to build, test, and deploy our application automatically whenever we push code changes.

Create `.gitlab-ci.yml` in your project root:

```yaml
# GitLab CI/CD Pipeline Configuration for GCP Integration
# This file defines the automated steps that run when code is pushed

# Define the stages that jobs will run in
# Jobs in the same stage run in parallel, stages run sequentially
stages:
  - build    # Compile and prepare application
  - test     # Run automated tests
  - deploy   # Deploy to cloud infrastructure

# Global variables available to all jobs
variables:
  DOCKER_DRIVER: overlay2                              # Docker storage driver for better performance
  GOOGLE_APPLICATION_CREDENTIALS: /tmp/gcp-key.json    # Path where GCP credentials will be stored

# Commands that run before each job
# This sets up authentication with Google Cloud
before_script:
  # Decode the base64-encoded service account key and save it to a file
  - echo $GCP_SERVICE_ACCOUNT_KEY | base64 -d > ${GOOGLE_APPLICATION_CREDENTIALS}
  # Authenticate with Google Cloud using the service account
  - gcloud auth activate-service-account --key-file ${GOOGLE_APPLICATION_CREDENTIALS}
  # Set the default project for all gcloud commands
  - gcloud config set project $GCP_PROJECT_ID

# Build job - compiles and prepares the application
build:
  stage: build                           # This job runs in the build stage
  image: google/cloud-sdk:latest         # Docker image that includes gcloud CLI
  script:
    - echo "Building application..."      # Placeholder for actual build commands
    - echo "Build completed successfully"
  artifacts:                             # Files to save after job completion
    paths:
      - build/                           # Save build directory for later stages
    expire_in: 1 hour                    # Automatically delete artifacts after 1 hour

# Test job - runs automated tests
test:
  stage: test                           # This job runs in the test stage
  image: google/cloud-sdk:latest        # Use same image for consistency
  script:
    - echo "Running tests..."           # Placeholder for actual test commands
    - echo "All tests passed"

# Deploy job - uploads files to Google Cloud Storage
deploy_to_gcs:
  stage: deploy                         # This job runs in the deploy stage
  image: google/cloud-sdk:latest        # Use gcloud CLI for deployment
  script:
    - echo "Deploying to Google Cloud Storage..."
    # Create a simple file to demonstrate deployment
    - echo "Hello from GitLab CI/CD!" > deploy.txt
    # Upload the file to our Cloud Storage bucket
    - gsutil cp deploy.txt gs://gitlab-demo-bucket-${GCP_PROJECT_ID}/
    - echo "Deployment completed"
  only:
    - main                              # Only run this job when pushing to main branch

## Testing Your First Deployment

**What we're doing**: Verifying that your CI/CD pipeline works correctly and that files are successfully deployed to Google Cloud Storage. This section shows you how to check your deployment and troubleshoot common issues.

### Step 11a: Verify the Pipeline Execution

1. **Check Pipeline Status in GitLab**:
   - Go to your GitLab project
   - Click "CI/CD" → "Pipelines"
   - You should see a pipeline running or completed
   - **Expected Result**: Green checkmarks for all stages (build, test, deploy)

2. **View Pipeline Logs**:
   - Click on the pipeline to see detailed logs
   - Check each job's output for errors
   - **Expected Result**: Deploy job should show "Deployment completed" message

### Step 11b: Verify Files in Google Cloud Storage

1. **Check via Google Cloud Console**:
   - Go to [Google Cloud Console](https://console.cloud.google.com)
   - Navigate to "Cloud Storage" → "Buckets"
   - Click on your bucket `gitlab-demo-bucket-[your-project-id]`
   - **Expected Result**: You should see `deploy.txt` file listed

2. **Check via Command Line**:
   ```bash
   # List files in your bucket
   gsutil ls gs://gitlab-demo-bucket-[your-project-id]/
   
   # Expected output:
   # gs://gitlab-demo-bucket-[your-project-id]/deploy.txt
   
   # View file contents
   gsutil cat gs://gitlab-demo-bucket-[your-project-id]/deploy.txt
   
   # Expected output:
   # Hello from GitLab CI/CD!
   ```

3. **Access via Direct URL**:
   - Open browser and go to: `https://storage.googleapis.com/gitlab-demo-bucket-[your-project-id]/deploy.txt`
   - **Expected Result**: You should see "Hello from GitLab CI/CD!" displayed in the browser
```

### Step 12: Create a Simple Web Application

**What we're doing**: Building a real Node.js web application that we can deploy to Google Cloud. This will demonstrate how to containerize an application and deploy it using our CI/CD pipeline. We'll create a simple Express.js server with health checks and proper logging.

Create a simple Node.js application to deploy:

**package.json:**
```json
{
  "name": "gitlab-gcp-demo",                           // Project name
  "version": "1.0.0",                                  // Semantic version number
  "description": "Demo app for GitLab-GCP integration", // Project description
  "main": "server.js",                                 // Entry point file
  "scripts": {
    "start": "node server.js",                         // Production start command
    "test": "echo \"No tests specified\" && exit 0"    // Test command (placeholder)
  },
  "dependencies": {
    "express": "^4.18.0"                               // Web framework dependency
  }
}
```

**server.js:**
```javascript
// Import the Express.js framework for creating web servers
const express = require('express');

// Create an Express application instance
const app = express();

// Set the port from environment variable or default to 8080
// Google Cloud Run expects applications to listen on PORT environment variable
const port = process.env.PORT || 8080;

// Define the main route that responds to GET requests at the root path
app.get('/', (req, res) => {
  // Send a JSON response with application information
  res.json({
    message: 'Hello from GitLab CI/CD deployed to Google Cloud!',
    timestamp: new Date().toISOString(),                    // Current timestamp in ISO format
    environment: process.env.NODE_ENV || 'development'      // Environment (development/production)
  });
});

// Health check endpoint - important for container orchestration
app.get('/health', (req, res) => {
  // Return application health status and uptime
  res.json({ 
    status: 'healthy',           // Health status indicator
    uptime: process.uptime()     // How long the application has been running (seconds)
  });
});

// Start the server and listen on the specified port
app.listen(port, () => {
  // Log server startup information
  console.log(`Server running on port ${port}`);
});
```

**Dockerfile:**
```dockerfile
# Use the official Node.js 16 Alpine Linux image as base
# Alpine is a security-oriented, lightweight Linux distribution
FROM node:16-alpine

# Set the working directory inside the container
# All subsequent commands will be run from this directory
WORKDIR /app

# Copy package.json and package-lock.json (if available)
# We copy these first to take advantage of Docker layer caching
COPY package*.json ./

# Install only production dependencies
# --ci ensures a clean install based on package-lock.json
# --only=production excludes development dependencies to reduce image size
RUN npm ci --only=production

# Copy the rest of the application source code
# This is done after npm install to optimize Docker layer caching
COPY . .

# Expose port 8080 to allow external connections
# This is the port our Node.js application listens on
EXPOSE 8080

# Switch to the node user for security
# Running as root is a security risk, so we switch to a non-privileged user
USER node

# Define the command to run when the container starts
# This runs our Node.js application
CMD ["npm", "start"]
```

### Step 13: Enhanced CI/CD Pipeline for Cloud Run

**What we're doing**: Upgrading our CI/CD pipeline to build Docker containers and deploy them to Google Cloud Run. Cloud Run is a serverless platform that automatically scales our containerized applications. This pipeline will build our Node.js app into a Docker image and deploy it.

Update your `.gitlab-ci.yml`:

```yaml
# Enhanced GitLab CI/CD Pipeline for Container Deployment
# This pipeline builds Docker images and deploys to Google Cloud Run

stages:
  - build     # Build and push Docker image
  - test      # Run application tests
  - deploy    # Deploy to Cloud Run

# Global variables used across all jobs
variables:
  DOCKER_DRIVER: overlay2                                    # Docker storage driver
  GOOGLE_APPLICATION_CREDENTIALS: /tmp/gcp-key.json          # GCP credentials file path
  IMAGE_NAME: gcr.io/$GCP_PROJECT_ID/gitlab-demo-app        # Full Docker image name with registry

# Setup commands that run before each job
before_script:
  # Decode and save the GCP service account credentials
  - echo $GCP_SERVICE_ACCOUNT_KEY | base64 -d > ${GOOGLE_APPLICATION_CREDENTIALS}
  # Authenticate with Google Cloud using the service account
  - gcloud auth activate-service-account --key-file ${GOOGLE_APPLICATION_CREDENTIALS}
  # Set the default GCP project for all commands
  - gcloud config set project $GCP_PROJECT_ID
  # Configure Docker to authenticate with Google Container Registry
  - gcloud auth configure-docker

# Build job - creates and pushes Docker image
build:
  stage: build                              # Runs in the build stage
  image: google/cloud-sdk:latest            # Image with gcloud and docker
  services:
    - docker:dind                           # Docker-in-Docker service for building images
  script:
    # Build Docker image with commit SHA as tag for unique versioning
    - docker build -t $IMAGE_NAME:$CI_COMMIT_SHA .
    # Push the image with commit SHA tag to Google Container Registry
    - docker push $IMAGE_NAME:$CI_COMMIT_SHA
    # Also tag the image as 'latest' for convenience
    - docker tag $IMAGE_NAME:$CI_COMMIT_SHA $IMAGE_NAME:latest
    # Push the 'latest' tagged image
    - docker push $IMAGE_NAME:latest

# Test job - runs application tests
test:
  stage: test                               # Runs in the test stage
  image: node:16-alpine                     # Use Node.js image for testing
  script:
    # Install dependencies (including dev dependencies for testing)
    - npm ci
    # Run the test suite defined in package.json
    - npm test

# Deploy job - deploys to Google Cloud Run
deploy_to_cloud_run:
  stage: deploy                             # Runs in the deploy stage
  image: google/cloud-sdk:latest            # Use gcloud CLI for deployment
  script:
    # Deploy to Cloud Run with specific configuration
    - gcloud run deploy gitlab-demo-app      # Service name in Cloud Run
      --image $IMAGE_NAME:$CI_COMMIT_SHA     # Use the specific image we built
      --platform managed                     # Use fully managed Cloud Run
      --region $GCP_REGION                   # Deploy to specified region
      --allow-unauthenticated                # Allow public access (no authentication required)
      --port 8080                           # Port the container listens on
      --memory 512Mi                        # Allocate 512MB RAM
      --cpu 1                               # Allocate 1 CPU unit
      --max-instances 10                    # Limit maximum number of instances
    - echo "Application deployed to Cloud Run"
  only:
    - main                                  # Only deploy when pushing to main branch

## Testing Your Cloud Run Deployment

**What we're doing**: Verifying that your containerized application is successfully deployed to Google Cloud Run and is accessible via the internet. We'll test both the main endpoint and the health check endpoint.

### Step 13a: Verify Cloud Run Deployment

1. **Check Deployment Status in GitLab**:
   - Go to your GitLab project → "CI/CD" → "Pipelines"
   - Verify all stages completed successfully
   - **Expected Result**: All three stages (build, test, deploy) should show green checkmarks

2. **View Deployment in Google Cloud Console**:
   - Go to [Google Cloud Console](https://console.cloud.google.com)
   - Navigate to "Cloud Run"
   - **Expected Result**: You should see "gitlab-demo-app" service listed with status "✓ Receiving traffic"

### Step 13b: Test the Deployed Application

1. **Get the Application URL**:
   ```bash
   # Get the service URL using gcloud CLI
   gcloud run services describe gitlab-demo-app \
     --platform=managed \
     --region=[your-region] \
     --format="value(status.url)"
   
   # Expected output (example):
   # https://gitlab-demo-app-abc123-uc.a.run.app
   ```

2. **Test the Main Endpoint**:
   ```bash
   # Test using curl command
   curl https://gitlab-demo-app-[your-hash].a.run.app/
   
   # Expected JSON response:
   {
     "message": "Hello from GitLab CI/CD deployed to Google Cloud!",
     "timestamp": "2025-06-28T10:30:45.123Z", 
     "environment": "production"
   }
   ```

3. **Test the Health Check Endpoint**:
   ```bash
   # Test health endpoint
   curl https://gitlab-demo-app-[your-hash].a.run.app/health
   
   # Expected JSON response:
   {
     "status": "healthy",
     "uptime": 142.5
   }
   ```

4. **Test in Web Browser**:
   - Open your browser and navigate to the Cloud Run URL
   - **Expected Result**: You should see the JSON response displayed in the browser
   - Try adding `/health` to the URL to test the health endpoint

### Step 13c: Monitor and Verify Performance

1. **Check Application Logs**:
   ```bash
   # View recent logs from your application
   gcloud logs read "resource.type=cloud_run_revision AND resource.labels.service_name=gitlab-demo-app" \
     --limit=50 \
     --format="table(timestamp,textPayload)"
   
   # Expected output:
   # TIMESTAMP                  TEXT_PAYLOAD
   # 2025-06-28T10:30:45.123Z   Server running on port 8080
   ```

2. **Verify in Cloud Console Logs**:
   - Go to Google Cloud Console → "Logging" → "Logs Explorer"
   - Filter by "Cloud Run Revision"
   - **Expected Result**: You should see startup logs and any request logs

3. **Check Metrics and Performance**:
   - In Cloud Console, go to "Cloud Run" → click on your service
   - Click on "Metrics" tab
   - **Expected Result**: You should see request counts, latency, and instance metrics
```

## Hands-on Projects

### Project 1: Static Website Deployment

**What we're doing**: Creating and deploying a static website to Google Cloud Storage using GitLab CI/CD. This project demonstrates how to automatically deploy web content whenever you push changes to your repository. Static websites are perfect for documentation, portfolios, and simple web applications.

**Objective**: Deploy a static website to Google Cloud Storage with GitLab CI/CD

1. Create `index.html`:
```html
<!DOCTYPE html>
<html>
<head>
    <!-- Page title that appears in browser tab -->
    <title>GitLab + GCP Demo</title>
    <style>
        /* CSS styling for a clean, modern look */
        body { 
            font-family: Arial, sans-serif; 
            margin: 40px;                    /* Add space around content */
            background-color: #f5f5f5;      /* Light gray background */
        }
        .container { 
            max-width: 800px;               /* Limit content width for readability */
            margin: 0 auto;                 /* Center the container */
            background: white;              /* White background for content */
            padding: 20px;                  /* Inner spacing */
            border-radius: 10px;            /* Rounded corners */
            box-shadow: 0 2px 10px rgba(0,0,0,0.1); /* Subtle shadow */
        }
        .header { 
            background: #6366f1;            /* Purple background */
            color: white; 
            padding: 20px; 
            border-radius: 8px;             /* Rounded corners */
            text-align: center;             /* Center-align text */
        }
        .features {
            margin-top: 30px;               /* Space above features section */
        }
        .feature-list {
            list-style-type: none;          /* Remove default bullet points */
            padding-left: 0;                /* Remove default indentation */
        }
        .feature-list li {
            background: #e7f3ff;            /* Light blue background */
            margin: 10px 0;                 /* Vertical spacing between items */
            padding: 15px;                  /* Inner spacing */
            border-radius: 5px;             /* Rounded corners */
            border-left: 4px solid #6366f1; /* Purple left border accent */
        }
    </style>
</head>
<body>
    <div class="container">
        <!-- Main header section -->
        <div class="header">
            <h1>GitLab + Google Cloud Platform</h1>
            <p>Successfully deployed via GitLab CI/CD!</p>
            <!-- Display current timestamp when page was generated -->
            <small>Deployed on: <span id="deployTime"></span></small>
        </div>
        
        <!-- Features section -->
        <div class="features">
            <h2>Project Features</h2>
            <ul class="feature-list">
                <li>✅ Automated deployment with GitLab CI/CD</li>
                <li>☁️ Static hosting on Google Cloud Storage</li>
                <li>📝 Version control with Git</li>
                <li>🔧 Infrastructure as Code</li>
                <li>🚀 Continuous Integration and Deployment</li>
            </ul>
        </div>
        
        <!-- Technology stack information -->
        <div class="tech-stack">
            <h3>Technology Stack</h3>
            <p><strong>Version Control:</strong> GitLab</p>
            <p><strong>CI/CD Platform:</strong> GitLab CI/CD</p>
            <p><strong>Cloud Provider:</strong> Google Cloud Platform</p>
            <p><strong>Hosting:</strong> Google Cloud Storage</p>
        </div>
    </div>
    
    <script>
        // JavaScript to display current date and time
        document.getElementById('deployTime').textContent = new Date().toLocaleString();
    </script>
</body>
</html>
```

2. Update `.gitlab-ci.yml` for static site deployment:
```yaml
# Add this job to your existing .gitlab-ci.yml file
# This job deploys static files to Google Cloud Storage

deploy_static_site:
  stage: deploy                                    # Runs in the deploy stage
  image: google/cloud-sdk:latest                   # Use gcloud CLI for deployment
  script:
    # Copy all HTML files to the Cloud Storage bucket
    # -m flag enables multi-threading for faster uploads
    # -r flag enables recursive copying of directories
    - gsutil -m cp -r *.html gs://gitlab-demo-bucket-${GCP_PROJECT_ID}/
    
    # Configure the bucket for static website hosting
    # Set index.html as the main page and 404.html as error page
    - gsutil web set -m index.html -e 404.html gs://gitlab-demo-bucket-${GCP_PROJECT_ID}/
    
    # Make the bucket publicly readable so people can access the website
    - gsutil iam ch allUsers:objectViewer gs://gitlab-demo-bucket-${GCP_PROJECT_ID}
    
    # Output the URL where the website can be accessed
    - echo "Static site deployed to https://storage.googleapis.com/gitlab-demo-bucket-${GCP_PROJECT_ID}/index.html"
  only:
    - main                                         # Only deploy from main branch

## Testing Your Static Website Deployment

**What we're doing**: Verifying that your static website is properly deployed to Google Cloud Storage and accessible via the internet. We'll test the website functionality, performance, and ensure all assets load correctly.

### Project 1 Testing Steps:

1. **Verify Pipeline Completion**:
   - Check GitLab CI/CD pipeline status
   - **Expected Result**: `deploy_static_site` job should complete successfully

2. **Test Website Access**:
   ```bash
   # Test the main website URL
   curl -I https://storage.googleapis.com/gitlab-demo-bucket-[your-project-id]/index.html
   
   # Expected response headers:
   # HTTP/2 200 
   # content-type: text/html
   # cache-control: public, max-age=3600
   ```

3. **Visual Verification**:
   - Open browser and navigate to: `https://storage.googleapis.com/gitlab-demo-bucket-[your-project-id]/index.html`
   - **Expected Result**: 
     - Purple header with "GitLab + Google Cloud Platform" title
     - "Successfully deployed via GitLab CI/CD!" message
     - Feature list with checkmarks and emojis
     - Technology stack information
     - Current timestamp displayed at bottom

4. **Check Website Features**:
   - Verify responsive design works on different screen sizes
   - Check that JavaScript timestamp updates correctly
   - Ensure all CSS styling loads properly
   - **Expected Result**: Professional-looking website with modern styling

5. **Verify File Permissions**:
   ```bash
   # Check bucket permissions
   gsutil iam get gs://gitlab-demo-bucket-[your-project-id]
   
   # Expected: Should include allUsers with objectViewer role
   # This allows public access to view the website
   ```
```

### Project 2: Serverless Function with Cloud Functions

**What we're doing**: Creating and deploying a serverless Python function to Google Cloud Functions. Serverless functions are perfect for handling specific tasks like API endpoints, data processing, or responding to events. They automatically scale and you only pay for what you use.

**Objective**: Deploy a serverless function to Google Cloud Functions

1. Create `main.py`:
```python
# Import required libraries for Cloud Functions
import functions_framework      # Google Cloud Functions framework
import json                    # For JSON data handling
from datetime import datetime  # For timestamp generation

# Decorator that marks this function as an HTTP-triggered Cloud Function
@functions_framework.http
def hello_gitlab(request):
    """
    HTTP Cloud Function for GitLab demo
    
    This function responds to HTTP requests and returns information
    about the deployment and current time.
    
    Args:
        request: Flask Request object containing HTTP request data
        
    Returns:
        Tuple containing (response_body, status_code, headers)
    """
    
    # Handle preflight CORS requests (OPTIONS method)
    # This is required for web browsers to access the function from different domains
    if request.method == 'OPTIONS':
        headers = {
            'Access-Control-Allow-Origin': '*',          # Allow requests from any domain
            'Access-Control-Allow-Methods': 'GET,POST',   # Allowed HTTP methods
            'Access-Control-Allow-Headers': 'Content-Type', # Allowed headers
            'Access-Control-Max-Age': '3600'             # Cache preflight response for 1 hour
        }
        return ('', 204, headers)  # Return empty response with CORS headers
    
    # Set CORS headers for actual requests
    headers = {
        'Access-Control-Allow-Origin': '*',              # Allow cross-origin requests
        'Content-Type': 'application/json'               # Response content type
    }
    
    # Get request method and any query parameters
    method = request.method
    args = request.args.to_dict() if request.args else {}
    
    # Create response data with function information
    data = {
        'message': 'Hello from GitLab CI/CD deployed Cloud Function!',
        'timestamp': datetime.now().isoformat(),         # Current timestamp
        'source': 'GitLab CI/CD Pipeline',               # Deployment source
        'method': method,                                # HTTP method used
        'query_parameters': args,                        # Any query parameters
        'function_name': 'hello_gitlab',                 # Function identifier
        'version': '1.0.0'                              # Function version
    }
    
    # Return JSON response with success status code and CORS headers
    return (json.dumps(data, indent=2), 200, headers)
```

2. Create `requirements.txt`:
```txt
# Python dependencies for the Cloud Function
# This file tells Google Cloud Functions what packages to install

# Google Cloud Functions framework - required for HTTP functions
functions-framework==3.4.0

# Optional: Add other dependencies as needed
# flask==2.3.0          # Web framework (included with functions-framework)
# requests==2.31.0      # HTTP library for making API calls
# google-cloud-storage==2.10.0  # Google Cloud Storage client library
```

3. Add Cloud Functions deployment to `.gitlab-ci.yml`:
```yaml
# Add this job to your existing .gitlab-ci.yml file
# This job deploys a Python function to Google Cloud Functions

deploy_cloud_function:
  stage: deploy                                    # Runs in the deploy stage
  image: google/cloud-sdk:latest                   # Use gcloud CLI
  script:
    # Deploy the function to Google Cloud Functions
    - gcloud functions deploy hello-gitlab         # Function name in Cloud Functions
      --runtime python39                           # Python 3.9 runtime environment
      --trigger-http                               # HTTP trigger (responds to web requests)
      --allow-unauthenticated                      # Allow public access without authentication
      --region $GCP_REGION                         # Deploy to specified region
      --source .                                   # Use current directory as source
      --entry-point hello_gitlab                   # Python function name to execute
      --memory 256MB                               # Allocate 256MB memory
      --timeout 60s                                # Function timeout after 60 seconds
    
    # Get and display the function URL
    - FUNCTION_URL=$(gcloud functions describe hello-gitlab --region=$GCP_REGION --format="value(httpsTrigger.url)")
    - echo "Cloud Function deployed successfully at: $FUNCTION_URL"
  only:
    - main                                         # Only deploy from main branch

## Testing Your Cloud Function Deployment

**What we're doing**: Verifying that your serverless Python function is properly deployed to Google Cloud Functions and responds correctly to HTTP requests. We'll test different request methods and verify CORS functionality.

### Project 2 Testing Steps:

1. **Verify Deployment Status**:
   ```bash
   # Check function status
   gcloud functions describe hello-gitlab --region=[your-region]
   
   # Expected output should include:
   # status: ACTIVE
   # httpsTrigger:
   #   url: https://[region]-[project-id].cloudfunctions.net/hello-gitlab
   ```

2. **Test GET Request**:
   ```bash
   # Test the function with a simple GET request
   curl "https://[region]-[project-id].cloudfunctions.net/hello-gitlab"
   
   # Expected JSON response:
   {
     "message": "Hello from GitLab CI/CD deployed Cloud Function!",
     "timestamp": "2025-06-28T10:30:45.123456",
     "source": "GitLab CI/CD Pipeline",
     "method": "GET",
     "query_parameters": {},
     "function_name": "hello_gitlab",
     "version": "1.0.0"
   }
   ```

3. **Test with Query Parameters**:
   ```bash
   # Test with query parameters
   curl "https://[region]-[project-id].cloudfunctions.net/hello-gitlab?name=GitLab&env=production"
   
   # Expected response should include:
   # "query_parameters": {
   #   "name": "GitLab",
   #   "env": "production"
   # }
   ```

4. **Test CORS Functionality**:
   ```bash
   # Test CORS preflight request
   curl -X OPTIONS \
     -H "Origin: https://example.com" \
     -H "Access-Control-Request-Method: GET" \
     -H "Access-Control-Request-Headers: Content-Type" \
     "https://[region]-[project-id].cloudfunctions.net/hello-gitlab"
   
   # Expected response headers:
   # Access-Control-Allow-Origin: *
   # Access-Control-Allow-Methods: GET,POST
   # Access-Control-Max-Age: 3600
   ```

5. **Test POST Request**:
   ```bash
   # Test POST request with data
   curl -X POST \
     -H "Content-Type: application/json" \
     -d '{"test": "data"}' \
     "https://[region]-[project-id].cloudfunctions.net/hello-gitlab"
   
   # Expected response should show:
   # "method": "POST"
   ```

6. **Monitor Function Logs**:
   ```bash
   # View function execution logs
   gcloud functions logs read hello-gitlab --region=[your-region] --limit=10
   
   # Expected log entries showing function executions and any errors
   ```

7. **Verify in Web Browser**:
   - Open browser and navigate to the function URL
   - **Expected Result**: JSON response displayed in browser with all expected fields
   - Response should be properly formatted and include current timestamp
```

### Project 3: Database Integration with Cloud SQL

**What we're doing**: Setting up a MySQL database in Google Cloud SQL and connecting it to our application. This demonstrates how to work with persistent data storage in the cloud, including database creation, user management, and secure connections from applications.

**Objective**: Set up a database connection with Cloud SQL

1. Create Cloud SQL instance:
```bash
# Create a MySQL database instance in Google Cloud SQL
# This command sets up a managed MySQL database server

gcloud sql instances create gitlab-demo-db \
  --database-version=MYSQL_8_0 \              # Use MySQL version 8.0
  --tier=db-f1-micro \                        # Smallest instance size (free tier eligible)
  --region=$GCP_REGION \                      # Deploy in specified region
  --storage-size=10GB \                       # Allocate 10GB storage
  --storage-type=SSD \                        # Use SSD storage for better performance
  --backup-start-time=03:00 \                 # Daily backup at 3 AM UTC
  --maintenance-window-day=SUN \              # Maintenance on Sundays
  --maintenance-window-hour=04 \              # Maintenance at 4 AM UTC
  --enable-autobackup \                       # Enable automatic backups
  --retain-backups-days=7                     # Keep backups for 7 days
```

2. Create database and user:
```bash
# Create a database within the Cloud SQL instance
gcloud sql databases create appdb \
  --instance=gitlab-demo-db \                 # Instance name we created above
  --charset=utf8mb4 \                        # Character set for international text
  --collation=utf8mb4_unicode_ci             # Collation for proper sorting

# Create a database user with password
gcloud sql users create appuser \
  --instance=gitlab-demo-db \                 # Instance name
  --password=secure-password-change-this \   # User password (change this!)
  --host=%                                   # Allow connections from any host (% wildcard)

# Grant privileges to the user (run this in MySQL after connecting)
# GRANT ALL PRIVILEGES ON appdb.* TO 'appuser'@'%';
# FLUSH PRIVILEGES;
```

3. Update your application to connect to Cloud SQL (example in Node.js):
```javascript
// Database connection module for Cloud SQL
// This module handles connecting to Google Cloud SQL from a Node.js application

// Import MySQL client library with Promise support
const mysql = require('mysql2/promise');

// Database configuration object
// These values should come from environment variables in production
const dbConfig = {
  host: process.env.DB_HOST,                    // Cloud SQL instance IP address
  user: process.env.DB_USER,                    // Database username (appuser)
  password: process.env.DB_PASSWORD,            // Database password
  database: process.env.DB_NAME,                // Database name (appdb)
  
  // For Cloud SQL connections, you can use either:
  // 1. Unix socket path (recommended for Cloud Run/App Engine)
  socketPath: process.env.DB_SOCKET_PATH,       // e.g., '/cloudsql/project:region:instance'
  
  // 2. Or TCP connection with SSL
  ssl: {
    rejectUnauthorized: false                   // Required for Cloud SQL connections
  },
  
  // Connection pool settings for better performance
  connectionLimit: 10,                          // Maximum number of connections
  acquireTimeout: 60000,                        // Maximum time to get connection (60s)
  timeout: 60000,                               // Query timeout (60s)
  reconnect: true                               // Automatically reconnect on connection loss
};

/**
 * Create a connection to the Cloud SQL database
 * @returns {Promise<Connection>} Database connection object
 */
async function connectToDatabase() {
  try {
    // Attempt to create database connection
    console.log('Connecting to Cloud SQL database...');
    const connection = await mysql.createConnection(dbConfig);
    
    // Test the connection with a simple query
    await connection.execute('SELECT 1 as test');
    console.log('Successfully connected to Cloud SQL database');
    
    return connection;
  } catch (error) {
    // Log detailed error information for debugging
    console.error('Database connection error:', {
      message: error.message,
      code: error.code,
      errno: error.errno,
      sqlState: error.sqlState
    });
    
    // Re-throw the error so calling code can handle it
    throw error;
  }
}

/**
 * Create a connection pool for better performance with multiple requests
 * @returns {Pool} Database connection pool
 */
function createConnectionPool() {
  console.log('Creating database connection pool...');
  const pool = mysql.createPool(dbConfig);
  
  // Handle pool errors
  pool.on('error', (error) => {
    console.error('Database pool error:', error);
  });
  
  return pool;
}

/**
 * Example function to create a users table
 * @param {Connection} connection - Database connection
 */
async function createUsersTable(connection) {
  const createTableSQL = `
    CREATE TABLE IF NOT EXISTS users (
      id INT AUTO_INCREMENT PRIMARY KEY,        -- Unique user ID
      username VARCHAR(50) NOT NULL UNIQUE,     -- Username (must be unique)
      email VARCHAR(100) NOT NULL UNIQUE,       -- Email address (must be unique)
      created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP, -- Account creation time
      updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP -- Last update time
    ) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
  `;
  
  try {
    await connection.execute(createTableSQL);
    console.log('Users table created successfully');
  } catch (error) {
    console.error('Error creating users table:', error);
    throw error;
  }
}

/**
 * Example function to insert a new user
 * @param {Connection} connection - Database connection
 * @param {string} username - User's username
 * @param {string} email - User's email address
 * @returns {Object} Insert result
 */
async function insertUser(connection, username, email) {
  const insertSQL = 'INSERT INTO users (username, email) VALUES (?, ?)';
  
  try {
    const [result] = await connection.execute(insertSQL, [username, email]);
    console.log(`User inserted with ID: ${result.insertId}`);
    return result;
  } catch (error) {
    console.error('Error inserting user:', error);
    throw error;
  }
}

/**
 * Example function to get all users
 * @param {Connection} connection - Database connection
 * @returns {Array} Array of user objects
 */
async function getAllUsers(connection) {
  const selectSQL = 'SELECT id, username, email, created_at FROM users ORDER BY created_at DESC';
  
  try {
    const [rows] = await connection.execute(selectSQL);
    console.log(`Retrieved ${rows.length} users`);
    return rows;
  } catch (error) {
    console.error('Error retrieving users:', error);
    throw error;
  }
}

// Export functions for use in other modules
module.exports = {
  connectToDatabase,
  createConnectionPool,
  createUsersTable,
  insertUser,
  getAllUsers
};
```

## Testing Your Database Integration

**What we're doing**: Verifying that your Cloud SQL database is properly configured and your application can successfully connect and perform database operations. We'll test the connection, create tables, and perform basic CRUD operations.

### Project 3 Testing Steps:

1. **Verify Cloud SQL Instance Creation**:
   ```bash
   # Check instance status
   gcloud sql instances describe gitlab-demo-db
   
   # Expected output should include:
   # state: RUNNABLE
   # databaseVersion: MYSQL_8_0
   # settings:
   #   tier: db-f1-micro
   ```

2. **Test Database Connection**:
   ```bash
   # Connect to the database using gcloud
   gcloud sql connect gitlab-demo-db --user=root
   
   # Once connected to MySQL, test basic commands:
   SHOW DATABASES;
   # Expected: Should show 'appdb' in the list
   
   USE appdb;
   SHOW TABLES;
   # Expected: Should show empty set initially, then tables after application runs
   ```

3. **Test Application Database Integration**:
   ```javascript
   // Create a test file: test-database.js
   const { connectToDatabase, createUsersTable, insertUser, getAllUsers } = require('./database-module');
   
   async function testDatabase() {
     try {
       // Test 1: Connection
       console.log('Testing database connection...');
       const connection = await connectToDatabase();
       console.log('✅ Database connection successful');
       
       // Test 2: Table creation
       console.log('Creating users table...');
       await createUsersTable(connection);
       console.log('✅ Users table created');
       
       // Test 3: Insert user
       console.log('Inserting test user...');
       await insertUser(connection, 'testuser', 'test@example.com');
       console.log('✅ User inserted successfully');
       
       // Test 4: Retrieve users
       console.log('Retrieving all users...');
       const users = await getAllUsers(connection);
       console.log('✅ Users retrieved:', users);
       
       // Close connection
       await connection.end();
       console.log('✅ All database tests passed!');
       
     } catch (error) {
       console.error('❌ Database test failed:', error);
     }
   }
   
   testDatabase();
   ```

4. **Run Database Tests**:
   ```bash
   # Run the test file
   node test-database.js
   
   # Expected output:
   # Testing database connection...
   # ✅ Database connection successful
   # Creating users table...
   # ✅ Users table created
   # Inserting test user...
   # ✅ User inserted successfully
   # Retrieving all users...
   # ✅ Users retrieved: [{ id: 1, username: 'testuser', email: 'test@example.com', ... }]
   # ✅ All database tests passed!
   ```

5. **Verify Data in Cloud Console**:
   - Go to Google Cloud Console → "SQL" → "gitlab-demo-db"
   - Click "Connect using Cloud Shell"
   - Run: `mysql -u appuser -p -h [INSTANCE-IP] appdb`
   - Execute: `SELECT * FROM users;`
   - **Expected Result**: Should show the test user data

6. **Test Connection Pooling**:
   ```javascript
   // Test connection pool
   const { createConnectionPool } = require('./database-module');
   
   async function testConnectionPool() {
     const pool = createConnectionPool();
     
     // Test multiple concurrent connections
     const promises = [];
     for (let i = 0; i < 5; i++) {
       promises.push(pool.execute('SELECT ? as test_number', [i]));
     }
     
     const results = await Promise.all(promises);
     console.log('Pool test results:', results);
     
     await pool.end();
   }
   ```

7. **Monitor Database Performance**:
   ```bash
   # Check database metrics
   gcloud sql instances describe gitlab-demo-db --format="table(
     settings.dataDiskSizeGb,
     settings.dataDiskType,
     state,
     databaseVersion
   )"
   
   # View database logs
   gcloud logging read "resource.type=gce_instance AND logName=projects/[PROJECT-ID]/logs/mysql.error" --limit=10
   ```

8. **Test Error Handling**:
   ```javascript
   // Test connection failure handling
   async function testErrorHandling() {
     try {
       // Try connecting with wrong credentials
       const badConfig = { ...dbConfig, password: 'wrong-password' };
       const connection = await mysql.createConnection(badConfig);
     } catch (error) {
       console.log('✅ Error handling works:', error.code);
       // Expected: ER_ACCESS_DENIED_ERROR
     }
   }
   ```
```

## Best Practices

### Security Best Practices

**What we're doing**: Implementing security measures to protect your code, credentials, and cloud resources. Security should be built into every aspect of your CI/CD pipeline and cloud infrastructure.

1. **Never commit sensitive data**:
   - Use GitLab CI/CD variables for secrets (API keys, passwords, certificates)
   - Mark sensitive variables as "protected" and "masked" to prevent exposure in logs
   - Use `.gitignore` to exclude sensitive files from version control
   - Regularly audit your repository for accidentally committed secrets

2. **Service Account Security**:
   - Use minimal required permissions (principle of least privilege)
   - Rotate service account keys regularly (every 90 days recommended)
   - Monitor service account usage through Cloud Logging
   - Use separate service accounts for different environments (dev, staging, prod)

3. **Network Security**:
   - Use VPC for production deployments to isolate resources
   - Implement proper firewall rules to restrict access
   - Use Cloud IAM for granular access control
   - Enable audit logging for all API calls

### CI/CD Best Practices

**What we're doing**: Optimizing our CI/CD pipeline for performance, reliability, and maintainability. These practices ensure fast, reliable deployments and make debugging easier when issues occur.

1. **Pipeline Optimization**:
   - Use caching for dependencies to speed up builds
   - Run jobs in parallel when possible to reduce total time
   - Fail fast on errors to save time and resources
   - Use specific Docker image tags instead of 'latest' for reproducibility

2. **Environment Management**:
   - Use different environments (dev, staging, prod) with separate configurations
   - Store environment-specific configurations in GitLab variables
   - Implement automated testing before deployment to catch issues early
   - Use feature flags to safely deploy new features

3. **Monitoring and Logging**:
   - Enable Cloud Logging for all services to troubleshoot issues
   - Set up alerts for deployment failures and application errors
   - Monitor resource usage to optimize costs and performance
   - Use structured logging with JSON format for better searchability

### Example optimized `.gitlab-ci.yml`:

**What we're doing**: Creating a production-ready CI/CD pipeline with advanced features like dependency caching, parallel testing, environment management, and comprehensive monitoring. This pipeline is optimized for speed and reliability.

```yaml
# Production-Ready GitLab CI/CD Pipeline
# This configuration includes optimization techniques and best practices

# Use a specific version of the Google Cloud SDK for reproducibility
image: google/cloud-sdk:415.0.0

# Define pipeline stages that run sequentially
# Jobs within the same stage can run in parallel
stages:
  - build      # Compile code and create artifacts
  - test       # Run automated tests
  - security   # Security scanning and compliance checks
  - deploy     # Deploy to target environment

# Global variables available to all jobs
variables:
  DOCKER_DRIVER: overlay2                           # Docker storage driver for performance
  GOOGLE_APPLICATION_CREDENTIALS: /tmp/gcp-key.json # GCP credentials file location
  NODE_VERSION: "16"                                # Node.js version for consistency

# Cache configuration to speed up builds
# Dependencies are cached between pipeline runs
cache:
  key: ${CI_COMMIT_REF_SLUG}                       # Cache key based on branch name
  paths:
    - node_modules/                                 # Node.js dependencies
    - .npm/                                         # npm cache directory
    - .cache/                                       # General cache directory
  policy: pull-push                                 # Pull cache at start, push at end

# Commands that run before each job
# Set up authentication and basic configuration
before_script:
  # Decode and save GCP service account credentials
  - echo $GCP_SERVICE_ACCOUNT_KEY | base64 -d > ${GOOGLE_APPLICATION_CREDENTIALS}
  # Authenticate with Google Cloud
  - gcloud auth activate-service-account --key-file ${GOOGLE_APPLICATION_CREDENTIALS}
  # Set default project for all gcloud commands
  - gcloud config set project $GCP_PROJECT_ID
  # Configure Docker authentication for Google Container Registry
  - gcloud auth configure-docker

# Build job - compiles application and creates Docker image
build:
  stage: build                                      # Runs in build stage
  image: node:${NODE_VERSION}-alpine                # Use specific Node.js version
  script:
    # Install dependencies using npm ci for faster, reliable installs
    - npm ci --cache .npm --prefer-offline          # Use cache and prefer offline packages
    
    # Run build process (compile TypeScript, bundle assets, etc.)
    - npm run build                                 # Execute build script from package.json
    
    # Create build info file with deployment metadata
    - echo "{\"version\":\"${CI_COMMIT_SHA}\",\"build_time\":\"$(date -u +%Y-%m-%dT%H:%M:%SZ)\",\"branch\":\"${CI_COMMIT_REF_NAME}\"}" > dist/build-info.json
    
    # Build Docker image with specific tag
    - docker build -t gcr.io/$GCP_PROJECT_ID/$CI_PROJECT_NAME:$CI_COMMIT_SHA .
    
    # Push image to Google Container Registry
    - docker push gcr.io/$GCP_PROJECT_ID/$CI_PROJECT_NAME:$CI_COMMIT_SHA
  
  # Save build artifacts for later stages
  artifacts:
    name: "build-${CI_COMMIT_SHORT_SHA}"           # Artifact name with commit hash
    paths:
      - dist/                                       # Built application files
      - docker-compose.yml                          # Docker configuration
    expire_in: 1 week                              # Keep artifacts for 1 week
    reports:
      dotenv: build.env                            # Environment variables for next stages

# Test jobs - run in parallel for speed
test:unit:
  stage: test                                       # Runs in test stage
  image: node:${NODE_VERSION}-alpine
  script:
    # Install dependencies
    - npm ci --cache .npm --prefer-offline
    
    # Run unit tests with coverage reporting
    - npm run test:unit -- --coverage --reporter=json --outputFile=test-results.json
    
    # Generate test coverage badge
    - npm run coverage:badge
  
  # Save test results and coverage reports
  artifacts:
    when: always                                   # Save artifacts even if tests fail
    paths:
      - coverage/                                  # Coverage reports
      - test-results.json                          # Test results
    reports:
      junit: test-results.xml                      # JUnit format for GitLab test reporting
      coverage_report:
        coverage_format: cobertura                 # Coverage format for GitLab
        path: coverage/cobertura-coverage.xml
    expire_in: 1 week
  
  # Extract coverage percentage for GitLab to display
  coverage: '/Lines\s*:\s*(\d+\.\d+)%/'

test:integration:
  stage: test                                      # Runs in parallel with unit tests
  image: google/cloud-sdk:latest
  services:
    - docker:dind                                  # Docker-in-Docker for testing containers
  script:
    # Start test database using Docker
    - docker run -d --name test-db -e MYSQL_ROOT_PASSWORD=test mysql:8.0
    
    # Wait for database to be ready
    - sleep 30
    
    # Run integration tests against the test database
    - npm run test:integration
  
  # Clean up test containers
  after_script:
    - docker stop test-db || true
    - docker rm test-db || true

# Security scanning job
security:scan:
  stage: security                                  # Runs after tests pass
  image: docker:stable
  services:
    - docker:dind
  script:
    # Scan Docker image for security vulnerabilities
    - docker run --rm -v /var/run/docker.sock:/var/run/docker.sock 
      -v $(pwd):/tmp/.cache/ aquasec/trivy 
      image --exit-code 0 --no-progress --format table 
      gcr.io/$GCP_PROJECT_ID/$CI_PROJECT_NAME:$CI_COMMIT_SHA
  
  # Allow job to fail without stopping pipeline (for now)
  allow_failure: true

# Deployment job for staging environment
deploy:staging:
  stage: deploy                                    # Runs after all tests pass
  image: google/cloud-sdk:latest
  environment:
    name: staging                                  # GitLab environment name
    url: https://$CI_PROJECT_NAME-staging-$GCP_PROJECT_ID.a.run.app
  script:
    # Deploy to Cloud Run with staging configuration
    - gcloud run deploy $CI_PROJECT_NAME-staging
      --image gcr.io/$GCP_PROJECT_ID/$CI_PROJECT_NAME:$CI_COMMIT_SHA
      --platform managed
      --region $GCP_REGION
      --allow-unauthenticated
      --port 8080
      --memory 512Mi
      --cpu 1
      --max-instances 5                            # Lower limit for staging
      --set-env-vars="NODE_ENV=staging,LOG_LEVEL=debug"
    
    # Run smoke tests against deployed staging environment
    - STAGING_URL=$(gcloud run services describe $CI_PROJECT_NAME-staging --platform=managed --region=$GCP_REGION --format="value(status.url)")
    - curl -f $STAGING_URL/health || exit 1       # Verify health endpoint works
    
    # Output deployment information
    - echo "Staging deployment successful at $STAGING_URL"
  
  only:
    - develop                                      # Only deploy develop branch to staging

# Production deployment job (manual approval required)
deploy:production:
  stage: deploy
  image: google/cloud-sdk:latest
  environment:
    name: production                               # Production environment
    url: https://$CI_PROJECT_NAME-$GCP_PROJECT_ID.a.run.app
  script:
    # Deploy to production with production configuration
    - gcloud run deploy $CI_PROJECT_NAME
      --image gcr.io/$GCP_PROJECT_ID/$CI_PROJECT_NAME:$CI_COMMIT_SHA
      --platform managed
      --region $GCP_REGION
      --allow-unauthenticated
      --port 8080
      --memory 1Gi                                 # More memory for production
      --cpu 2                                      # More CPU for production
      --max-instances 100                          # Higher scaling limit
      --set-env-vars="NODE_ENV=production,LOG_LEVEL=info"
    
    # Configure traffic splitting for blue-green deployment (optional)
    - gcloud run services update-traffic $CI_PROJECT_NAME --to-revisions=LATEST=100
    
    # Run production health checks
    - PROD_URL=$(gcloud run services describe $CI_PROJECT_NAME --platform=managed --region=$GCP_REGION --format="value(status.url)")
    - curl -f $PROD_URL/health || exit 1
    - curl -f $PROD_URL/ || exit 1
    
    # Send deployment notification (optional)
    - echo "Production deployment successful at $PROD_URL"
  
  # Require manual approval for production deployments
  when: manual                                     # Manual trigger required
  only:
    - main                                         # Only deploy main branch to production
```

## Testing Your Production-Ready Pipeline

**What we're doing**: Verifying that your advanced CI/CD pipeline with multiple environments, security scanning, and comprehensive monitoring works correctly. We'll test each stage and verify the deployment across different environments.

### Advanced Pipeline Testing Steps:

1. **Test Multi-Stage Pipeline Execution**:
   ```bash
   # Trigger pipeline by pushing to develop branch (staging)
   git checkout develop
   git push origin develop
   
   # Then push to main branch (production)
   git checkout main  
   git merge develop
   git push origin main
   
   # Expected Result: Two separate deployments to staging and production
   ```

2. **Verify Build Stage Testing**:
   - Check GitLab CI/CD → Pipelines
   - Verify build job creates Docker image successfully
   - **Expected Result**: Build artifacts saved and Docker image pushed to Container Registry

3. **Test Security Scanning Results**:
   ```bash
   # Check security scan results in pipeline logs
   # Look for vulnerability reports
   
   # Expected output should include:
   # - Number of vulnerabilities found
   # - Severity levels (LOW, MEDIUM, HIGH, CRITICAL)
   # - Recommendations for fixes
   ```

4. **Verify Staging Environment**:
   ```bash
   # Test staging deployment
   curl https://gitlab-demo-app-staging-[project-id].a.run.app/
   
   # Expected JSON response with:
   # "environment": "staging"
   
   # Test staging health endpoint
   curl https://gitlab-demo-app-staging-[project-id].a.run.app/health
   
   # Expected: Health check should pass
   ```

5. **Test Production Deployment (Manual Trigger)**:
   - Go to GitLab → CI/CD → Pipelines
   - Find the main branch pipeline
   - Click the "Play" button on the `deploy:production` job
   - **Expected Result**: Manual approval required, then deployment proceeds

6. **Verify Production Environment**:
   ```bash
   # Test production deployment
   curl https://gitlab-demo-app-[project-id].a.run.app/
   
   # Expected JSON response with:
   # "environment": "production"
   
   # Performance test
   curl -w "@curl-format.txt" -o /dev/null -s https://gitlab-demo-app-[project-id].a.run.app/
   
   # Expected: Response time should be under 500ms
   ```

7. **Test Caching Effectiveness**:
   - Compare first pipeline run time vs subsequent runs
   - Check pipeline logs for "Using cached dependencies"
   - **Expected Result**: Second run should be significantly faster

8. **Verify Environment-Specific Configurations**:
   ```bash
   # Check staging environment variables
   gcloud run services describe gitlab-demo-app-staging --region=[region] --format="yaml(spec.template.spec.template.spec.containers[0].env)"
   
   # Expected: NODE_ENV=staging, LOG_LEVEL=debug
   
   # Check production environment variables  
   gcloud run services describe gitlab-demo-app --region=[region] --format="yaml(spec.template.spec.template.spec.containers[0].env)"
   
   # Expected: NODE_ENV=production, LOG_LEVEL=info
   ```

9. **Test Rollback Capability**:
   ```bash
   # List all revisions
   gcloud run revisions list --service=gitlab-demo-app --region=[region]
   
   # Rollback to previous revision if needed
   gcloud run services update-traffic gitlab-demo-app --to-revisions=[PREVIOUS-REVISION]=100 --region=[region]
   
   # Expected: Traffic switched to previous version
   ```

10. **Monitor Pipeline Metrics**:
    - Check GitLab Analytics → CI/CD Analytics
    - View pipeline success rate, duration trends
    - **Expected Result**: High success rate, consistent build times

11. **Test Alert Notifications** (if configured):
    ```bash
    # Simulate failure by breaking the application
    # Push code with syntax error or failing test
    
    # Expected Result: 
    # - Pipeline fails at appropriate stage
    # - Notifications sent to configured channels
    # - Deployment doesn't proceed to production
    ```

### Performance Benchmarking:

```bash
# Create curl-format.txt for detailed timing
cat > curl-format.txt << 'EOF'
     time_namelookup:  %{time_namelookup}\n
        time_connect:  %{time_connect}\n
     time_appconnect:  %{time_appconnect}\n
    time_pretransfer:  %{time_pretransfer}\n
       time_redirect:  %{time_redirect}\n
  time_starttransfer:  %{time_starttransfer}\n
                     ----------\n
          time_total:  %{time_total}\n
EOF

# Run performance test
for i in {1..10}; do
  curl -w "@curl-format.txt" -o /dev/null -s https://gitlab-demo-app-[project-id].a.run.app/
done

# Expected Results:
# - DNS lookup: < 50ms
# - Connection: < 100ms  
# - First byte: < 300ms
# - Total time: < 500ms
```
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

## Comprehensive Testing and Verification Checklist

**What we're doing**: Providing a complete checklist to verify that all components of your GitLab-GCP integration are working correctly. Use this checklist after completing the tutorial to ensure everything is functioning as expected.

### ✅ GitLab Setup Verification

- [ ] GitLab account created and verified
- [ ] Project created with proper visibility settings
- [ ] Repository cloned locally and basic Git operations work
- [ ] Merge request workflow tested successfully

### ✅ Google Cloud Platform Setup Verification

- [ ] GCP account activated with free tier credits
- [ ] Project created with billing enabled
- [ ] Required APIs enabled (Cloud Build, Cloud Run, Container Registry, Cloud Storage)
- [ ] Service account created with appropriate roles
- [ ] Service account key downloaded and stored securely

### ✅ CI/CD Pipeline Verification

```bash
# Test 1: Pipeline triggers correctly
git add .
git commit -m "Test pipeline trigger"  
git push origin main

# Expected: Pipeline should start automatically in GitLab

# Test 2: All stages complete successfully
# Check GitLab → CI/CD → Pipelines
# Expected: Green checkmarks for build, test, deploy stages

# Test 3: Environment variables are properly set
# Check pipeline logs for successful authentication
# Expected: No authentication errors in logs
```

### ✅ Deployment Verification

```bash
# Test 1: Cloud Storage deployment
curl -I https://storage.googleapis.com/gitlab-demo-bucket-[project-id]/index.html
# Expected: HTTP 200 response

# Test 2: Cloud Run deployment  
curl https://gitlab-demo-app-[hash].a.run.app/health
# Expected: {"status": "healthy", "uptime": [number]}

# Test 3: Cloud Function deployment
curl https://[region]-[project-id].cloudfunctions.net/hello-gitlab
# Expected: JSON response with timestamp and function info

# Test 4: Database connectivity (if implemented)
node test-database.js
# Expected: All database tests should pass
```

### ✅ Security and Performance Verification

```bash
# Test 1: Verify secrets are properly masked
# Check GitLab pipeline logs
# Expected: Service account key should appear as [MASKED]

# Test 2: Check resource quotas and limits
gcloud compute project-info describe --project=[project-id]
# Expected: Within free tier limits

# Test 3: Performance testing
curl -w "@curl-format.txt" -o /dev/null -s [your-app-url]
# Expected: Response time under 500ms

# Test 4: Security scanning results
# Check pipeline security stage logs
# Expected: No critical vulnerabilities or acceptable risk level
```

### ✅ Monitoring and Logging Verification

```bash
# Test 1: Application logs are being captured
gcloud logging read "resource.type=cloud_run_revision" --limit=5
# Expected: Recent application logs visible

# Test 2: Error handling works correctly
curl https://[your-app-url]/nonexistent-endpoint
# Expected: Proper error response, not server crash

# Test 3: Health checks respond correctly
curl https://[your-app-url]/health
# Expected: Health status and uptime information
```

### 🚨 Common Issues and Quick Fixes

1. **Pipeline Authentication Errors**:
   ```bash
   # Quick test of service account
   echo $GCP_SERVICE_ACCOUNT_KEY | base64 -d | jq .
   # Expected: Valid JSON with service account details
   
   # Verify service account permissions
   gcloud projects get-iam-policy [project-id] --flatten="bindings[].members" --filter="bindings.members:serviceAccount:[sa-email]"
   ```

2. **Deployment Not Accessible**:
   ```bash
   # Check Cloud Run service status
   gcloud run services describe [service-name] --region=[region]
   # Expected: status should show "Ready: True"
   
   # Test local connectivity
   ping google.com
   # Expected: Successful ping (rules out network issues)
   ```

3. **Database Connection Issues**:
   ```bash
   # Test Cloud SQL instance connectivity
   gcloud sql instances describe [instance-name]
   # Expected: state: RUNNABLE
   
   # Test from Cloud Shell
   gcloud sql connect [instance-name] --user=root
   # Expected: MySQL prompt appears
   ```

### 🎯 Success Criteria

Your GitLab-GCP integration is successful when:

1. **✅ Automated Deployments**: Code changes automatically trigger deployments
2. **✅ Multi-Environment Support**: Separate staging and production environments
3. **✅ Security**: Secrets properly managed and masked in logs
4. **✅ Monitoring**: Logs and metrics available for troubleshooting
5. **✅ Performance**: Applications respond quickly and handle load
6. **✅ Reliability**: Deployments succeed consistently without manual intervention

### 📊 Performance Benchmarks

Expected performance metrics for free tier resources:

| Metric | Expected Value | How to Test |
|--------|---------------|-------------|
| Cloud Run Cold Start | < 2 seconds | `curl -w "%{time_total}" [app-url]` |
| Cloud Function Response | < 1 second | `curl -w "%{time_total}" [function-url]` |
| Static Site Load Time | < 500ms | Browser dev tools |
| Database Query Time | < 100ms | Application logs |
| Pipeline Build Time | < 5 minutes | GitLab pipeline duration |

Run these tests periodically to ensure your applications maintain good performance as they scale.

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