---
name: docker-operations
description: Use when managing Docker containers, images, or compose files for development workflow automation
---

# Docker Operations

## Overview

Docker operations through systematic CLI commands reduce development overhead.

## When to Use

**Trigger symptoms:**
- Container startup failures or hanging
- Image build errors or bloat
- docker-compose service conflicts
- Development environment setup needs
- Container resource issues

**Use cases:**
- Development environment automation
- Production image management
- Container debugging and monitoring
- Multi-service orchestration

## Core Pattern

### Container Management Flow
```bash
# Status check → targeted action → verification
docker ps           # Check current state
docker run/start/stop # Specific action
docker logs         # Verify success
```

### Image Management Flow
```bash
# Build → tag → push
docker build -t app:latest .    # Build with optimizations
docker tag app:latest repo/app:latest  # Version control
docker push repo/app:latest          # Deploy
```

## Quick Reference

| Situation | Command | Verification |
|-----------|---------|-------------|
| Start dev environment | `docker-compose up -d` | `docker-compose ps` |
| Stop environment | `docker-compose down` | `docker ps -q` |
| Rebuild image | `docker-compose build --no-cache` | `docker images | grep app` |
| View logs | `docker-compose logs -f service` | Error resolution |
| Clean up | `docker system prune -f` | `docker df` |

## Implementation

### Development Environment Setup
```bash
# Create docker-compose.override.yml for local development
cat > docker-compose.override.yml <<EOF
version: '3.8'
services:
  app:
    volumes:
      - .:/app  # Live reload
    environment:
      - NODE_ENV=development
      - DEBUG=*
EOF

docker-compose up -d  # Start with overrides
```

### Image Optimization
```bash
# Multi-stage builds for production
docker build -t app:production --target production .
docker history app:production  # Verify layer efficiency
```

## Common Mistakes

| Issue | Cause | Fix |
|--------|--------|-----|
| Container exits immediately | Missing dependencies or port conflicts | Check logs, verify ports, add health checks |
| Image too large | Development artifacts in production | Use .dockerignore, multi-stage builds |
| Slow builds | Rebuilding unchanged layers | Use build cache, proper layer ordering |
| Orphaned containers | Not cleaning up | Use `docker-compose down`, regular pruning |

## Real-World Impact

- **Environment setup**: 2 minutes vs 15 minutes manual
- **Container debugging**: 5 minutes vs 30 minutes trial-and-error  
- **Image management**: Consistent 200MB images vs 1GB+ bloat
- **Team collaboration**: Same compose files work everywhere