# GitHub Actions Setup Guide

## Overview
Your repository is configured with GitHub Actions to automatically test, build, and deploy your Spring Boot application using Docker.

**Workflow:** `maven-docker.yml`
- **Trigger:** Automatically runs when you push to the `main` branch
- **Steps:** Test → Build → Deploy

---

## Required Setup

### 1. GitHub Secrets Configuration
Navigate to your repository settings and add the following secrets:

#### Docker Hub Credentials (Required for Docker builds)
- **DOCKER_USERNAME**: Your Docker Hub username
- **DOCKER_PASSWORD**: Your Docker Hub password (or access token)

Steps:
1. Go to GitHub repository → Settings → Secrets and variables → Actions
2. Click "New repository secret"
3. Add the following secrets:

```
DOCKER_USERNAME = <your-docker-hub-username>
DOCKER_PASSWORD = <your-docker-hub-token>
```

#### Deployment Server Credentials (Required for deployment)
- **SERVER_HOST**: Your production/staging server IP or hostname
- **SERVER_USER**: SSH username for your server
- **SERVER_PASSWORD**: SSH password for your server

```
SERVER_HOST = <your-server-ip-or-domain>
SERVER_USER = <your-ssh-username>
SERVER_PASSWORD = <your-ssh-password>
```

#### Database Credentials (Used in deployment)
- **DB_URL**: Database connection URL
- **DB_USERNAME**: Database username
- **DB_PASSWORD**: Database password

```
DB_URL = jdbc:mysql://your-db-host:3306/fazon_db_v1
DB_USERNAME = root
DB_PASSWORD = admin123
```

#### Ingenico API Credentials (Used in deployment)
- **INGENICO_API_KEY_ID**: Your Ingenico API key ID
- **INGENICO_API_SECRET_KEY**: Your Ingenico API secret key

```
INGENICO_API_KEY_ID = <your-api-key-id>
INGENICO_API_SECRET_KEY = <your-api-secret>
```

---

## Workflow Steps Explained

### Step 1: Test
- Runs on Ubuntu latest
- Sets up Java 21
- Executes: `mvn clean test`
- ✅ Must pass before proceeding to build

### Step 2: Build
- Only runs if tests pass
- Builds Docker image
- Pushes to Docker Hub: `{DOCKER_USERNAME}/simplybyte-springboot:latest`

### Step 3: Deploy
- Only runs if build succeeds
- Connects to your server via SSH
- Pulls latest Docker image
- Stops old container
- Runs new container with environment variables

---

## Docker Image Details

**Current Configuration:**
- Image Name: `{DOCKER_USERNAME}/simplybyte-springboot:latest`
- Container Name: `simplybyte-container`
- Port Mapping: `8090:8090`
- Environment Variables: DB credentials & Ingenico API keys

**Dockerfile Requirements:**
Ensure your `Dockerfile` is in the root directory (already present).

---

## How to Trigger the Workflow

### Automatic Trigger
- Simply push code to the `main` branch:
```bash
git add .
git commit -m "Your message"
git push origin main
```

### Manual Trigger (Optional)
1. Go to GitHub repository → Actions
2. Select "Maven Test, Build and Docker Push"
3. Click "Run workflow"

---

## Monitoring Workflow Execution

1. Go to your GitHub repository
2. Click "Actions" tab
3. View the workflow run:
   - ✅ Green checkmark = Success
   - ❌ Red X = Failed
   - Click on the run to see detailed logs

---

## Troubleshooting

### Test Failures
- Check logs in Actions → Test job
- Ensure your database is running and accessible
- Review test output for specific errors

### Build Failures
- Verify pom.xml is correct
- Check Java version compatibility (Java 21)
- Ensure Dockerfile is correct and in root directory

### Docker Push Failures
- Verify DOCKER_USERNAME and DOCKER_PASSWORD secrets
- Ensure Docker Hub account has access
- Check Docker Hub storage quota

### Deployment Failures
- Verify server credentials (SERVER_HOST, SERVER_USER, SERVER_PASSWORD)
- Ensure SSH access is enabled on your server
- Check database connectivity from the server
- Verify firewall rules allow port 8090

---

## Next Steps

1. **Create Docker Hub Account** (if not already done)
   - Visit https://hub.docker.com
   - Create free account
   - Generate access token

2. **Set Up GitHub Secrets**
   - Follow the steps in "Required Setup" section above

3. **Test the Workflow**
   - Make a small change to your code
   - Push to main branch
   - Monitor the Actions tab

4. **Verify Deployment**
   - Once deployed, access your app:
   - `http://{your-server-ip}:8090/api/health`

---

## Current Configuration Summary

| Component | Value |
|-----------|-------|
| Build Tool | Maven 3.9.x |
| Java Version | 21 (Temurin) |
| App Port (Container) | 8090 |
| App Port (Host) | 8090 |
| Database | MySQL (localhost:3306) |
| Container Registry | Docker Hub |
| Deployment Method | SSH + Docker |

---

For more details, see `.github/workflows/maven-docker.yml`
