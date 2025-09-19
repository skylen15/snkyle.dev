# snkyle.dev - Multi-Service Docker Stack

A comprehensive Docker-based setup featuring n8n workflow automation, Ollama LLM server, Open WebUI chat interface, and Crawl4AI web scraping service, all managed through Caddy reverse proxy with automatic HTTPS.

## Services Included

-   **n8n** - Workflow automation platform with PostgreSQL database
-   **PostgreSQL with pgvector** - Database for n8n with vector extension support
-   **Ollama** - Local LLM server for running language models
-   **Open WebUI** - Chat interface for Ollama models
-   **Crawl4AI** - Web scraping and crawling service
-   **Caddy** - Reverse proxy with automatic HTTPS

## Prerequisites

Before setting up this project, ensure you have:

-   Docker and Docker Compose installed
-   A domain name with DNS configured to point to your server
-   Basic knowledge of Docker, containers, and server management
-   At least 8GB RAM (recommended for Ollama models)
-   Sufficient disk space for Docker volumes and models

⚠️ **Important**: Self-hosting requires technical knowledge. Mistakes can lead to data loss, security issues, and downtime.

## Quick Start

### 1. Clone the Repository

```bash
git clone https://github.com/skylen15/snkyle.dev.git
cd snkyle.dev
```

### 2. Environment Configuration

Copy the example environment file and configure it:

```bash
cp .env.example .env
```

Edit the `.env` file with your settings:

```env
# Replace with your actual data folder path
DATA_FOLDER=/path/to/your/data

# Your domain configuration
DOMAIN_NAME=yourdomain.com
SUBDOMAIN=n8n

# Timezone for n8n (optional)
GENERIC_TIMEZONE=Asia/Ho_Chi_Minh

# Email for SSL certificate generation
SSL_EMAIL=your-email@example.com

# Secret key for Open WebUI
WEBUI_SECRET_KEY=your-secret-key-here

# PostgreSQL Database Configuration
POSTGRES_USER=postgres
POSTGRES_PASSWORD=your-secure-postgres-password
POSTGRES_DB=n8n
POSTGRES_NON_ROOT_USER=n8n_user
POSTGRES_NON_ROOT_PASSWORD=your-secure-n8n-password
```

### 3. SSH Key Setup for GitHub Actions

Before configuring GitHub secrets, you need to set up SSH key authentication on your server:

#### On Your Server:

1. **Generate an SSH key pair** (if you don't have one):

```bash
ssh-keygen -t ed25519 -C "github-actions-deployment"
```

-   Press Enter to save in default location (`~/.ssh/id_ed25519`)
-   Optionally set a passphrase (leave empty for automated deployment)

2. **Add the public key to authorized_keys**:

```bash
cat ~/.ssh/id_ed25519.pub >> ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
chmod 700 ~/.ssh
```

3. **Get the private key** for GitHub secrets:

```bash
cat ~/.ssh/id_ed25519
```

-   Copy the entire output (including `-----BEGIN OPENSSH PRIVATE KEY-----` and `-----END OPENSSH PRIVATE KEY-----`)
-   This will be your `SSH_PRIVATE_KEY` secret

#### Security Best Practices:

-   **Restrict SSH access** by editing `/etc/ssh/sshd_config`:

```bash
sudo nano /etc/ssh/sshd_config
```

Add or modify these lines:

```
PasswordAuthentication no
PubkeyAuthentication yes
PermitRootLogin no
```

-   **Restart SSH service**:

```bash
sudo systemctl restart ssh
```

-   **Test SSH connection** from your local machine:

```bash
ssh -i ~/.ssh/id_ed25519 username@your-server-ip
```

### 4. GitHub Repository Secrets

This project uses GitHub Actions for automatic deployment. Set up the following secrets in your GitHub repository:

1. Go to your GitHub repository
2. Navigate to **Settings** → **Secrets and variables** → **Actions**
3. Add the following repository secrets:

| Secret Name       | Description                          | Example Value                            |
| ----------------- | ------------------------------------ | ---------------------------------------- |
| `SSH_HOST`        | Your server's IP address or hostname | `123.456.789.123`                        |
| `SSH_USERNAME`    | SSH username for your server         | `ubuntu`                                 |
| `SSH_PRIVATE_KEY` | Private SSH key for server access    | `-----BEGIN OPENSSH PRIVATE KEY-----...` |
| `SSH_PORT`        | SSH port (optional, defaults to 22)  | `22`                                     |

### 5. Server Preparation

On your target server, ensure:

```bash
# Create the app directory
mkdir -p /home/your-username/app

# Create your .env file on the server
nano /home/your-username/app/.env
```

Add your environment configuration to the `.env` file on the server with your actual values.

### 6. Deploy

Once secrets are configured, deployment happens automatically:

-   Push to the `main` branch triggers automatic deployment
-   GitHub Actions will sync files and restart services
-   Monitor the deployment in the **Actions** tab of your repository

## Service Access

Once running, your services will be available at:

-   **n8n**: `https://n8n.yourdomain.com` (replace with your domain)
-   **Open WebUI**: `https://chat-ai.yourdomain.com`
-   **Ollama API**: `https://ollama.yourdomain.com`
-   **Crawl4AI**: `https://crawl.yourdomain.com`

## Configuration Details

### Domain Configuration

The Caddyfile is configured for the following subdomains:

-   `n8n.yourdomain.com` → n8n service
-   `chat-ai.yourdomain.com` → Open WebUI
-   `ollama.yourdomain.com` → Ollama API
-   `crawl.yourdomain.com` → Crawl4AI service

Update the `caddy_config/Caddyfile` to match your domain structure.

### n8n Configuration

n8n is configured with:

-   Production environment
-   HTTPS protocol
-   PostgreSQL database backend with pgvector support
-   Webhook support
-   File access through `/files` directory
-   Resource limits: 1 CPU, 1GB RAM
-   Health check dependency on PostgreSQL

### PostgreSQL Configuration

PostgreSQL is configured with:

-   pgvector extension for vector operations
-   Dedicated database and user for n8n
-   Health checks for service dependencies
-   Persistent data storage in Docker volume
-   Automatic user and permissions setup via init script

### Ollama Configuration

Ollama is configured with:

-   Security hardening (read-only, no new privileges)
-   Resource limits: 2 CPUs, 5GB RAM
-   Model storage in Docker volume

### Open WebUI Configuration

Connected to Ollama with:

-   Automatic Ollama detection
-   Persistent data storage
-   Secret key authentication

## Automated Deployment

This project includes GitHub Actions for automated deployment to your production server. The workflow:

1. **Triggers** on push to `main` branch
2. **Syncs files** to your server via rsync (excludes `.env` file)
3. **Creates Docker volumes** if they don't exist
4. **Pulls latest images** and updates services
5. **Cleans up** unused Docker images

The deployment process handles:

-   Directory setup
-   Docker volume creation
-   Service updates and restarts
-   Image cleanup

Monitor deployments in the **Actions** tab of your GitHub repository.

## Management Commands

### View Logs

```bash
# All services
docker-compose logs -f

# Specific service
docker-compose logs -f n8n
docker-compose logs -f ollama
docker-compose logs -f open-webui
```

### Stop Services

```bash
docker-compose down
```

### Update Services

```bash
docker-compose pull
docker-compose up -d
```

### Backup Data

```bash
# Backup Docker volumes
docker run --rm -v n8n_data:/data -v $(pwd):/backup ubuntu tar czf /backup/n8n_backup.tar.gz /data
docker run --rm -v postgres_data:/data -v $(pwd):/backup ubuntu tar czf /backup/postgres_backup.tar.gz /data
docker run --rm -v ollama:/data -v $(pwd):/backup ubuntu tar czf /backup/ollama_backup.tar.gz /data

# Alternative: PostgreSQL database dump
docker-compose exec postgres pg_dump -U postgres n8n > n8n_database_backup.sql
```

## Troubleshooting

### Common Issues

1. **Services not accessible**: Check DNS configuration and firewall settings
2. **SSL certificate issues**: Verify email configuration and domain ownership
3. **Ollama models not loading**: Ensure sufficient RAM and disk space
4. **n8n workflows failing**: Check file permissions in `local_files` directory
5. **n8n database connection issues**: Verify PostgreSQL credentials and container health
6. **PostgreSQL startup issues**: Check logs and ensure sufficient disk space

### Health Checks

```bash
# Check service status
docker-compose ps

# Check resource usage
docker stats

# Test connectivity
curl -I https://n8n.yourdomain.com

# Check PostgreSQL connectivity
docker-compose exec postgres pg_isready -U postgres -d n8n

# View specific service logs
docker-compose logs postgres
docker-compose logs n8n
```

## Security Considerations

-   All services run behind Caddy with automatic HTTPS
-   Ollama runs with security hardening
-   PostgreSQL database with dedicated user credentials
-   Regular updates are recommended
-   Monitor resource usage
-   Backup data regularly

## Resource Requirements

### Minimum Requirements

-   **CPU**: 2 cores
-   **RAM**: 10GB (increased for PostgreSQL database)
-   **Storage**: 50GB free space

### Recommended Requirements

-   **CPU**: 4+ cores
-   **RAM**: 16GB+ (PostgreSQL and Ollama can be memory intensive)
-   **Storage**: 100GB+ SSD

## Support

For issues and questions:

-   [n8n Community Forums](https://community.n8n.io/)
-   [Ollama GitHub Issues](https://github.com/ollama/ollama/issues)
-   [Open WebUI Documentation](https://docs.openwebui.com/)
-   [Crawl4AI Documentation](https://github.com/unclecode/crawl4ai)

## License

This project is licensed under the terms specified in the LICENSE file.
