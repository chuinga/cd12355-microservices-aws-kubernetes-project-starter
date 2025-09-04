# Coworking Space Analytics Microservice

A cloud-native analytics application deployed on AWS EKS that provides business intelligence data for coworking space usage patterns.

## Architecture

This project implements a microservices architecture using Flask API, PostgreSQL database, AWS EKS for orchestration, ECR for container registry, CodeBuild for CI/CD, and CloudWatch for monitoring.

## Deployment Process

The CI/CD pipeline uses AWS CodeBuild connected to GitHub that automatically builds Docker images on code changes and pushes them to ECR with semantic versioning using build numbers. The Kubernetes deployment consists of ConfigMap for environment variables, Secret for sensitive data, Deployment with health checks and resource limits, and LoadBalancer Service for external access. New releases are deployed by updating the image tag in `coworking-deployment.yaml` and applying changes with `kubectl apply`, which triggers a rolling update with zero downtime.

## Resource Configuration

The application uses conservative resource limits of 100m-200m CPU and 128Mi-256Mi memory to ensure efficient utilization while preventing resource starvation.

## Stand-Out Suggestions

### Optimal AWS Instance Type
For this analytics application, **t3.medium** instances would be optimal for production workloads. The t3.medium provides 2 vCPUs and 4GB RAM, offering better performance headroom than t3.small while maintaining cost efficiency through burstable performance. The application's lightweight nature and intermittent usage patterns align well with T3's credit-based CPU model, allowing cost savings during low-traffic periods while providing burst capacity for peak analytics queries.

### Cost Optimization Strategies
**Spot Instances**: Implement mixed instance types using Spot instances for non-critical workloads, potentially reducing compute costs by 70-90%. **Horizontal Pod Autoscaler**: Configure HPA to scale pods based on CPU/memory utilization, ensuring resources match actual demand. **Cluster Autoscaler**: Enable automatic node scaling to minimize idle capacity during off-peak hours, combined with scheduled scaling for predictable traffic patterns.

## API Endpoints

- `GET /health_check`: Application health status
- `GET /api/reports/daily_usage`: Daily check-in analytics  
- `GET /api/reports/user_visits`: User visit patterns