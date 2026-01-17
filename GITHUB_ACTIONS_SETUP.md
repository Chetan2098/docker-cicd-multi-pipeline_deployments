# GitHub Actions Setup Guide

## Quick Setup Instructions

### Step 1: Add GitHub Secrets
Go to your repository → **Settings** → **Secrets and variables** → **Actions**

Click "New repository secret" and add the following secrets:

| Secret Name | Value | Example |
|------------|-------|---------|
| `DOCKER_USERNAME` | Your Docker Hub username | `your-docker-id` |
| `DOCKER_PASSWORD` | Your Docker Hub personal access token | `dckr_pat_xxxxx` |
| `KUBECONFIG` | Base64 encoded kubeconfig file | (see below) |

### Step 2: Encode Your Kubeconfig

**On Windows (PowerShell):**
```powershell
# Copy your kubeconfig file path and run this:
$content = Get-Content "C:\ProgramData\Jenkins\.kube\config" -Raw
$encoded = [Convert]::ToBase64String([System.Text.Encoding]::UTF8.GetBytes($content))
Set-Clipboard -Value $encoded
# Now paste the value in GitHub Secrets
```

**On Linux/Mac:**
```bash
cat ~/.kube/config | base64 | pbcopy
# Now paste the value in GitHub Secrets
```

### Step 3: Verify Workflow File Location
Ensure the workflow file is at the correct path:
```
.github/workflow/deployment.yaml
```

If you need `workflows` (plural) instead, move it to:
```
.github/workflows/deployment.yaml
```

### Step 4: Test the Workflow

**Trigger manually:**
1. Go to repository → **Actions** tab
2. Click **Build and Deploy to Kubernetes**
3. Click **Run workflow** → Select environment → **Run workflow**

**Or trigger by pushing:**
```bash
git add .
git commit -m "Trigger GitHub Actions workflow"
git push origin main
```

## Workflow Jobs Explained

### 1. **Build Job**
- Checks out code
- Sets up Docker Buildx for efficient builds
- Logs in to Docker Hub
- Extracts metadata (tags, labels)
- Builds and pushes Docker image
- **Output**: Image tag and digest

### 2. **Deploy Job**
- Checks out code
- Sets up kubectl
- Configures kubeconfig from secrets
- Tests Kubernetes connectivity
- Gets current deployments
- Updates deployment with new image
- Waits for rollout to complete
- Verifies deployment status
- Displays access information

### 3. **Test Job**
- Sets up Python 3.10
- Installs dependencies from requirements.txt
- Runs pytest tests (optional)

### 4. **Notify Job**
- Sends deployment status
- Creates deployment summary
- Uploads artifact with summary

## Workflow Triggers

The workflow runs on:
- ✅ **Push** to main, master, or develop branches
- ✅ **Pull requests** to main or master branches
- ✅ **Manual trigger** via workflow_dispatch (with environment selection)

## Important Notes

### Docker Hub Credentials
- Use a **Personal Access Token** instead of password
- Create token at: https://hub.docker.com/settings/security
- Permissions needed: Read/Write

### Kubeconfig File
- Must be properly formatted
- Ensure cluster URL is accessible from GitHub Actions
- Test locally first: `kubectl cluster-info`

### Deployment Requirements
- Kubernetes deployment named: `cicd-demo`
- Container name in deployment: `cicd-demo`
- Service named: `cicd-demo-service`
- Ensure these match your actual Kubernetes configuration

## Monitoring Workflow Execution

### View Workflow Runs
1. Go to **Actions** tab in repository
2. Click on workflow run to see details
3. Click on job name to see step outputs

### Check Logs
Each step has logs you can expand to see:
- Build output
- Docker push results
- Kubectl commands
- Deployment status

## Troubleshooting

### Issue: "Workflow not triggering"
**Solution**: 
- Verify workflow file is in `.github/workflow/` directory
- Check branch name matches trigger conditions
- Ensure file has proper YAML syntax

### Issue: "Docker push fails"
**Solution**:
- Verify DOCKER_USERNAME and DOCKER_PASSWORD are correct
- Use PAT instead of password
- Check Docker Hub account permissions

### Issue: "Kubernetes deployment fails"
**Solution**:
- Verify KUBECONFIG is properly base64 encoded
- Test kubeconfig locally: `kubectl get nodes`
- Ensure cluster is accessible from GitHub
- Check if deployment/service exist: `kubectl get deployment cicd-demo`

### Issue: "Permission denied" on kubeconfig
**Solution**:
- Verify kubeconfig file is readable
- Check permissions: `ls -la ~/.kube/config`
- Ensure KUBECONFIG secret is base64 encoded

## Advanced Customization

### Change Docker Registry
Edit environment variable:
```yaml
env:
  REGISTRY: gcr.io  # For Google Container Registry
  # or
  REGISTRY: ghcr.io  # For GitHub Container Registry
```

### Add Slack Notifications
Add this step to notify job:
```yaml
- name: Slack Notification
  uses: slackapi/slack-github-action@v1.24.0
  with:
    webhook-url: ${{ secrets.SLACK_WEBHOOK }}
    payload: |
      {
        "text": "Deployment ${{ job.status }}"
      }
```

### Add Email Notifications
```yaml
- name: Send Email
  uses: dawidd6/action-send-mail@v3
  with:
    server_address: ${{ secrets.EMAIL_SERVER }}
    server_port: 465
    username: ${{ secrets.EMAIL_USERNAME }}
    password: ${{ secrets.EMAIL_PASSWORD }}
    subject: "Deployment ${{ job.status }}"
    to: ${{ secrets.EMAIL_TO }}
    from: GitHub Actions
```

### Matrix Strategy (Multiple Python Versions)
```yaml
strategy:
  matrix:
    python-version: ['3.8', '3.9', '3.10', '3.11']
```

## Security Best Practices

✅ **Do**:
- Use Personal Access Tokens instead of passwords
- Rotate tokens regularly
- Use environments for production deployments
- Add branch protection rules
- Use status checks to require passing workflows

❌ **Don't**:
- Commit secrets to repository
- Use hardcoded credentials
- Share kubeconfig files
- Use admin-level permissions unnecessarily

## Useful GitHub Actions Marketplace Actions

- `actions/checkout@v4` - Clone repository
- `docker/setup-buildx-action@v3` - Setup Docker build
- `docker/login-action@v3` - Login to registry
- `docker/build-push-action@v5` - Build and push images
- `docker/metadata-action@v5` - Generate metadata
- `azure/setup-kubectl@v3` - Setup kubectl
- `actions/setup-python@v4` - Setup Python
- `actions/upload-artifact@v3` - Upload artifacts

## Next Steps

1. ✅ Add all required secrets
2. ✅ Verify kubeconfig file
3. ✅ Push changes to repository
4. ✅ Monitor Actions tab for execution
5. ✅ Verify deployment in Kubernetes
6. ✅ Set up notifications (optional)

## Support Resources

- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [Docker Build Push Action](https://github.com/docker/build-push-action)
- [Kubernetes GitHub Actions](https://github.com/Azure/setup-kubectl)
- [GitHub Marketplace](https://github.com/marketplace?type=actions)
