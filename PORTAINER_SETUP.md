# Portainer Setup Guide

Portainer is a powerful web-based container management platform that provides an intuitive UI for managing your Docker containers, images, volumes, and networks.

## Quick Start

### 1. Start Portainer

```bash
docker-compose -f docker-compose-portainer.yml up -d
```

### 2. Access Portainer Web UI

Open your browser and navigate to:
- **HTTP:** http://localhost:9000
- **HTTPS:** https://localhost:9443

### 3. Initial Setup

On first access, you'll be prompted to:
1. Create an admin user account
2. Set a strong password (minimum 12 characters)
3. Choose your environment (Local Docker)

## Managing the Three Roots Stack with Portainer

### Viewing Existing Containers

1. Navigate to **Home** → **local** → **Containers**
2. You'll see all running containers from your `docker-compose.yml`
3. Filter by stack name or container status

### Managing the MicroRealEstate Stack

```bash
# First, ensure your main stack is running
docker-compose up -d

# Start Portainer
docker-compose -f docker-compose-portainer.yml up -d

# Access Portainer at http://localhost:9000
```

In Portainer:
1. Go to **Stacks**
2. You'll see your existing stack
3. You can:
   - Start/Stop individual services
   - View logs
   - Access container console
   - Monitor resource usage

## Portainer Features

### Container Management
- ✅ Start, stop, restart, pause containers
- ✅ View real-time logs
- ✅ Execute commands inside containers
- ✅ Inspect container details and stats

### Image Management
- ✅ Pull images from Docker Hub
- ✅ Build images from Dockerfile
- ✅ Remove unused images
- ✅ Import/export images

### Volume Management
- ✅ Create and delete volumes
- ✅ Browse volume contents
- ✅ Backup and restore volumes

### Network Management
- ✅ Create custom networks
- ✅ Connect/disconnect containers
- ✅ View network topology

### Stack Deployment
- ✅ Deploy using docker-compose
- ✅ Update existing stacks
- ✅ View stack status

## Common Tasks

### Deploy the Three Roots Stack via Portainer

1. Go to **Stacks** → **Add stack**
2. Name it "three-roots"
3. Paste your `docker-compose.yml` content
4. Add environment variables if needed
5. Click **Deploy the stack**

### View Container Logs

1. Go to **Containers**
2. Click on a container name
3. Click **Logs** tab
4. Enable **Auto-refresh** for real-time logs

### Access Container Shell

1. Go to **Containers**
2. Click on a container
3. Click **Console** tab
4. Select `/bin/bash` or `/bin/sh`
5. Click **Connect**

### Monitor Resource Usage

1. Go to **Dashboard**
2. View CPU, Memory, Network usage
3. Click on any metric for detailed stats

## Connecting to Remote Docker Hosts

If you need to manage Docker on other servers:

### Enable Portainer Agent

1. Uncomment the `portainer-agent` service in `docker-compose-portainer.yml`
2. Deploy agent on remote host:

```bash
docker run -d \
  -p 9001:9001 \
  --name portainer_agent \
  --restart=always \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v /var/lib/docker/volumes:/var/lib/docker/volumes \
  portainer/agent:latest
```

3. In Portainer UI:
   - Go to **Environments**
   - Click **Add environment**
   - Select **Agent**
   - Enter agent details (hostname:9001)
   - Click **Connect**

## Managing Three Roots with Portainer

### View All Services

Navigate to **Containers** to see:
- `redis` - Redis cache
- `mongo` - MongoDB database
- `gateway` - API Gateway
- `authenticator` - Auth service
- `api` - Main API
- `tenantapi` - Tenant API
- `pdfgenerator` - PDF service
- `emailer` - Email service
- `landlord-frontend` - Landlord UI
- `tenant-frontend` - Tenant UI
- `three-roots-frontend` - Your custom frontend

### Update Three Roots Frontend

1. Go to **Images**
2. Pull new image: `yourusername/three-roots-frontend:latest`
3. Go to **Containers** → Select three-roots container
4. Click **Recreate**
5. Select the new image
6. Click **Deploy**

### Backup MongoDB Data

1. Go to **Volumes**
2. Find `mongodb` volume
3. Click **Browse**
4. Use **Download** option
5. Or use container console:

```bash
docker exec mongo mongodump --out /backup/$(date +%Y%m%d)
```

### View Service Logs

Real-time log monitoring:
1. **Containers** → Select service
2. **Logs** tab
3. Enable auto-refresh
4. Filter by search term
5. Download logs if needed

## Security Best Practices

### 1. Change Default Port (Optional)

Edit `docker-compose-portainer.yml`:
```yaml
ports:
  - "9000:9000"  # Change to custom port, e.g., "8090:9000"
```

### 2. Use HTTPS Only

Remove HTTP port and use only HTTPS:
```yaml
ports:
  - "9443:9443"
```

Remove `--http-enabled` from command section.

### 3. Restrict Access

Use a reverse proxy (nginx/caddy) with authentication:
```yaml
services:
  nginx:
    image: nginx:alpine
    ports:
      - "443:443"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
    depends_on:
      - portainer
```

### 4. Regular Updates

Update Portainer regularly:
```bash
docker-compose -f docker-compose-portainer.yml pull
docker-compose -f docker-compose-portainer.yml up -d
```

### 5. Backup Portainer Data

```bash
# Create backup
docker run --rm \
  -v portainer_data:/data \
  -v $(pwd):/backup \
  alpine tar czf /backup/portainer-backup-$(date +%Y%m%d).tar.gz /data

# Restore from backup
docker run --rm \
  -v portainer_data:/data \
  -v $(pwd):/backup \
  alpine tar xzf /backup/portainer-backup-20240101.tar.gz -C /
```

## Integrating with CI/CD

### Webhook for Auto-Deploy

1. In Portainer, go to your container
2. Click **Webhooks**
3. Copy the webhook URL
4. Add to GitHub Actions:

```yaml
- name: Trigger Portainer Webhook
  run: |
    curl -X POST ${{ secrets.PORTAINER_WEBHOOK_URL }}
```

## Troubleshooting

### Portainer Won't Start

```bash
# Check logs
docker logs portainer

# Verify Docker socket access
ls -la /var/run/docker.sock

# Restart Portainer
docker-compose -f docker-compose-portainer.yml restart
```

### Can't Access Web UI

1. Check if container is running:
   ```bash
   docker ps | grep portainer
   ```

2. Verify port is not in use:
   ```bash
   lsof -i :9000
   ```

3. Check firewall rules:
   ```bash
   sudo ufw status
   ```

### Permission Denied Errors

Ensure Docker socket has correct permissions:
```bash
sudo chmod 666 /var/run/docker.sock
# Or add user to docker group
sudo usermod -aG docker $USER
```

## Useful Commands

```bash
# Start Portainer
docker-compose -f docker-compose-portainer.yml up -d

# Stop Portainer
docker-compose -f docker-compose-portainer.yml down

# View Portainer logs
docker logs -f portainer

# Restart Portainer
docker-compose -f docker-compose-portainer.yml restart

# Update Portainer
docker-compose -f docker-compose-portainer.yml pull
docker-compose -f docker-compose-portainer.yml up -d

# Remove Portainer (keeps data)
docker-compose -f docker-compose-portainer.yml down

# Remove Portainer and data
docker-compose -f docker-compose-portainer.yml down -v
```

## Advanced Configuration

### Custom SSL Certificates

Mount your certificates:
```yaml
volumes:
  - /var/run/docker.sock:/var/run/docker.sock
  - portainer_data:/data
  - ./certs/cert.pem:/certs/cert.pem:ro
  - ./certs/key.pem:/certs/key.pem:ro
command: 
  - --sslcert=/certs/cert.pem
  - --sslkey=/certs/key.pem
```

### Using with Traefik

```yaml
labels:
  - "traefik.enable=true"
  - "traefik.http.routers.portainer.rule=Host(`portainer.yourdomain.com`)"
  - "traefik.http.routers.portainer.entrypoints=https"
  - "traefik.http.services.portainer.loadbalancer.server.port=9000"
```

## Next Steps

1. ✅ Deploy Portainer
2. ✅ Create admin account
3. ✅ Connect to local Docker
4. ✅ Explore existing containers
5. ✅ Deploy Three Roots stack
6. ✅ Set up monitoring
7. ✅ Configure webhooks for CI/CD

## Resources

- [Portainer Documentation](https://docs.portainer.io/)
- [Portainer Community Forums](https://community.portainer.io/)
- [Portainer GitHub](https://github.com/portainer/portainer)
- [Docker Documentation](https://docs.docker.com/)

## Support

For issues:
- Check Portainer logs: `docker logs portainer`
- Visit: https://www.portainer.io/community-support
- GitHub Issues: https://github.com/portainer/portainer/issues
