# GitHub Actions Setup Guide

This guide helps you configure GitHub Actions for automated deployment of the Pipecat Voice AI Agent on AWS (ECS or EKS).

## Required Repository Secrets

To use the GitHub Actions workflow, you need to configure the following secrets in your GitHub repository:

### 1. AWS Credentials

Go to your GitHub repository → Settings → Secrets and variables → Actions, and add:

- **`AWS_ACCESS_KEY_ID`**: Your AWS access key ID
- **`AWS_SECRET_ACCESS_KEY`**: Your AWS secret access key

### 2. Getting AWS Credentials

You have two options for AWS credentials:

#### Option A: Use Existing AWS User

If you already have an AWS user with appropriate permissions:

```bash
# Get your current credentials
aws configure list
```

Use the Access Key ID and Secret Access Key from your AWS configuration.

#### Option B: Create New IAM User for CI/CD

1. **Create IAM User**:

   ```bash
   aws iam create-user --user-name github-actions-pipecat
   ```

2. **Attach Required Policies**:

   ```bash
   # Attach ECS and ECR permissions
   aws iam attach-user-policy --user-name github-actions-pipecat --policy-arn arn:aws:iam::aws:policy/AmazonECS_FullAccess
   aws iam attach-user-policy --user-name github-actions-pipecat --policy-arn arn:aws:iam::aws:policy/AmazonEC2ContainerRegistryFullAccess
   aws iam attach-user-policy --user-name github-actions-pipecat --policy-arn arn:aws:iam::aws:policy/CloudFormationReadOnlyAccess
   ```

3. **Create Access Keys**:

   ```bash
   aws iam create-access-key --user-name github-actions-pipecat
   ```

4. **Note the Access Key ID and Secret Access Key** from the output.

### 3. Required IAM Permissions

The AWS user/role needs the following permissions:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "ecr:GetAuthorizationToken",
        "ecr:BatchCheckLayerAvailability",
        "ecr:GetDownloadUrlForLayer",
        "ecr:BatchGetImage",
        "ecr:PutImage",
        "ecr:InitiateLayerUpload",
        "ecr:UploadLayerPart",
        "ecr:CompleteLayerUpload",
        "ecr:DescribeRepositories",
        "ecr:DescribeImages"
      ],
      "Resource": "*"
    },
    {
      "Effect": "Allow",
      "Action": [
        "ecs:DescribeClusters",
        "ecs:DescribeServices",
        "ecs:DescribeTaskDefinition",
        "ecs:RegisterTaskDefinition",
        "ecs:UpdateService",
        "ecs:ListTasks"
      ],
      "Resource": "*"
    },
    {
      "Effect": "Allow",
      "Action": [
        "cloudformation:DescribeStacks",
        "cloudformation:DescribeStackResources"
      ],
      "Resource": "*"
    }
  ]
}
```

## Workflow Configuration

### Environment Setup

The workflow supports multiple environments and deployment platforms:

**Environment Mapping:**
- **`main` branch** → `prod` environment
- **`develop` branch** → `staging` environment
- **Pull requests** → `test` environment (build only, no deployment)
- **Manual dispatch** → User-specified environment

**Deployment Platforms:**
- **ECS**: Container orchestration with Fargate (default)
- **EKS**: Kubernetes with Fargate profiles

### Environment Protection Rules

For production deployments, consider setting up environment protection rules:

1. Go to your repository → Settings → Environments
2. Create environments: `prod`, `staging`, `test`
3. For `prod` environment, add protection rules:
   - Required reviewers
   - Wait timer
   - Deployment branches (restrict to `main`)

### Customizing the Workflow

You can customize the workflow by modifying `.github/workflows/deploy.yml`:

#### Change Default Region

```yaml
env:
  AWS_REGION: us-west-2 # Change from us-east-1
```

#### Modify Environment Mapping

```yaml
- name: Determine environment and settings
  id: env
  run: |
    if [ "${{ github.ref }}" = "refs/heads/main" ]; then
      ENVIRONMENT="production"  # Change from "prod"
    elif [ "${{ github.ref }}" = "refs/heads/develop" ]; then
      ENVIRONMENT="staging"
    # ... rest of the logic
```

#### Add Slack Notifications

Add this step to the `notify` job:

```yaml
- name: Slack Notification
  uses: 8398a7/action-slack@v3
  with:
    status: ${{ job.status }}
    channel: "#deployments"
    webhook_url: ${{ secrets.SLACK_WEBHOOK }}
  if: always()
```

## Testing the Workflow

### 1. Test with Pull Request

Create a pull request to test the build process:

```bash
git checkout -b test-deployment
git push origin test-deployment
# Create PR in GitHub UI
```

This will trigger the workflow to build the image but not deploy it.

### 2. Test Manual Deployment

Use the manual workflow dispatch:

1. Go to your repository → Actions
2. Select "Deploy Pipecat to AWS ECS"
3. Click "Run workflow"
4. Choose environment and options
5. Click "Run workflow"

### 3. Test Automatic Deployment

Push to main or develop branch:

```bash
git checkout main
git merge test-deployment
git push origin main
```

This will trigger automatic deployment to the production environment.

## Monitoring Deployments

### GitHub Actions

- View workflow runs in the Actions tab
- Check deployment summaries for application URLs
- Monitor job logs for detailed information

### AWS Console

- **ECS Console**: Monitor service health and task status
- **CloudWatch**: View application logs and metrics
- **ECR Console**: Check pushed container images

### Command Line

```bash
# Check service status
aws ecs describe-services --cluster pipecat-cluster-prod --services pipecat-service-prod

# View recent logs
aws logs tail /ecs/pipecat-voice-agent-prod --follow

# Get application URL
aws cloudformation describe-stacks --stack-name PipecatEcsStack-prod --query 'Stacks[0].Outputs[?OutputKey==`LoadBalancerDnsName`].OutputValue' --output text
```

## Troubleshooting

### Common Issues

1. **AWS Credentials Invalid**

   - Verify secrets are correctly set in GitHub
   - Check IAM user has required permissions
   - Ensure credentials haven't expired

2. **ECR Repository Not Found**

   - Ensure infrastructure is deployed first
   - Check repository name matches environment

3. **ECS Service Update Fails**

   - Verify ECS cluster and service exist
   - Check task definition is valid
   - Review CloudWatch logs for errors

4. **Build Fails**
   - Check Dockerfile syntax
   - Verify all required files are present
   - Review build logs for specific errors

### Getting Help

- Check the [Scripts README](../scripts/README.md) for detailed command usage
- Review the [Deployment Guide](../infrastructure/DEPLOYMENT_GUIDE.md) for infrastructure setup
- Check AWS CloudWatch logs for runtime errors
- Review GitHub Actions logs for build/deployment issues

## Security Best Practices

1. **Use Environment-Specific Secrets**: Don't share production secrets across environments
2. **Rotate Credentials Regularly**: Update AWS access keys periodically
3. **Limit IAM Permissions**: Use principle of least privilege
4. **Enable Branch Protection**: Require reviews for production deployments
5. **Monitor Access**: Review CloudTrail logs for API usage
6. **Use Environment Protection Rules**: Add approval requirements for sensitive environments

This setup provides a robust CI/CD pipeline for your Pipecat ECS deployment with proper security and monitoring in place.
