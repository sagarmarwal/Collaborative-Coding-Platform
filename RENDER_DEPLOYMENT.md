# Render Deployment Guide

## Overview

This guide explains how to deploy the Collaborative Coding Platform to [Render](https://render.com) using the `render.yml` configuration file.

## Prerequisites

1. A Render account (free tier available)
2. GitHub repository with this project
3. Docker Hub or GitHub Container Registry access (for Piston API image)

## Deployment Steps

### 1. Connect Your Repository to Render

1. Go to [Render Dashboard](https://dashboard.render.com)
2. Click "New +" and select "Web Service"
3. Connect your GitHub repository
4. Render will automatically detect `render.yml`

### 2. Configure Services

The `render.yml` file defines three services:

#### Web Service: `collaborative-coding-platform`
- **Type**: Docker
- **Dockerfile**: Uses the `Dockerfile` in your repository
- **Port**: 5000
- **Auto-Deploy**: Enabled (deploys on git push)

#### Web Service: `piston-api`
- **Type**: Docker Image
- **Image**: `ghcr.io/engineer-man/piston:latest`
- **Port**: 2000
- **Purpose**: Handles code execution

#### Redis Service: `coding-platform-redis`
- **Type**: Redis (managed by Render)
- **Plan**: Free tier
- **Memory Policy**: `allkeys-lru` (evict least recently used keys)

### 3. Environment Variables

The main app will receive these environment variables:

- `NODE_ENV` - Set to `production`
- `PORT` - Set to `5000`
- `PISTON_API_URL` - Auto-populated with the Piston API service URL
- `PISTON_API_PORT` - Set to `2000`

### 4. Using Redis in Your Code

To use Redis, update your `server.js`:

```javascript
const redis = require('redis');

const redisUrl = process.env.REDIS_URL || 'redis://coding-platform-redis:6379';
const redisClient = redis.createClient({ url: redisUrl });

redisClient.on('error', (err) => console.log('Redis Client Error', err));
redisClient.connect();
```

### 5. Using Piston API

Your app can communicate with Piston API at:

```javascript
const pistonUrl = `http://${process.env.PISTON_API_URL || process.env.PISTON_API_PORT}`;
// Or use the environment variable directly
const pistonUrl = `http://piston-api:${process.env.PISTON_API_PORT || 2000}`;
```

## Important Notes

### Free Tier Limitations

- Services may shut down after 15 minutes of inactivity
- Limited resources (0.5 CPU, 512 MB RAM)
- No custom domains on free tier (you get Render's subdomain)

### Upgrading to Paid

To ensure 24/7 uptime:

1. Go to your service settings on Render
2. Upgrade the plan from "Free" to "Starter" or higher
3. Services will remain running continuously

## Monitoring

### View Logs

```bash
# In Render Dashboard
1. Navigate to your service
2. Click "Logs" tab
3. View real-time logs
```

### Health Checks

Both services include health check endpoints:

- **App**: `/` (default GET request)
- **Piston API**: `/api/v1/health`

Render automatically monitors these endpoints.

## Redeploy

### Manual Redeploy

1. Go to Render Dashboard
2. Select the service
3. Click "Rerun latest deployment"

### Automatic Redeploy

- Enabled by default (`autoDeploy: true`)
- Automatically redeploysr when you push to your main branch

## Troubleshooting

### Services Not Starting

1. Check the logs in Render Dashboard
2. Verify environment variables are set correctly
3. Ensure Docker image builds successfully

### Connection Issues Between Services

- Services communicate using service names (e.g., `piston-api`)
- Ensure both services are on the same deployment

### Redis Connection Errors

- Verify `REDIS_URL` is in your code
- Check that your Redis service is running in Render Dashboard

## Updating render.yml

After modifying `render.yml`:

1. Push changes to GitHub
2. Render will automatically redeploy with new configuration
3. Monitor logs during redeployment

## Cost Estimation

- **Free Tier**: $0/month (with limitations)
- **Starter Plan** (recommended): $7/month per service
- **Standard Plan**: $12/month per service

Total for 3 services on Starter Plan: ~$21/month

## Additional Resources

- [Render Documentation](https://render.com/docs)
- [render.yaml Specification](https://render.com/docs/infrastructure-as-code)
- [Docker on Render](https://render.com/docs/docker)
