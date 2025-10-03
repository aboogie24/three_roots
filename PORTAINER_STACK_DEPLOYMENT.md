# Portainer Stack Deployment Guide

This guide explains how to deploy the Three Roots + MicroRealEstate stack using Portainer's web interface.

## Prerequisites

1. ✅ Portainer is installed and running
2. ✅ Docker Hub contains your Three Roots frontend image
3. ✅ You have admin access to Portainer

## Quick Start

### 1. Access Portainer

Open your browser and navigate to:
```
http://localhost:9000
```

Log in with your admin credentials.

### 2. Navigate to Stacks

1. Click on **Home** → **local** environment
2. Click on **Stacks** in the left sidebar
3. Click **+ Add stack** button

### 3. Create the Stack

#### Name Your Stack
- **Name:** `three-roots-stack`

#### Build Method
Select **Web editor**

#### Paste Stack Configuration

Copy the entire contents of `docker-compose-stack.yml` and paste it into the editor.

### 4. Add Environment Variables

Scroll down to the **Environment variables** section and choose one of these methods:

#### Method A: Advanced Mode (Recommended)

1. Toggle to **Advanced mode**
2. Copy the contents of `.env-stack` file
3. Paste into the text area
4. Update all placeholder values with your actual configuration

#### Method B: Simple Mode

Click **+ add environment variable** for each variable:

**Required Variables:**

| Variable | Example Value | Description |
|----------|---------------|-------------|
| `DOCKER_HUB_USERNAME` | `yourusername` | Your Docker Hub username |
| `THREE_ROOTS_VERSION` | `latest` | Three Roots image tag |
| `THREE_ROOTS_PORT` | `3000` | Port for Three Roots website |
| `MRE_VERSION` | `latest` | MicroRealEstate version |
| `APP_DOMAIN` | `localhost` | Your domain name |
| `APP_PORT` | `80` | Main application port |
| `APP_PROTOCOL` | `http` | Protocol (http/https) |
| `MONGO_URL` | `mongodb://mongo/mredb` | MongoDB connection string |
| `REDIS_PASSWORD` | `your_secure_password` | Redis password |
| `ACCESS_TOKEN_SECRET` | `your_secret_here` | Access token secret |
| `REFRESH_TOKEN_SECRET` | `your_secret_here` | Refresh token secret |
| `RESET_TOKEN_SECRET` | `your_secret_here` | Reset token secret |
| `APPCREDZ_TOKEN_SECRET` | `your_secret_here` | App credentials secret |
| `CIPHER_KEY` | `your_cipher_key` | Encryption key |
| `CIPHER_IV_KEY` | `your_cipher_iv_key` | Encryption IV key |
| `GMAIL_EMAIL` | `your@gmail.com` | Gmail address |
| `GMAIL_APP_PASSWORD` | `app_password` | Gmail app password |
| `EMAIL_FROM` | `noreply@threeroots.com` | From email address |
| `EMAIL_REPLY_TO` | `support@threeroots.com` | Reply-to email |

### 5. Deploy the Stack

1. Review your configuration
2. Click **Deploy the stack** button
3. Wait for Portainer to pull images and start containers

### 6. Monitor Deployment

Portainer will show the deployment progress. You can:
- View real-time container status
- Check logs for any errors
- Monitor resource usage

## Accessing Your Services

Once deployed, access your services at:

| Service | URL | Description |
|---------|-----|-------------|
| **Three Roots Website** | http://localhost:3000 | Your custom investment website |
| **MRE Landlord Portal** | http://localhost/landlord | Property management dashboard |
| **MRE Tenant Portal** | http://localhost/tenant | Tenant portal |
| **Gateway API** | http://localhost:8081 | API Gateway |

## Post-Deployment Steps

### 1. Verify All Containers Are Running

1. Go to **Containers** in Portainer
2. Check that all services show "running" status
3. Look for any containers with errors

### 2. Check Container Logs

If any service fails:
1. Click on the container name
2. Go to **Logs** tab
3. Review error messages
4. Fix configuration and redeploy

### 3. Test the Application

1. Open http://localhost:3000 (Three Roots website)
2. Open http://localhost/landlord (MRE landlord portal)
3. Create a test account
4. Verify email delivery works

## Updating the Stack

### Update Environment Variables

1. Go to **Stacks** → Select `three-roots-stack`
2. Click **Editor**
3. Scroll to **Environment variables**
4. Update values as needed
5. Click **Update the stack**

### Update Stack Configuration

1. Go to **Stacks** → Select `three-roots-stack`
2. Click **Editor**
3. Modify the docker-compose content
4. Click **Update the stack**

### Update Docker Images

To pull the latest images:

1. Go to **Stacks** → Select `three-roots-stack`
2. Check **Pull latest image versions**
3. Click **Update the stack**

Or update individual services:

1. Go to **Containers**
2. Select the container
3. Click **Recreate**
4. Check **Pull latest image**
5. Click **Recreate**

## Managing the Stack

### View Container Logs

1. **Containers** → Click container name
2. **Logs** tab
3. Enable **Auto-refresh** for real-time logs

### Access Container Console

1. **Containers** → Click container name
2. **Console** tab
3. Select shell (`/bin/bash` or `/bin/sh`)
4. Click **Connect**

### Monitor Resources

1. **Containers** → Click container name
2. **Stats** tab
3. View CPU, Memory, Network usage

### Backup Data

#### MongoDB Backup
```bash
# In Portainer console for mongo container
mongodump --out /backup/$(date +%Y%m%d)
```

#### Download Volume Data
1. **Volumes** → Select volume
2. **Browse** → Navigate files
3. Use **Download** for specific files

## Troubleshooting

### Stack Fails to Deploy

**Check:**
- All required environment variables are set
- Docker Hub credentials are correct (if using private images)
- Port conflicts with existing containers
- Review deployment logs in Portainer

**Solution:**
1. Delete the failed stack
2. Fix configuration issues
3. Redeploy

### Container Keeps Restarting

1. View container logs for error messages
2. Check environment variables
3. Verify dependent services are running
4. Check resource limits

### Cannot Access Services

**Check:**
- Container is running: **Containers** list
- Port mappings are correct
- Firewall allows the ports
- Network connectivity between containers

**Test connectivity:**
```bash
# From container console
curl http://gateway:8080/health
ping mongo
```

### Email Not Working

1. Verify email credentials in environment variables
2. Check emailer container logs
3. Test SMTP connection from emailer container
4. Verify email provider allows app passwords

## Stack Configuration Reference

### Ports Exposed

| Port | Service | Description |
|------|---------|-------------|
| 3000 | three-roots-web | Three Roots website |
| 80 | reverse-proxy | Main proxy (Caddy) |
| 443 | reverse-proxy | HTTPS (Caddy) |
| 8081 | gateway | Gateway API |

### Volumes Created

| Volume | Purpose | Backup Priority |
|--------|---------|-----------------|
| `three_roots_redis_data` | Redis cache | Low |
| `three_roots_mongodb_data` | Database | **HIGH** |
| `three_roots_backup_data` | Backup storage | Medium |

### Networks

| Network | Type | Purpose |
|---------|------|---------|
| `three_roots_network` | Bridge | Internal service communication |

## Advanced Configuration

### Using Custom Domain

Update environment variables:
```
APP_DOMAIN=yourdomain.com
APP_PROTOCOL=https
APP_PORT=443
```

### HTTPS with SSL Certificates

1. Add volumes for certificates in reverse-proxy service
2. Configure Caddy with your certificates
3. Update APP_PROTOCOL to https

### Scaling Services

Some services can be scaled:
1. **Containers** → Select service
2. **Duplicate/Edit**
3. Change container name and ports
4. Add to load balancer

## Security Best Practices

1. ✅ **Change all default secrets** in production
2. ✅ **Use strong passwords** (min 32 characters)
3. ✅ **Enable HTTPS** with valid certificates
4. ✅ **Restrict port access** with firewall rules
5. ✅ **Regular backups** of MongoDB data
6. ✅ **Monitor logs** for suspicious activity
7. ✅ **Update images** regularly for security patches

## Generating Strong Secrets

Use this command to generate secure secrets:

```bash
# Generate a 32-character base64 secret
openssl rand -base64 32

# Generate multiple secrets at once
for i in {1..5}; do openssl rand -base64 32; done
```

Replace placeholder values in `.env-stack`:
- `ACCESS_TOKEN_SECRET`
- `REFRESH_TOKEN_SECRET`
- `RESET_TOKEN_SECRET`
- `APPCREDZ_TOKEN_SECRET`
- `CIPHER_KEY`
- `CIPHER_IV_KEY`
- `REDIS_PASSWORD`

## Production Checklist

Before deploying to production:

- [ ] All secrets changed from defaults
- [ ] Email configuration tested
- [ ] Database backups configured
- [ ] HTTPS enabled with valid certificates
- [ ] Domain name configured correctly
- [ ] Firewall rules configured
- [ ] Monitoring set up
- [ ] Resource limits set appropriately
- [ ] Documentation updated with production values

## Backup and Restore

### Create Backup

1. **MongoDB:**
   ```bash
   docker exec mre-mongo mongodump --out /backup/prod-backup-$(date +%Y%m%d)
   ```

2. **Download from Portainer:**
   - **Volumes** → `three_roots_backup_data`
   - **Browse** → Select backup folder
   - Download backup files

### Restore from Backup

1. Upload backup to volume
2. Access mongo container console:
   ```bash
   mongorestore /backup/prod-backup-20240101
   ```

## Support and Resources

- **Portainer Docs:** https://docs.portainer.io
- **MicroRealEstate:** https://github.com/microrealestate/microrealestate
- **Docker Compose:** https://docs.docker.com/compose

## Common Issues and Solutions

### "Cannot pull image"
- Verify Docker Hub username in environment variables
- Check if image exists: `yourusername/three-roots-frontend:latest`
- Ensure image is public or credentials are configured

### "Port already in use"
- Change port in environment variables
- Stop conflicting containers
- Use different port mapping

### "Service unhealthy"
- Check service logs
- Verify dependent services are running
- Restart the service
- Check resource availability

### "Network not found"
- Stack might not have created network properly
- Delete stack and redeploy
- Manually create network if needed

## Next Steps

1. ✅ Deploy stack in Portainer
2. ✅ Verify all services are running
3. ✅ Test Three Roots website
4. ✅ Test MRE landlord portal
5. ✅ Configure email settings
6. ✅ Set up regular backups
7. ✅ Configure monitoring
8. ✅ Document your deployment

---

**Need Help?** Check Portainer logs and container logs for detailed error messages.
