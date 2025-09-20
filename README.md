# Spring PetClinic CI/CD Deployment Pipeline

This repository contains a comprehensive GitHub Actions workflow for automatically building, testing, and deploying the Spring PetClinic application with a complete monitoring stack.

## Overview

The pipeline automatically:
- Downloads and builds the Spring PetClinic application
- Runs comprehensive tests
- Creates Docker containers for the application and monitoring services
- Deploys the entire stack with health monitoring
- Provides detailed deployment reporting

## Architecture

### Applications Deployed
- **Spring PetClinic**: Main web application (Pet clinic management system)
- **MySQL 8.0**: Database backend
- **Prometheus**: Metrics collection and monitoring
- **Grafana**: Dashboard and visualization
- **Node Exporter**: System metrics collection

### Ports and Access
- **PetClinic Application**: http://localhost:9500
- **Grafana Dashboard**: http://localhost:3000 (admin/admin123)
- **Prometheus Metrics**: http://localhost:9090
- **Node Exporter**: http://localhost:9100
- **MySQL Database**: localhost:3306

## Prerequisites

### For Self-Hosted Runner (Recommended)
- Ubuntu/WSL2 environment
- Docker and Docker Compose installed
- GitHub Actions self-hosted runner configured
- Minimum 4GB RAM, 20GB disk space

### For GitHub Cloud Runner
- Repository with GitHub Actions enabled
- Note: Containers will not persist after workflow completion

## Workflow Structure

### Job 1: `build-and-test`
- **Environment**: Ubuntu with MySQL service
- **Purpose**: Build and test the application
- **Steps**:
  - Sets up Java 17 and Maven
  - Downloads Spring PetClinic source if not present
  - Compiles and tests the application
  - Packages JAR files
  - Uploads build artifacts

### Job 2: `docker-build-and-deploy`
- **Environment**: Ubuntu with Docker
- **Purpose**: Containerize and deploy the complete stack
- **Steps**:
  - Downloads PetClinic source for Docker build
  - Creates optimized multi-stage Dockerfile
  - Generates Docker Compose configuration
  - Creates Prometheus monitoring configuration
  - Builds and deploys all services
  - Performs comprehensive health checks
  - Generates deployment reports

### Job 3: `cleanup`
- **Environment**: Ubuntu
- **Purpose**: Workflow summary and cleanup reporting
- **Steps**:
  - Provides execution summary
  - Reports deployment status
  - Gives access instructions

## Configuration Files Generated

### Dockerfile (Multi-stage)
```dockerfile
# Build stage with Maven
FROM maven:3.8.7-eclipse-temurin-17 AS build
# ... build steps

# Runtime stage with OpenJDK
FROM openjdk:17-jdk-slim
# ... runtime configuration
```

### Docker Compose Services
- MySQL with health checks and persistent volumes
- PetClinic app with MySQL integration
- Prometheus with custom configuration
- Grafana with admin credentials
- Node Exporter for system metrics
- Custom network for service communication

### Monitoring Configuration
- Prometheus scrape configurations for all services
- Alertmanager basic setup (email notifications)
- Grafana data source provisioning

## Triggering the Workflow

### Automatic Triggers
- Push to `main` or `develop` branches
- Pull requests to `main` branch

### Manual Trigger
```bash
# Via GitHub UI: Actions tab -> Run workflow
# Via API:
curl -X POST \
  -H "Authorization: token YOUR_TOKEN" \
  -H "Accept: application/vnd.github.v3+json" \
  https://api.github.com/repos/YOUR_USERNAME/YOUR_REPO/actions/workflows/deploy.yml/dispatches \
  -d '{"ref":"main"}'
```

## Post-Deployment

### Accessing Services
After successful deployment on self-hosted runner:

```bash
# Check running containers
docker ps

# View application logs
docker logs petclinic-app

# Check all service logs
docker compose logs

# Stop all services
docker compose down

# Remove with volumes (complete cleanup)
docker compose down --volumes
```

### Health Check URLs
- Application Health: http://localhost:9500/actuator/health
- Prometheus Metrics: http://localhost:9500/actuator/prometheus
- Prometheus Targets: http://localhost:9090/targets
- Grafana Health: http://localhost:3000/api/health

## Troubleshooting

### Common Issues

#### Containers Not Persisting
- **Issue**: No containers after workflow completion
- **Cause**: Using GitHub cloud runners (ephemeral environment)
- **Solution**: Use self-hosted runner on WSL/Ubuntu

#### Port Conflicts
- **Issue**: Services fail to start
- **Check**: `netstat -tulpn | grep -E ':(3000|3306|9090|9100|9500)'`
- **Solution**: Stop conflicting services or change ports in docker-compose.yml

#### Build Failures
- **Issue**: Docker build fails with missing files
- **Cause**: PetClinic source not downloaded properly
- **Solution**: Check "Setup PetClinic Project for Docker" step logs

#### Memory Issues
- **Issue**: Services crash or don't start
- **Check**: `docker system df` and `free -h`
- **Solution**: Increase system memory or reduce services

### Debugging Commands

```bash
# Check Docker system status
docker system info
docker system df

# Inspect specific containers
docker inspect petclinic-app
docker logs mysql

# Check network connectivity
docker network ls
docker network inspect [network_name]

# Monitor resource usage
docker stats

# Clean up system resources
docker system prune -f
docker volume prune -f
```

## Customization

### Modifying Services
Edit the workflow's "Create Docker Compose Configuration" section to:
- Change port mappings
- Add new services
- Modify environment variables
- Update resource limits

### Adding Monitoring
The pipeline includes Prometheus configuration. To add custom metrics:
1. Modify the Spring Boot application to expose additional metrics
2. Update prometheus.yml configuration in the workflow
3. Create custom Grafana dashboards

### Environment-Specific Configuration
For different environments, modify:
- Database credentials and connection strings
- Resource allocation (CPU/Memory limits)
- Network configuration
- Volume mount points

## Security Considerations

### Default Credentials
- **Grafana**: admin/admin123 (change immediately)
- **MySQL**: root/petclinic (change for production)

### Production Readiness
This workflow is designed for development/testing. For production:
- Use proper secrets management
- Implement proper authentication
- Use external databases
- Set up proper SSL/TLS
- Configure backup strategies
- Implement proper logging and monitoring

## Workflow Artifacts

The pipeline generates several artifacts:
- **Build Artifacts**: JAR files (30-day retention)
- **Deployment Logs**: Container logs (7-day retention)
- **Health Check Reports**: Service status summaries

## Support

### Logs and Debugging
All workflow logs are available in GitHub Actions interface. For container-specific issues, check:
- Deployment logs artifact
- Individual container logs via `docker logs`
- Compose service logs via `docker compose logs`

### Performance Monitoring
Access Grafana dashboard at http://localhost:3000 for:
- Application performance metrics
- System resource usage
- Database performance
- Custom monitoring dashboards

This pipeline provides a complete CI/CD solution for the Spring PetClinic application with comprehensive monitoring and logging capabilities.