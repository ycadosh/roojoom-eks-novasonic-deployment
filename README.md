# Pipecat Voice AI Agent - AWS Cloud Deployment

A production-ready containerized deployment of the Pipecat Voice AI Agent on AWS, featuring both WebRTC and Twilio phone integration with AWS Nova Sonic for natural voice conversations. Supports both ECS and EKS deployment options.

## What is This Project?

This project provides a complete AWS cloud deployment solution for Pipecat, an open-source framework for building voice AI agents. It includes:

- **WebRTC Voice Chat**: Browser-based voice conversations using Daily.co
- **Phone Integration**: Twilio-powered phone calls with bidirectional audio
- **AWS Nova Sonic**: Advanced text-to-speech and speech-to-text using AWS Bedrock
- **Production Infrastructure**: Complete AWS deployment with CDK (ECS or EKS)
- **Monitoring & Logging**: CloudWatch integration with health checks
- **Security**: AWS Secrets Manager, IAM roles, and container security best practices
- **Kubernetes Support**: EKS deployment with Fargate and Network Load Balancer

## Server Applications

### `server.py` - Production WebRTC Server

The main production server focused on WebRTC voice conversations:

**Features:**

- Full-featured FastAPI server with Daily.co WebRTC integration
- Production-ready with comprehensive logging and health checks
- Resource monitoring and automatic cleanup of stale processes
- RTVI client support for advanced voice applications
- Browser-based voice chat interface
- Comprehensive error handling and graceful shutdown

**Use Cases:**

- Web-based voice AI applications
- RTVI client integrations
- Browser voice chat interfaces
- Production WebRTC deployments

### `server_clean.py` - Phone Service Server

Specialized server for Twilio phone integration with Nova Sonic:

**Features:**

- Twilio WebSocket integration for phone calls
- AWS Nova Sonic for natural voice synthesis and recognition
- Bidirectional audio streaming over phone networks
- Enhanced session management for phone calls
- Weather function example with tool calling
- Optimized for 8kHz phone audio quality

**Use Cases:**

- Phone-based AI assistants
- Call center automation
- Voice IVR systems
- Twilio phone integrations

## Project Structure

```
pipecat-cloud-deployment/
├── 📁 aws/                    # AWS configuration files
│   ├── policies/              # IAM policies for ECS tasks
│   └── task-definitions/      # ECS task definitions
├── 📁 config/                 # Application configuration
│   └── deployment_config.py   # Environment and resource settings
├── 📁 docker/                 # Docker configuration
│   ├── Dockerfile             # Main WebRTC application container
│   ├── Dockerfile.phone       # Phone service container
│   ├── docker-compose.test.yml
│   └── scripts/               # Docker build scripts
├── 📁 infrastructure/         # AWS CDK infrastructure code (ECS)
│   ├── lib/                   # CDK stack definitions
│   ├── deploy.sh              # Infrastructure deployment script
│   └── DEPLOYMENT_GUIDE.md    # Detailed deployment guide
├── 📁 infrastructure/-eks/    # AWS CDK infrastructure code (EKS)
│   ├── lib/                   # EKS CDK stack definitions
│   └── pipecat-eks-stack.ts   # EKS cluster configuration
├── 📁 scripts/                # Deployment and utility scripts
│   ├── build-and-push.sh      # Build and push to ECR
│   ├── deploy-service.sh      # Deploy ECS service
│   └── setup-secrets.py       # Configure AWS Secrets Manager
├── 📁 tests/                  # Test suite
├── 📁 docs/                   # Documentation
├── bot.py                     # Core Pipecat bot logic
├── server.py                  # Production WebRTC server
├── server_clean.py            # Phone service server
├── pipecat-phone-service.yaml # Kubernetes deployment manifest
└── requirements.txt           # Python dependencies
```

## Requirements and Dependencies

The `requirements.txt` includes all necessary dependencies:

```
python-dotenv          # Environment variable management
fastapi[all]           # Web framework with all extras
uvicorn               # ASGI server
websockets==13.1      # WebSocket support (pinned for compatibility)
pipecat-ai[daily,aws-nova-sonic,silero]==0.0.79  # Core Pipecat with integrations
loguru                # Advanced logging
boto3                 # AWS SDK
botocore              # AWS core library
aioboto3              # Async AWS SDK
psutil                # System monitoring
twilio                # Twilio integration for phone service
```

## Environment Variables

### Required for All Services

- `DAILY_API_KEY` - Daily.co API key for WebRTC
- `AWS_REGION` - AWS region (e.g., eu-north-1)

### Required for Phone Service (server_clean.py)

- `TWILIO_ACCOUNT_SID` - Twilio account identifier
- `TWILIO_AUTH_TOKEN` - Twilio authentication token
- `TWILIO_PHONE_NUMBER` - Your Twilio phone number
- `AWS_ACCESS_KEY_ID` - AWS credentials for Nova Sonic
- `AWS_SECRET_ACCESS_KEY` - AWS secret key

### Optional Configuration

- `ENVIRONMENT` - deployment environment (development/production)
- `LOG_LEVEL` - logging level (DEBUG/INFO/WARNING/ERROR)
- `HOST` - server host (default: 0.0.0.0)
- `FAST_API_PORT` - server port (default: 7860)
- `MAX_BOTS_PER_ROOM` - maximum bots per room (default: 1)
- `MAX_CONCURRENT_ROOMS` - maximum concurrent rooms (default: 10)

## Complete Deployment Guide

Choose between ECS (container orchestration) or EKS (Kubernetes) deployment based on your needs:

- **ECS**: Simpler setup, managed container orchestration
- **EKS**: Kubernetes-native, more flexibility and control

### 1. Prerequisites

Ensure you have the following installed and configured:

```bash
# Required tools
aws --version          # AWS CLI v2
node --version         # Node.js 18+
npm --version          # npm 8+
docker --version       # Docker 20+
python3 --version      # Python 3.10+

# Install CDK globally
npm install -g aws-cdk

# For EKS deployment, also install kubectl
kubectl version --client

# Configure AWS credentials
aws configure
```

### 2. Deploy AWS Infrastructure (CDK)

Choose your deployment platform:

#### Option A: ECS Deployment (Recommended for simplicity)

```bash
# Navigate to infrastructure directory
cd infrastructure

# Install CDK dependencies
npm install

# Deploy infrastructure (creates ECS cluster, ECR, ALB, etc.)
./deploy.sh --environment test --region us-east-1

# For production with custom VPC
./deploy.sh --environment prod --custom-vpc --region us-east-1
```

**ECS Resources Created:**
- ECS Cluster with Fargate capacity
- ECR repositories for container images
- Application Load Balancer with health checks
- VPC, subnets, and security groups
- IAM roles with least-privilege permissions
- CloudWatch log groups
- Secrets Manager integration

#### Option B: EKS Deployment (Kubernetes-native)

```bash
# Navigate to EKS infrastructure directory
cd infrastructure/-eks

# Install CDK dependencies
npm install

# Deploy EKS infrastructure
cdk deploy PipecatEksStack --parameters environment=test --parameters useDefaultVpc=false

# Configure kubectl to connect to your cluster
aws eks update-kubeconfig --region us-east-1 --name pipecat-eks-cluster-test
```

**EKS Setup Details:**
- **Region**: us-east-1
- **Node Type**: t3.medium instances
- **Kubernetes Version**: v1.31 (CDK managed)
- **Container Runtime**: containerd://1.7.27
- **OS**: Amazon Linux 2023.7.20250512
- **Architecture**: amd64 (x86_64)
- **Network Flow**: NLB (TLS Listener) → Target Group → k8s Pod

**EKS Resources Created:**
- EKS Cluster with Fargate profiles
- ECR repositories (reused from ECS if exists)
- Network Load Balancer with TLS termination
- VPC with public/private subnets
- IAM service accounts with IRSA
- AWS Load Balancer Controller
- CloudWatch log groups for EKS
- Kubernetes namespace: `pipecat`

### 3. Configure Secrets

Set up AWS Secrets Manager with your API keys:

```bash
# Return to project root
cd ..

# Create .env file with your credentials
cat > .env << EOF
DAILY_API_KEY=your_daily_api_key_here
AWS_ACCESS_KEY_ID=your_aws_access_key
AWS_SECRET_ACCESS_KEY=your_aws_secret_key
AWS_REGION=eu-north-1
TWILIO_ACCOUNT_SID=your_twilio_account_sid
TWILIO_AUTH_TOKEN=your_twilio_auth_token
TWILIO_PHONE_NUMBER=+1234567890
EOF

# Upload secrets to AWS Secrets Manager
python3 scripts/setup-secrets.py

# Verify secrets were created
aws secretsmanager list-secrets --query 'SecretList[?contains(Name, `pipecat`)].Name'
```

### 4. Build and Deploy Containers

#### For ECS Deployment

##### Deploy WebRTC Service (server.py)

```bash
# Build and push main WebRTC container to ECR
./scripts/build-and-push.sh -e test -t latest

# Deploy to ECS
./scripts/deploy-service.sh -e test -t latest
```

##### Deploy Phone Service (server_clean.py)

```bash
# Build and push phone service container
./scripts/build-phone-service.sh -e test -t latest

# Deploy phone service to ECS
./scripts/deployment/deploy-phone-service.sh -e test -t latest
```

#### For EKS Deployment

##### Build and Push Container Images

```bash
# Build and push both services to ECR
./scripts/build-and-push.sh -e test -t latest
./scripts/build-phone-service.sh -e test -t latest
```

##### Deploy to Kubernetes

```bash
# Create the pipecat namespace
kubectl create namespace pipecat

# Apply the Kubernetes deployment manifest
kubectl apply -f pipecat-phone-service.yaml -n pipecat

# Verify deployment
kubectl get pods -n pipecat
kubectl get services -n pipecat
kubectl get ingress -n pipecat
```

##### Monitor EKS Deployment

```bash
# Check pod status
kubectl describe pods -n pipecat

# View application logs
kubectl logs -f deployment/pipecat-phone-service -n pipecat

# Check service endpoints
kubectl get endpoints -n pipecat
```

### 5. Verify Deployment

#### For ECS Deployment

```bash
# Get application URLs
aws cloudformation describe-stacks \
  --stack-name PipecatEcsStack-test \
  --query 'Stacks[0].Outputs[?OutputKey==`LoadBalancerDnsName`].OutputValue' \
  --output text

# Check ECS service status
aws ecs describe-services \
  --cluster pipecat-cluster-test \
  --services pipecat-service-test

# View application logs
aws logs tail /ecs/pipecat-voice-agent-test/application --follow
```

#### For EKS Deployment

```bash
# Get cluster information
aws eks describe-cluster --name pipecat-eks-cluster-test --region us-east-1

# Check pod status and get external IP
kubectl get pods,services,ingress -n pipecat -o wide

# Get Network Load Balancer endpoint
kubectl get service pipecat-phone-service -n pipecat -o jsonpath='{.status.loadBalancer.ingress[0].hostname}'

# View application logs
kubectl logs -f deployment/pipecat-phone-service -n pipecat

# Check cluster nodes
kubectl get nodes -o wide
```

### 6. Test Your Deployment

**WebRTC Service:**

- Visit `http://your-alb-dns-name/` for browser-based voice chat
- Use `/health` endpoint for health checks
- Use `/connect` for RTVI client integration

**Phone Service:**

- Configure your Twilio webhook to point to your ALB
- Call your Twilio phone number to test voice AI
- Monitor active calls at `/active-calls` endpoint

## Health Check Endpoints

Both services provide comprehensive health monitoring:

### `/health`

Validates:

- Daily API helper initialization
- Required environment variables
- Active bot process tracking
- Resource usage (memory/CPU)
- AWS service connectivity

### `/ready`

Readiness check for deployment strategies:

- Service initialization status
- Basic functionality verification

## Container Features

### Security

- Non-root user execution (UID/GID 1001)
- Minimal base image (python:3.12-slim)
- Secrets injected via AWS Secrets Manager
- Security groups with least-privilege access

### Monitoring

- Structured logging for CloudWatch
- Health checks with 30s intervals
- Resource monitoring and cleanup
- Performance metrics and tracing

### Scalability

- Horizontal scaling via ECS service
- Auto-scaling based on CPU/memory
- Load balancing across multiple instances
- Graceful shutdown handling

## Cleanup and Maintenance

### Local Development Cleanup

```bash
# Clean up Docker resources
./scripts/cleanup.sh local --docker --cache
```

### AWS Environment Cleanup

#### ECS Environment

```bash
# Clean up test environment
./scripts/cleanup.sh aws -e test

# Destroy CDK infrastructure
cd infrastructure
cdk destroy PipecatEcsStack-test
```

#### EKS Environment

```bash
# Delete Kubernetes resources
kubectl delete namespace pipecat

# Destroy EKS CDK infrastructure
cd infrastructure/-eks
cdk destroy PipecatEksStack

# Clean up kubectl config (optional)
kubectl config delete-context arn:aws:eks:us-east-1:ACCOUNT:cluster/pipecat-eks-cluster-test
```

### Complete Cleanup

```bash
# Clean everything (local + AWS)
./scripts/cleanup.sh all -e test --local-all --aws-all
```

## Monitoring and Troubleshooting

### View Logs

#### ECS Logs

```bash
# Application logs
aws logs tail /ecs/pipecat-voice-agent-test/application --follow

# ECS service events
aws ecs describe-services --cluster pipecat-cluster-test --services pipecat-service-test
```

#### EKS Logs

```bash
# Application logs
kubectl logs -f deployment/pipecat-phone-service -n pipecat

# Pod events
kubectl describe pods -n pipecat

# Cluster logs (if enabled)
aws logs tail /aws/eks/pipecat-eks-cluster-test/cluster --follow
```

### Common Issues

#### General Issues
- **Container fails to start**: Check secrets configuration and IAM permissions
- **Health checks failing**: Verify environment variables and AWS connectivity
- **Phone calls not working**: Check Twilio webhook configuration and Nova Sonic setup
- **WebRTC not connecting**: Verify Daily.co API key and network connectivity

#### EKS-Specific Issues
- **Pods stuck in Pending**: Check Fargate profile selectors and node capacity
- **LoadBalancer not getting external IP**: Verify AWS Load Balancer Controller installation
- **IRSA permissions**: Ensure service account annotations match IAM role ARN
- **Network connectivity**: Check security groups and NACLs for t3.medium nodes

## Documentation

For detailed information, see:

- [Infrastructure Deployment Guide](infrastructure/DEPLOYMENT_GUIDE.md)
- [Deployment Scripts README](scripts/README.md)
- [Cleanup Guide](docs/CLEANUP_GUIDE.md)
- [Troubleshooting Guide](docs/TROUBLESHOOTING_GUIDE.md)

## Architecture

This deployment provides a production-ready, scalable voice AI platform with multiple deployment options:

### ECS Architecture
- **High Availability**: Multi-AZ deployment with Application Load Balancer
- **Security**: AWS IAM, Secrets Manager, and VPC isolation
- **Monitoring**: CloudWatch logs, metrics, and health checks
- **Scalability**: Auto-scaling ECS services with Fargate
- **Cost Optimization**: Fargate spot instances and efficient resource usage

### EKS Architecture
- **Kubernetes-Native**: Full Kubernetes API and ecosystem support
- **Network Load Balancer**: TLS termination with target group routing to pods
- **Fargate Serverless**: No EC2 instance management required
- **IRSA Security**: IAM Roles for Service Accounts with least-privilege access
- **Container Runtime**: containerd on Amazon Linux 2023 (t3.medium nodes)
- **Monitoring**: CloudWatch Container Insights and Kubernetes-native logging

### Network Flow (EKS)
```
Internet → NLB (TLS Listener) → Target Group → Kubernetes Pod (Fargate)
```

Perfect for building voice AI applications, phone assistants, and WebRTC-based conversational interfaces at scale with your choice of container orchestration platform.
