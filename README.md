# Docker Compose PostgreSQL Documentation

## Overview

This document explains how to use Docker Compose to run a PostgreSQL database in a Docker container.

## Configuration File

```yaml
version: '3.8'

services:
  postgres:
    image: postgres:15-alpine
    container_name: postgres_db
    environment:
      POSTGRES_DB: mydatabase
      POSTGRES_USER: myuser
      POSTGRES_PASSWORD: mypassword
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data
    networks:
      - postgres_network

volumes:
  postgres_data:

networks:
  postgres_network:
    driver: bridge
```

## Configuration Explanation

### Services

- **postgres**: PostgreSQL database service
  - `image: postgres:15-alpine` - Uses PostgreSQL version 15 with lightweight Alpine Linux base image
  - `container_name: postgres_db` - Name of the container to be created
  - `environment` - Environment variables for database configuration:
    - `POSTGRES_DB` - Default database name
    - `POSTGRES_USER` - Username for database connection
    - `POSTGRES_PASSWORD` - Password for database user

### Ports

- `"5432:5432"` - Maps container port 5432 to host port 5432

### Volumes

- `postgres_data` - Volume for database data persistence
- Data is stored at `/var/lib/postgresql/data` within the container

### Networks

- `postgres_network` - Bridge network for container connectivity

## Usage Guide

### 1. Prerequisites

Ensure Docker and Docker Compose are installed on your system.

### 2. Starting PostgreSQL

```bash
# Start services in detached mode (background)
docker-compose up -d

# Or start with logs visible
docker-compose up
```

### 3. Checking Status

```bash
# Check container status
docker-compose ps

# View logs
docker-compose logs postgres

# View real-time logs
docker-compose logs -f postgres
```

### 4. Stopping Services

```bash
# Stop services
docker-compose down

# Stop and remove volumes (WARNING: data will be lost)
docker-compose down -v
```

### 5. Other Management Commands

```bash
# Restart services
docker-compose restart

# Stop services without removing containers
docker-compose stop

# Start previously stopped services
docker-compose start
```

## Database Connection

### Using psql inside Container

```bash
# Access shell inside container
docker exec -it postgres_db bash

# Then connect to database
psql -U myuser -d mydatabase
```

### Using psql from Host

```bash
# Ensure psql is installed on host system
psql -h localhost -p 5432 -U myuser -d mydatabase
```

### Using Database Client Application

- **Host**: `localhost`
- **Port**: `5432`
- **Database**: `mydatabase`
- **Username**: `myuser`
- **Password**: `mypassword`

## Backup and Restore

### Backup Database

```bash
# Backup using docker exec
docker exec -t postgres_db pg_dump -U myuser mydatabase > backup.sql
```

### Restore Database

```bash
# Restore from backup file
cat backup.sql | docker exec -i postgres_db psql -U myuser -d mydatabase
```

## Available Environment Variables

| Variable                  | Description             | Default                  |
| ------------------------- | ----------------------- | ------------------------ |
| POSTGRES_DB               | Default database        | postgres                 |
| POSTGRES_USER             | Superuser username      | postgres                 |
| POSTGRES_PASSWORD         | Superuser password      | -                        |
| POSTGRES_HOST_AUTH_METHOD | Authentication method   | md5                      |
| PGDATA                    | Data directory location | /var/lib/postgresql/data |

## Security

### Best Practices:

1. **Change default passwords** - Always use strong passwords
2. **Use custom networks** - For better isolation
3. **Regular backups** - Perform regular data backups
4. **Environment files** - Use environment files to store credentials

### Using Environment File

Create `.env` file:

```env
POSTGRES_DB=mydatabase
POSTGRES_USER=myuser
POSTGRES_PASSWORD=strongpassword123
```

Update docker-compose.yml:

```yaml
environment:
  POSTGRES_DB: ${POSTGRES_DB}
  POSTGRES_USER: ${POSTGRES_USER}
  POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
```

## Troubleshooting

### Common Issues:

1. **Port already in use**

   ```bash
   # Check processes using port 5432
   netstat -tulpn | grep 5432

   # Or change port in docker-compose.yml
   ports:
     - "5433:5432"
   ```
2. **Permission error on volume**

   ```bash
   # Remove volume and recreate
   docker-compose down -v
   docker-compose up -d
   ```
3. **Container won't start**

   ```bash
   # Check logs for error details
   docker-compose logs postgres
   ```

## Tips

1. **Development vs Production**:

   - Development: Use port mapping for easy access
   - Production: Consider not exposing ports and use internal networking
2. **Performance**:

   - Adjust resource limits based on requirements
   - Use optimal volumes for I/O performance
3. **Monitoring**:

   - Use `docker stats` to monitor resource usage
   - Set up appropriate logging for audit purposes

This documentation covers basic Docker Compose usage with PostgreSQL. Adjust configurations based on your specific application requirements.

## Advanced Configuration

### Resource Limits

```yaml
services:
  postgres:
    # ... other configs
    deploy:
      resources:
        limits:
          memory: 1G
          cpus: '1.0'
        reservations:
          memory: 512M
          cpus: '0.5'
```

### Health Checks

```yaml
services:
  postgres:
    # ... other configs
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U myuser -d mydatabase"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s
```

### Multiple Databases

```yaml
environment:
  POSTGRES_MULTIPLE_DATABASES: "mydatabase,anotherdb,testdb"
```

This documentation provides comprehensive guidance for running PostgreSQL with Docker Compose in various environments.
