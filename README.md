# Coworking Space Analytics Microservice

A cloud-native analytics application deployed on AWS EKS that provides business intelligence data for coworking space usage patterns.

## Architecture Overview

This project implements a microservices architecture using:
- **Flask API**: Python-based analytics service
- **PostgreSQL**: Database for user and token data
- **AWS EKS**: Kubernetes orchestration platform
- **AWS ECR**: Container registry for Docker images
- **AWS CodeBuild**: CI/CD pipeline for automated builds
- **AWS CloudWatch**: Monitoring and logging

## Deployment Process

### Infrastructure Components

1. **EKS Cluster**: Managed Kubernetes cluster (`my-cluster`) running on t3.small nodes
2. **ECR Repository**: `coworking-analytics` for storing Docker images
3. **CodeBuild Project**: Automated CI/CD pipeline triggered by GitHub commits
4. **CloudWatch Container Insights**: Monitoring and log aggregation

### Build Pipeline

The CI/CD process uses AWS CodeBuild with the following workflow:

1. **Source**: GitHub repository triggers build on code changes
2. **Pre-build**: Authenticate with ECR using `aws ecr get-login-password`
3. **Build**: Docker image built using `buildspec.yml` configuration
4. **Tag**: Images tagged with `$CODEBUILD_BUILD_NUMBER` for versioning
5. **Push**: Image pushed to ECR repository automatically

### Kubernetes Deployment

The application uses a multi-component Kubernetes deployment:

- **ConfigMap**: Stores non-sensitive environment variables (DB_HOST, DB_USERNAME, DB_PORT, DB_NAME)
- **Secret**: Securely stores sensitive data (DB_PASSWORD) with base64 encoding
- **Deployment**: Manages application pods with health checks and resource limits
- **Service**: LoadBalancer exposes the application externally on port 5153

### Health Monitoring

- **Liveness Probe**: `/health_check` endpoint ensures container restart on failure
- **Readiness Probe**: `/readiness_check` endpoint controls traffic routing
- **CloudWatch Logs**: Centralized logging for debugging and monitoring
- **Container Insights**: Resource utilization and performance metrics

## Releasing New Builds

To deploy application changes:

1. **Code Changes**: Push commits to the main branch of the GitHub repository
2. **Automatic Build**: CodeBuild automatically triggers and builds new Docker image
3. **Image Versioning**: New image tagged with incremental build number
4. **Manual Deployment**: Update `coworking-deployment.yaml` with new image tag
5. **Apply Changes**: Run `kubectl apply -f coworking-deployment.yaml`
6. **Rolling Update**: Kubernetes performs zero-downtime deployment

For automated deployments, consider implementing GitOps with ArgoCD or Flux to automatically sync Kubernetes manifests with the repository.

## Resource Configuration

The application is configured with conservative resource limits:
- **CPU Request**: 100m (0.1 CPU cores)
- **CPU Limit**: 200m (0.2 CPU cores)  
- **Memory Request**: 128Mi
- **Memory Limit**: 256Mi

These limits ensure efficient resource utilization while preventing resource starvation in multi-tenant environments.

## Stand-Out Suggestions

### Optimal AWS Instance Type

For this analytics application, **t3.medium** instances would be optimal for production workloads. The t3.medium provides 2 vCPUs and 4GB RAM, offering better performance headroom than t3.small while maintaining cost efficiency through burstable performance. The application's lightweight nature and intermittent usage patterns align well with T3's credit-based CPU model, allowing cost savings during low-traffic periods while providing burst capacity for peak analytics queries.

### Cost Optimization Strategies

**Spot Instances**: Implement mixed instance types using Spot instances for non-critical workloads, potentially reducing compute costs by 70-90%. **Horizontal Pod Autoscaler**: Configure HPA to scale pods based on CPU/memory utilization, ensuring resources match actual demand. **Cluster Autoscaler**: Enable automatic node scaling to minimize idle capacity during off-peak hours, combined with scheduled scaling for predictable traffic patterns.

## API Endpoints

- `GET /health_check`: Application health status
- `GET /readiness_check`: Readiness for traffic
- `GET /api/reports/daily_usage`: Daily check-in analytics
- `GET /api/reports/user_visits`: User visit patterns

## Prerequisites

- AWS CLI configured with appropriate permissions
- kubectl configured for EKS cluster access
- Docker for local development and testing

## Local Development

```bash
# Set environment variables
export DB_USERNAME=myuser
export DB_PASSWORD=mypassword
export DB_HOST=localhost
export DB_PORT=5432
export DB_NAME=mydatabase

# Install dependencies
pip install -r analytics/requirements.txt

# Run application
python analytics/app.py
```

## Monitoring

Access CloudWatch Container Insights for:
- Cluster performance metrics
- Pod resource utilization
- Application logs and error tracking
- Custom metrics and alerting