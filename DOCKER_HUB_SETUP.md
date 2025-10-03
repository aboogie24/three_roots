# Docker Hub Setup for GitHub Actions

This guide explains how to configure GitHub Actions to automatically build and push the Three Roots frontend Docker image to Docker Hub.

## Prerequisites

1. A Docker Hub account (create one at https://hub.docker.com)
2. Admin access to the GitHub repository

## Step 1: Create Docker Hub Access Token

1. Log in to Docker Hub (https://hub.docker.com)
2. Click on your username in the top right corner
3. Select **Account Settings**
4. Go to **Security** tab
5. Click **New Access Token**
6. Enter a description (e.g., "GitHub Actions Three Roots")
7. Select **Read, Write, Delete** permissions
8. Click **Generate**
9. **IMPORTANT:** Copy the token immediately - you won't be able to see it again!

## Step 2: Add Secrets to GitHub Repository

1. Go to your GitHub repository
2. Click **Settings** → **Secrets and variables** → **Actions**
3. Click **New repository secret**
4. Add the following secrets:

### Required Secrets

#### `DOCKER_HUB_USERNAME`
- **Value:** Your Docker Hub username (not your email)
- **Example:** `yourusername`

#### `DOCKER_HUB_ACCESS_TOKEN`
- **Value:** The access token you generated in Step 1
- **Example:** `dckr_pat_xxxxxxxxxxxxxxxxxxxxx`

## Step 3: Create GitHub Environments (Optional but Recommended)

For better control over deployments:

1. Go to **Settings** → **Environments**
2. Create two environments:
   - **production** (for main branch)
   - **development** (for develop branch)
3. For production, you can add protection rules:
   - Required reviewers
   - Wait timer
   - Deployment branches (restrict to main)

## How the Workflow Works

### Automatic Triggers

The workflow automatically runs when:

1. **Push to `main` branch** → Builds and pushes with tags: `latest`, `prod-{sha}`
2. **Push to `develop` branch** → Builds and pushes with tags: `dev`, `dev-{sha}`
3. **Pull Request** → Builds only (doesn't push to Docker Hub)

### Manual Trigger

You can manually trigger the workflow:

1. Go to **Actions** tab in GitHub
2. Select **Build and Push Three Roots to Docker Hub**
3. Click **Run workflow**
4. Choose environment (production or development)
5. Click **Run workflow**

## Docker Image Tags

### Production (main branch)
```bash
# Latest stable version
docker pull yourusername/three-roots-frontend:latest

# Specific commit
docker pull yourusername/three-roots-frontend:prod-abc1234
```

### Development (develop branch)
```bash
# Latest development version
docker pull yourusername/three-roots-frontend:dev

# Specific commit
docker pull yourusername/three-roots-frontend:dev-abc1234
```

## Using the Docker Image

### Standalone Deployment

```bash
# Pull the image
docker pull yourusername/three-roots-frontend:latest

# Run the container
docker run -d \
  -p 8080:80 \
  --name three-roots \
  yourusername/three-roots-frontend:latest

# Access at http://localhost:8080
```

### Docker Compose Integration

Add to your `docker-compose.yml`:

```yaml
services:
  three-roots-frontend:
    image: yourusername/three-roots-frontend:latest
    ports:
      - "8080:80"
    networks:
      - net
    restart: unless-stopped
```

## Workflow Features

✅ **Multi-platform builds** - Supports both AMD64 and ARM64 architectures
✅ **Build caching** - Uses GitHub Actions cache for faster builds
✅ **Environment-specific tags** - Separate tags for production and development
✅ **Automated deployment summaries** - See deployment details in GitHub Actions
✅ **Pull request testing** - Builds are tested on PRs without pushing to Docker Hub

## Troubleshooting

### Workflow fails with "unauthorized" error
- Verify `DOCKER_HUB_USERNAME` matches your Docker Hub username exactly
- Regenerate `DOCKER_HUB_ACCESS_TOKEN` and update the secret
- Check that the access token has write permissions

### Image not appearing in Docker Hub
- Check that the workflow completed successfully in GitHub Actions
- Verify you're logged into the correct Docker Hub account
- Check the repository visibility (public vs private)

### Build fails
- Review the GitHub Actions logs for specific error messages
- Ensure all files in `three_roots/` directory are correct
- Check that the Dockerfile is valid

## Branch Strategy

### Main Branch (Production)
- Protected branch with required reviews
- Deploys to Docker Hub with `:latest` tag
- Used for production deployments

### Develop Branch (Development)
- Active development branch
- Deploys to Docker Hub with `:dev` tag  
- Used for testing and staging

### Feature Branches
- Pull requests trigger test builds only
- No automatic deployment to Docker Hub

## Security Best Practices

1. ✅ Use access tokens instead of passwords
2. ✅ Store credentials as GitHub Secrets (never commit them)
3. ✅ Limit access token permissions to only what's needed
4. ✅ Rotate access tokens periodically
5. ✅ Use protected branches for production
6. ✅ Enable two-factor authentication on Docker Hub

## Next Steps

After setting up:

1. Push changes to trigger the workflow
2. Check GitHub Actions tab for build status
3. Verify the image appears in Docker Hub
4. Test pulling and running the image
5. Update `docker-compose.yml` to use your Docker Hub image

## Support

For issues with:
- **GitHub Actions:** Check the Actions tab logs
- **Docker Hub:** Visit https://hub.docker.com/support
- **Three Roots Frontend:** Create an issue in this repository
