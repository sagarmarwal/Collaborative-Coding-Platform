# Docker Deployment Guide

## Overview

This project includes Docker and Docker Compose configurations for easy deployment. The setup includes:

1. **Collaborative Coding Platform** - React frontend + Node.js/Express backend with Socket.io
2. **Piston API** - Code execution engine for running code snippets

## Prerequisites

- Docker (version 20.10+)
- Docker Compose (version 1.29+)

## Quick Start

### Build and Run with Docker Compose

```bash
# Start all services
docker-compose up -d

# View logs
docker-compose logs -f

# Stop services
docker-compose down
```

### Access the Application

- **Application**: http://localhost:5000
- **Piston API**: http://localhost:2000

## File Structure

- `Dockerfile` - Multi-stage build for optimized production image
- `docker-compose.yml` - Orchestrates both the app and Piston API services
- `.dockerignore` - Excludes unnecessary files from Docker build context

## Configuration

### Environment Variables

The application uses the following environment variables:

- `NODE_ENV` - Set to `production` in Docker (default in docker-compose)

### Volume Mounts

The Piston API uses volumes for:

- `./data/piston/packages:/piston/packages` - Persistent storage for language packages
- `/tmp:exec` - Temporary file system with execution permissions

## Docker Commands Reference

### Build the Image Manually

```bash
docker build -t collaborative-coding-platform .
```

### Run a Single Container

```bash
docker run -p 5000:5000 --name coding-app collaborative-coding-platform
```

### View Container Logs

```bash
docker-compose logs app
docker-compose logs piston_api
```

### Rebuild Services

```bash
docker-compose up -d --build
```

### Remove Everything

```bash
docker-compose down -v
```

## Performance Optimization

The Dockerfile uses a multi-stage build to:

1. Build the React application in the first stage
2. Create a minimal production image with only necessary files
3. Reduce the final image size significantly

## Networking

Both services communicate through a custom Docker network (`coding_network`), allowing them to communicate using service names:

- App can reach Piston API at: `http://piston_api:2000`

## Health Checks

Both services include health checks to ensure they're running properly:

- **App**: Checks HTTP 200 response on port 5000
- **Piston API**: Checks health endpoint at `/api/v1/health`

## Troubleshooting

### Port Already in Use

If ports 5000 or 2000 are already in use:

```bash
# Change ports in docker-compose.yml
# For example, change "5000:5000" to "3000:5000"
```

### Out of Disk Space

Clean up Docker resources:

```bash
docker system prune -a
```

### Permission Denied Errors

The Piston API container requires privileged mode for code execution. This is already configured in docker-compose.yml.

### Services Not Communicating

Verify the custom network is created:

```bash
docker network ls
```

Ensure both containers are on the `coding_network`.
