# n8n Local Setup with Docker - Step by Step Guide

This guide will walk you through setting up n8n (a workflow automation tool) on your local Windows PC using Docker.

## Prerequisites

Before starting, ensure you have the following installed on your Windows PC:

1. **Docker Desktop for Windows**
   - Download from: https://docs.docker.com/desktop/install/windows-install/
   - Make sure Docker Desktop is running before proceeding

2. **Git** (optional, for version control)
   - Download from: https://git-scm.com/download/win

## Step 1: Create Project Directory

1. Open PowerShell as Administrator
2. Navigate to your desired location (or use the current directory):
   ```powershell
   cd "C:\Users\sinadvd\Documents\VScode\n8n-homelab"
   ```

## Step 2: Create Docker Compose File

Create a `docker-compose.yml` file with the following content:

```yaml
version: '3.8'

services:
  n8n:
    image: docker.n8n.io/n8nio/n8n
    container_name: n8n
    restart: always
    ports:
      - "5678:5678"
    environment:
      - N8N_BASIC_AUTH_ACTIVE=true
      - N8N_BASIC_AUTH_USER=admin
      - N8N_BASIC_AUTH_PASSWORD=password123
      - N8N_HOST=localhost
      - N8N_PORT=5678
      - N8N_PROTOCOL=http
      - WEBHOOK_URL=http://localhost:5678/
    volumes:
      - n8n_data:/home/node/.n8n
      - /var/run/docker.sock:/var/run/docker.sock

volumes:
  n8n_data:
    driver: local
```

## Step 3: Create Environment File (Optional)

For better security, create a `.env` file to store sensitive information:

```env
N8N_BASIC_AUTH_USER=your_username
N8N_BASIC_AUTH_PASSWORD=your_secure_password
N8N_ENCRYPTION_KEY=your_encryption_key_here
```

Then update your `docker-compose.yml` to use environment variables:

```yaml
environment:
  - N8N_BASIC_AUTH_ACTIVE=true
  - N8N_BASIC_AUTH_USER=${N8N_BASIC_AUTH_USER}
  - N8N_BASIC_AUTH_PASSWORD=${N8N_BASIC_AUTH_PASSWORD}
  - N8N_ENCRYPTION_KEY=${N8N_ENCRYPTION_KEY}
```

## Step 4: Start n8n with Docker Compose

1. Open PowerShell and navigate to your project directory
2. Run the following command to start n8n:
   ```powershell
   docker-compose up -d
   ```

3. Wait for the container to start (you can check with):
   ```powershell
   docker-compose ps
   ```

## Step 5: Access n8n Web Interface

1. Open your web browser
2. Navigate to: `http://localhost:5678`
3. Log in with your credentials:
   - Username: `admin` (or your custom username)
   - Password: `password123` (or your custom password)

## Step 6: Verify Installation

1. Once logged in, you should see the n8n workflow editor
2. Create a simple test workflow to ensure everything is working
3. Try creating a workflow with a "Start" node

## Management Commands

### Stop n8n
```powershell
docker-compose down
```

### View Logs
```powershell
docker-compose logs -f n8n
```

### Update n8n
```powershell
docker-compose pull
docker-compose up -d
```

### Restart n8n
```powershell
docker-compose restart n8n
```

### Access Container Shell
```powershell
docker exec -it n8n /bin/sh
```

## Backup and Data Persistence

Your n8n data is stored in a Docker volume named `n8n_data`. To backup your workflows:

1. **Create a backup:**
   ```powershell
   docker run --rm -v n8n-homelab_n8n_data:/data -v ${PWD}:/backup alpine tar czf /backup/n8n-backup.tar.gz -C /data .
   ```

2. **Restore from backup:**
   ```powershell
   docker run --rm -v n8n-homelab_n8n_data:/data -v ${PWD}:/backup alpine tar xzf /backup/n8n-backup.tar.gz -C /data
   ```

## Advanced Configuration

### Using PostgreSQL Database

For production use, consider using PostgreSQL instead of the default SQLite:

```yaml
version: '3.8'

services:
  postgres:
    image: postgres:13
    container_name: n8n-postgres
    restart: always
    environment:
      - POSTGRES_USER=n8n
      - POSTGRES_PASSWORD=n8n_password
      - POSTGRES_DB=n8n
    volumes:
      - postgres_data:/var/lib/postgresql/data

  n8n:
    image: docker.n8n.io/n8nio/n8n
    container_name: n8n
    restart: always
    ports:
      - "5678:5678"
    environment:
      - DB_TYPE=postgresdb
      - DB_POSTGRESDB_HOST=postgres
      - DB_POSTGRESDB_PORT=5432
      - DB_POSTGRESDB_DATABASE=n8n
      - DB_POSTGRESDB_USER=n8n
      - DB_POSTGRESDB_PASSWORD=n8n_password
    depends_on:
      - postgres
    volumes:
      - n8n_data:/home/node/.n8n

volumes:
  n8n_data:
  postgres_data:
```

### Enable HTTPS (Optional)

For HTTPS access, you can use a reverse proxy like Nginx or Traefik, or configure n8n with SSL certificates.

## Troubleshooting

### Common Issues

1. **Port 5678 already in use:**
   - Change the port mapping in docker-compose.yml: `"5679:5678"`
   - Access via `http://localhost:5679`

2. **Docker Desktop not running:**
   - Ensure Docker Desktop is started and running
   - Check Docker Desktop system tray icon

3. **Permission issues:**
   - Run PowerShell as Administrator
   - Ensure Docker has necessary permissions

4. **Cannot access web interface:**
   - Check if container is running: `docker ps`
   - Check container logs: `docker-compose logs n8n`
   - Verify firewall settings

### Getting Help

- **n8n Documentation:** https://docs.n8n.io/
- **n8n Community:** https://community.n8n.io/
- **Docker Documentation:** https://docs.docker.com/

## Security Considerations

1. **Change default credentials** before deploying to production
2. **Use strong encryption keys**
3. **Implement proper firewall rules**
4. **Regularly update n8n and Docker images**
5. **Use HTTPS in production environments**
6. **Backup your data regularly**

## Next Steps

Once n8n is running:
1. Explore the workflow templates
2. Connect to your favorite services and APIs
3. Create your first automation workflow
4. Set up webhooks for external integrations
5. Explore n8n's extensive node library

Happy automating! 🚀
