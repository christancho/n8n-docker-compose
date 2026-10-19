# n8n Docker Compose Stack

A production-ready n8n automation platform setup using Docker Compose, optimized for Portainer deployment. This stack includes n8n with PostgreSQL database, Redis caching, and comprehensive configuration for reliable automation workflows.

## 🚀 Features

- **Production-ready n8n setup** with PostgreSQL database
- **Redis caching system** for improved performance
- **Optimized for Synology NAS** and other resource-constrained environments
- **Health checks** and resource limits with improved reliability
- **Security hardening** with proper user permissions
- **Performance optimizations** for better execution
- **Comprehensive webhook configuration** to fix localhost issues
- **Worker support** (commented out for free version compatibility)
- **Complete directory structure** with .gitkeep files

## 📋 Prerequisites

- Docker and Docker Compose installed
- Portainer (for stack deployment)
- At least 4GB RAM available
- 10GB+ free disk space

## 🔧 Environment Variables

This project includes a comprehensive `env.example` file with detailed documentation for all environment variables. Copy it to `.env` and update with your values:

```bash
cp env.example .env
```

### Required Variables

```bash
# Database Configuration
POSTGRES_USER=postgres
POSTGRES_PASSWORD=your_secure_postgres_password
POSTGRES_DB=n8n
POSTGRES_NON_ROOT_USER=n8n_user
POSTGRES_NON_ROOT_PASSWORD=your_secure_n8n_user_password

# n8n Configuration
ENCRYPTION_KEY=your_32_character_encryption_key_here
TIMEZONE=America/Toronto
PUID=1000
PGID=1000

# Webhook and Host Configuration (Fixes localhost webhook issue)
WEBHOOK_URL=https://your-domain.com
N8N_HOST=your-domain.com
N8N_PORT=5678
N8N_PROTOCOL=https

# Directory Configuration
N8N_DIR=/opt/n8n
```

### Optional Variables

```bash
# Logging
N8N_LOG_LEVEL=info

# Security
N8N_SECURE_COOKIE=false
N8N_TRUST_PROXY=true

# Performance
N8N_DEFAULT_BINARY_DATA_MODE=filesystem
N8N_EXECUTION_DATA_PRUNE_MAX_COUNT=10000
N8N_EXECUTION_DATA_MAX_AGE=168
```

> **📖 For complete documentation**: See `env.example` for detailed explanations of all variables, including security notes and best practices.

## 🐳 Quick Start with Docker Compose

1. **Clone the repository:**
   ```bash
   git clone <repository-url>
   cd n8n-docker-compose
   ```

2. **Create environment file:**
   ```bash
   cp env.example .env
   # Edit .env with your configuration
   ```

3. **Create required directories:**
   ```bash
   sudo mkdir -p /opt/n8n/{n8n/files,postgres/data,redis}
   sudo chown -R 1000:1000 /opt/n8n
   ```

4. **Start the stack:**
   ```bash
   docker-compose up -d
   ```

5. **Access n8n:**
   - Open your browser to `http://localhost:5678` (or your configured domain)
   - Complete the initial setup wizard

## 🔗 Webhook Configuration

This stack includes proper webhook configuration to prevent the common "localhost" webhook URL issue:

- **`WEBHOOK_URL`**: Set to your full domain (e.g., `https://n8n.yourdomain.com`)
- **`N8N_HOST`**: Set to your domain name
- **`N8N_PROTOCOL`**: Set to `https` for production
- **`N8N_PORT`**: Default is `5678`

This ensures webhooks use your actual domain instead of localhost.

## 🚢 Deploy with Portainer

### Method 1: Stack Deployment (Recommended)

1. **Access Portainer:**
   - Open your Portainer web interface
   - Navigate to **Stacks** in the left sidebar

2. **Create New Stack:**
   - Click **"Add stack"**
   - Name your stack (e.g., "n8n")

3. **Configure Stack:**
   - **Build method:** Select "Repository"
   - **Repository URL:** Enter your Git repository URL
   - **Reference:** Enter branch name (usually "main")
   - **Compose path:** Leave as "docker-compose.yml"

4. **Environment Variables:**
   - Click **"Add environment variable"** for each required variable
   - Add all variables from the `.env` file
   - **Important:** Set `N8N_DIR` to a path accessible by Portainer (e.g., `/opt/n8n`)

5. **Deploy:**
   - Click **"Deploy the stack"**
   - Monitor the deployment in the logs

### Method 2: Upload Compose File

1. **Prepare Files:**
   - Download `docker-compose.yml` to your local machine
   - Create `.env` file with your configuration

2. **Upload to Portainer:**
   - Go to **Stacks** → **Add stack**
   - Select **"Upload"** as build method
   - Upload the `docker-compose.yml` file
   - Add environment variables manually

3. **Deploy:**
   - Review configuration
   - Click **"Deploy the stack"**

## 📁 Directory Structure

```
/opt/n8n/
├── n8n/                 # n8n application data
│   └── files/          # File uploads and binary data
├── postgres/
│   └── data/           # PostgreSQL database files
└── redis/              # Redis data directory
```

## 🔧 Configuration Details

### Services Overview

- **n8n_main**: Main n8n application (port 5678)
- **n8n_worker**: Worker node for background processing
- **n8n_postgres**: PostgreSQL 16 database
- **n8n_redis**: Redis 7 queue system

### Resource Limits

- **n8n_main**: 1.5GB memory limit, 768MB reservation
- **n8n_worker**: 768MB memory limit, 384MB reservation
- **n8n_postgres**: 2.5GB memory limit, 1.5GB reservation
- **n8n_redis**: 1GB memory limit, 256MB reservation

### Network Configuration

- **Network**: `n8n_net` (172.20.0.0/16)
- **Driver**: Bridge
- **Internal communication**: All services communicate via internal network

## 🔒 Security Considerations

1. **Change default passwords** in the `.env` file
2. **Generate a secure encryption key** (32 characters minimum)
3. **Use HTTPS** in production (set `N8N_PROTOCOL=https`)
4. **Configure proper firewall rules** for port 5678
5. **Regular backups** of the PostgreSQL data directory

## 📊 Monitoring and Maintenance

### Health Checks

- **PostgreSQL**: `pg_isready` check every 10 seconds
- **Redis**: `redis-cli ping` check every 10 seconds
- **n8n**: Health check available (commented out by default)

### Logs

View logs for each service:
```bash
# All services
docker-compose logs -f

# Specific service
docker-compose logs -f n8n
docker-compose logs -f n8n_postgres
docker-compose logs -f n8n_redis
```

### Backup

```bash
# Backup PostgreSQL database
docker-compose exec n8n_postgres pg_dump -U postgres n8n > backup.sql

# Backup n8n data
tar -czf n8n-backup.tar.gz /opt/n8n/n8n/
```

## 🚨 Troubleshooting

### Common Issues

1. **Permission denied errors:**
   ```bash
   sudo chown -R 1000:1000 /opt/n8n
   ```

2. **Database connection failed:**
   - Check PostgreSQL logs: `docker-compose logs n8n_postgres`
   - Verify environment variables
   - Ensure database is healthy before n8n starts

3. **Memory issues:**
   - Adjust resource limits in `docker-compose.yml`
   - Monitor system resources in Portainer

4. **Webhook shows localhost instead of domain:**
   - Verify `WEBHOOK_URL`, `N8N_HOST`, `N8N_PROTOCOL` are correctly set
   - Ensure all webhook environment variables match your domain
   - Check firewall and port forwarding

5. **Container shows as unhealthy:**
   - Check container logs: `docker logs n8n_main`
   - Check the healthcheck result: `docker inspect --format '{{json .State.Health}}' n8n_main`
   - Verify n8n responds (the image has no curl/wget, so use Node): `docker exec n8n_main node -e "require('http').get('http://localhost:5678/healthz', r => console.log(r.statusCode))"`
   - Healthcheck may be temporarily disabled for troubleshooting

6. **Portainer deployment fails:**
   - Ensure all environment variables use simple `${VAR}` syntax
   - Avoid `${VAR:-default}` syntax as Portainer doesn't support it
   - Check that all required variables are set

### Logs and Debugging

```bash
# Check service status
docker-compose ps

# View detailed logs
docker-compose logs --tail=100 n8n

# Access container shell
docker-compose exec n8n sh
```

## 🔄 Updates

To update the stack:

1. **Pull latest changes:**
   ```bash
   git pull origin main
   ```

2. **Update in Portainer:**
   - Go to your stack
   - Click **"Editor"**
   - Update the compose file
   - Click **"Update the stack"**

## 📝 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Submit a pull request

## 📞 Support

For issues and questions:
- Check the [troubleshooting section](#-troubleshooting)
- Review n8n documentation
- Open an issue in this repository

## ☕ Support This Project

If this project has helped you, consider supporting its development:

[![Buy Me A Coffee](https://img.shields.io/badge/Buy%20Me%20A%20Coffee-☕-yellow.svg)](https://buymeacoffee.com/christancho)

Your support helps maintain and improve this n8n Docker Compose stack!
