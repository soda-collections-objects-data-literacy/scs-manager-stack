# SCS Manager Development Environment

A complete Docker Compose setup for running the SODa SCS Manager with all required services including Drupal, MariaDB, Redis, and Varnish.

## Overview

This repository provides a production-ready Docker Compose configuration for the SCS Manager application. The stack includes:

- **Drupal** with NGINX and PHP-FPM (SCS Manager image)
- **MariaDB** database with optimized configuration
- **Redis** for session storage and caching
- **Varnish** HTTP cache accelerator

## Architecture

```
Internet → Varnish (Port 80) → Drupal/NGINX (Port 8080) → PHP-FPM (Unix Socket)
                                    ↓
                              MariaDB (Port 3306)
                                    ↓
                              Redis (Port 6379)
```

- **Varnish** acts as the reverse proxy and HTTP cache, receiving all external traffic
- **Drupal** runs NGINX + PHP-FPM in a single container
- **MariaDB** stores all Drupal data
- **Redis** handles PHP sessions (database 2) and can be used for Drupal caching

## Prerequisites

- Docker Engine 20.10 or later
- Docker Compose 2.0 or later
- Git (for cloning the repository)

## Quick Start

1. Clone the repository:

```bash
git clone <repository-url>
cd scs-development
```

2. Create a `.env` file based on `example-env`:

```bash
cp example-env .env
```

3. Edit `.env` and set your passwords and configuration:

```env
# Database configuration
DB_DRIVER=mysql
DB_HOST=database
DB_NAME=drupal
DB_PASSWORD=your_secure_database_password
DB_PORT=3306
DB_ROOT_PASSWORD=your_secure_root_password
DB_USER=drupal

# Drupal site configuration
DRUPAL_PASSWORD=your_secure_admin_password
DRUPAL_SITE_NAME=SCS Manager
DRUPAL_USER=admin
```

4. Start all services:

```bash
docker compose up -d
```

5. Access the site:

- **Frontend (via Varnish):** `http://localhost:80` (or port specified by `VARNISH_PORT`)
- **Direct Drupal access:** `http://localhost:8080` (or port specified by `SCS_MANAGER_PORT`)

## Services

### Drupal (scs-manager--scs-manager)

The main application container running Drupal 11 with NGINX and PHP-FPM.

**Image:** `ghcr.io/soda-collections-objects-data-literacy/scs-manager-image-${MODE:-production}:${SCS_MANAGER_IMAGE_VERSION:-latest}`

**Ports:**
- `SCS_MANAGER_PORT` (default: 8080) → Container port 80

**Health Check:**
- Endpoint: `/health`
- Interval: 15s
- Timeout: 10s
- Retries: 10
- Start period: 120s (allows time for Drupal installation)

**Volumes:**
- `scs-manager--drupal-sites`: Persistent Drupal sites directory

### Database (scs-manager--database)

MariaDB database server with optimized configuration for high performance.

**Image:** `mariadb:${DATABASE_IMAGE_VERSION:-latest}`

**Configuration:**
- Optimized InnoDB settings (8GB buffer pool, 16 instances)
- UTF8MB4 character set
- Slow query logging enabled
- Max connections: 1000

**Volumes:**
- `scs-manager--database-data`: Persistent database storage

### Redis (scs-manager--redis)

Redis server for session storage and caching.

**Image:** `redis:${REDIS_IMAGE_VERSION:-8-alpine}`

**Configuration:**
- Max memory: 512MB
- Eviction policy: allkeys-lru
- Persistence: AOF (append-only file) enabled
- Resource limits: 768MB memory, 1 CPU

**Health Check:**
- Command: `redis-cli ping`
- Interval: 5s

**Volumes:**
- `scs-manager--redis-data`: Persistent Redis data

### Varnish (scs-manager--varnish)

HTTP reverse proxy and cache accelerator.

**Image:** `ghcr.io/soda-collections-objects-data-literacy/scs-varnish:${VARNISH_IMAGE_VERSION:-1.0.0}`

**Ports:**
- `VARNISH_PORT` (default: 80) → Container port 80

**Configuration:**
- Backend: Drupal container (`drupal:80`)
- Cache size: 256MB
- Memory lock: 512MB

**Dependencies:**
- Waits for Drupal health check to pass before starting

**Health Check:**
- Command: `varnishstat -1`
- Interval: 30s

## Environment Variables

### Required Variables

Create a `.env` file in the project root with the following variables:

| Variable | Description | Example |
|----------|-------------|---------|
| `DB_DRIVER` | Database driver | `mysql` |
| `DB_HOST` | Database hostname (service name) | `database` |
| `DB_NAME` | Database name | `drupal` |
| `DB_PASSWORD` | Database password | `your_secure_password` |
| `DB_PORT` | Database port | `3306` |
| `DB_ROOT_PASSWORD` | MariaDB root password | `your_secure_root_password` |
| `DB_USER` | Database username | `drupal` |
| `DRUPAL_PASSWORD` | Drupal admin password | `your_secure_password` |
| `DRUPAL_SITE_NAME` | Site display name | `SCS Manager` |
| `DRUPAL_USER` | Drupal admin username | `admin` |

### Optional Variables

| Variable | Description | Default |
|----------|-------------|---------|
| `MODE` | Build mode (`production` or `development`) | `production` |
| `SCS_MANAGER_IMAGE_VERSION` | SCS Manager image version | `latest` |
| `SCS_MANAGER_PORT` | Direct Drupal access port | `8080` |
| `VARNISH_PORT` | Varnish public port | `80` |
| `VARNISH_IMAGE_VERSION` | Varnish image version | `1.0.0` |
| `DATABASE_IMAGE_VERSION` | MariaDB image version | `latest` |
| `REDIS_IMAGE_VERSION` | Redis image version | `8-alpine` |

## Common Commands

### Start Services

```bash
docker compose up -d
```

### Stop Services

```bash
docker compose stop
```

### Stop and Remove Containers

```bash
docker compose down
```

### Stop and Remove Containers + Volumes

**WARNING:** This will delete all data including the database!

```bash
docker compose down -v
```

### View Logs

```bash
# All services
docker compose logs -f

# Specific service
docker compose logs -f drupal
docker compose logs -f database
docker compose logs -f redis
docker compose logs -f varnish
```

### Execute Drush Commands

```bash
docker compose exec drupal drush cr
docker compose exec drupal drush status
docker compose exec drupal drush user:login
```

### Execute Composer Commands

```bash
docker compose exec drupal composer require drupal/module_name
docker compose exec drupal composer update
```

### Check Service Status

```bash
docker compose ps
```

### Check Health Status

```bash
# Drupal health check
curl http://localhost:8080/health

# Varnish stats
docker compose exec varnish varnishstat -1

# Redis ping
docker compose exec redis redis-cli ping

# Database connection
docker compose exec database mysql -u root -p -e "SHOW DATABASES;"
```

## Volumes

The following named volumes are created:

- `scs-manager--database-data`: MariaDB data directory
- `scs-manager--drupal-sites`: Drupal sites directory (settings, files)
- `scs-manager--redis-data`: Redis persistent data

To backup volumes:

```bash
docker run --rm -v scs-manager--database-data:/data -v $(pwd):/backup alpine tar czf /backup/database-backup.tar.gz /data
```

## First Run

On the first startup:

1. Drupal will wait for MariaDB to be ready
2. If `settings.php` doesn't exist, Drupal will be installed automatically
3. All required modules will be enabled
4. Configuration will be imported from the image
5. Default content will be imported
6. Health check endpoint will become available
7. Varnish will start once Drupal is healthy

This process takes approximately 2-5 minutes. Subsequent startups are instant.

## Development Mode

To use the development image with Xdebug:

1. Set `MODE=development` in your `.env` file
2. Rebuild/restart:

```bash
docker compose down
docker compose up -d
```

Xdebug configuration:
- Port: `9003`
- IDE key: `scs`
- Trigger value: `scs`

## Troubleshooting

### Services Not Starting

Check logs:

```bash
docker compose logs
```

### Drupal Health Check Failing

Check if Drupal is fully installed:

```bash
docker compose exec drupal curl http://localhost/health
```

### Varnish Not Starting

Varnish waits for Drupal to be healthy. Check Drupal health:

```bash
docker compose ps drupal
```

### Redis Connection Issues

Test Redis connectivity:

```bash
docker compose exec drupal php -r "\$r = new Redis(); \$r->connect('redis', 6379); echo 'OK';"
```

### Database Connection Issues

Verify database is running:

```bash
docker compose ps database
docker compose logs database
```

### Permission Issues

Reset permissions:

```bash
docker compose exec drupal chown -R www-data:www-data /opt/drupal
docker compose exec drupal chmod -R 775 /opt/drupal
```

## Production Considerations

- Use strong passwords in `.env` file
- Set appropriate resource limits for your infrastructure
- Configure backup strategy for volumes
- Use environment-specific image tags instead of `latest`
- Configure proper firewall rules
- Set up monitoring and logging
- Review MariaDB configuration for your workload
- Configure Redis persistence strategy
- Review Varnish cache configuration

## License

See LICENSE.md for license information.

## Support

For issues and questions, please open an issue on the GitHub repository.
