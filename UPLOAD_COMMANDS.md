# Commands to Upload to New GitHub Repository

## Step 1: Remove current remote and add your new repository
```bash
# Remove current remote
git remote remove origin

# Add your new repository (replace with your actual repo URL)
git remote add origin https://github.com/YOUR_USERNAME/YOUR_REPO_NAME.git

# Verify the new remote
git remote -v
```

## Step 2: Push to new repository
```bash
# Push to main branch
git push -u origin main
```

## Step 3: Set up GitHub repository settings

### Repository Description:
```
Production-ready AWS deployment of Pipecat Voice AI Agent with WebRTC and phone integration. Supports both ECS and EKS deployment options with AWS Nova Sonic for natural voice conversations.
```

### Topics (add these in repository settings):
```
pipecat voice-ai aws ecs eks kubernetes webrtc twilio daily-co nova-sonic bedrock cdk infrastructure-as-code containerization fargate voice-assistant ai-agent speech-to-text text-to-speech phone-integration production-ready
```

### GitHub Actions Secrets (for CI/CD):
Go to Settings → Secrets and variables → Actions and add:
- `AWS_ACCESS_KEY_ID`: Your AWS access key
- `AWS_SECRET_ACCESS_KEY`: Your AWS secret key

## Step 4: Test the setup
```bash
# Run the setup script to verify everything works
./setup-project.sh
```

## Step 5: Create your first deployment
```bash
# For ECS deployment
cd infrastructure
./deploy.sh --environment test --region us-east-1

# For EKS deployment  
cd infrastructure/-eks
cdk deploy PipecatEksStack --parameters environment=test
```

Your repository is now ready for production use! 🎉