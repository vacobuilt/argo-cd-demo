# ArgoCD Slack Notifications Setup

This guide will help you set up Slack notifications for your ArgoCD demo, making it even more impressive by showing real-time alerts in Slack.

## Prerequisites

1. Slack workspace admin access
2. ArgoCD running in your cluster
3. kubectl access to your cluster

## Step 1: Create Slack App

1. **Go to Slack API**: https://api.slack.com/apps
2. **Create New App** → "From scratch"
3. **App Name**: "ArgoCD Notifications"
4. **Workspace**: Select your workspace

## Step 2: Configure Bot Permissions

1. **OAuth & Permissions** → **Scopes** → **Bot Token Scopes**
2. Add these scopes:
   - `chat:write` - Send messages
   - `chat:write.public` - Send messages to channels the app isn't in
   - `files:write` - Upload files (optional)

## Step 3: Install App to Workspace

1. **Install App** → **Install to Workspace**
2. **Copy Bot User OAuth Token** (starts with `xoxb-`)
3. **Save this token** - you'll need it shortly

## Step 4: Create Slack Channel

1. Create a channel: `#ami-platform-alerts`
2. **Invite the bot** to the channel: `/invite @ArgoCD Notifications`

## Step 5: Apply ArgoCD Configuration

1. **Update the secret with your Slack token**:
```bash
# Edit the secret file
vim argo-demo/notifications/argocd-notifications-secret.yaml

# Replace "xoxb-your-slack-bot-token-here" with your actual token
```

2. **Apply the configurations**:
```bash
# Apply the secret
kubectl apply -f argo-demo/notifications/argocd-notifications-secret.yaml

# Apply the notification configuration
kubectl apply -f argo-demo/notifications/argocd-notifications-configmap.yaml
```

3. **Restart ArgoCD notifications controller**:
```bash
kubectl rollout restart deployment argocd-notifications-controller -n shared-platform-tools
```

## Step 6: Add Notification Annotations to Applications

The applications need to be annotated to enable notifications. Run these commands:

```bash
# Add notifications to frontend app
kubectl patch application uhes-60-frontend -n shared-platform-tools --type='merge' -p='{
  "metadata": {
    "annotations": {
      "notifications.argoproj.io/subscribe.on-sync-succeeded.slack": "ami-platform-alerts",
      "notifications.argoproj.io/subscribe.on-sync-failed.slack": "ami-platform-alerts",
      "notifications.argoproj.io/subscribe.on-health-degraded.slack": "ami-platform-alerts"
    }
  }
}'

# Add notifications to backend app
kubectl patch application http-proxy-data-ingestion-service -n shared-platform-tools --type='merge' -p='{
  "metadata": {
    "annotations": {
      "notifications.argoproj.io/subscribe.on-sync-succeeded.slack": "ami-platform-alerts",
      "notifications.argoproj.io/subscribe.on-sync-failed.slack": "ami-platform-alerts",
      "notifications.argoproj.io/subscribe.on-health-degraded.slack": "ami-platform-alerts"
    }
  }
}'
```

## Step 7: Test Notifications

1. **Trigger a sync** by making a small change to your Git repository
2. **Watch Slack** for notifications
3. **Test failure scenario** using the erroneous image already in your demo

## Demo Integration

### For Your Client Demo

**Show Real-Time Integration:**
1. **Open Slack** on a second screen/window
2. **Make a change** during demo (e.g., fix the erroneous image)
3. **Show live notification** appearing in Slack
4. **Highlight enterprise features**: Real-time alerting, team collaboration

**Demo Script Addition:**
```
"Now let me show you our enterprise monitoring capabilities. 
As I fix this deployment issue, you'll see real-time notifications 
appearing in our Slack channel, keeping the entire team informed."
```

## Notification Types You'll Get

### ✅ Successful Deployments
- Green notifications when apps sync successfully
- Include repository info, revision, health status

### ⚠️ Health Issues  
- Yellow notifications when apps become degraded
- Links directly to ArgoCD for investigation

### ❌ Sync Failures
- Red notifications when deployments fail
- Immediate alerts for quick response

### 🔄 Sync Success
- Confirmations when syncs complete successfully
- Perfect for tracking deployment progress

## Troubleshooting

**If notifications aren't working:**

1. **Check bot permissions** in Slack
2. **Verify bot is in channel**: `/invite @ArgoCD Notifications`
3. **Check ArgoCD logs**:
```bash
kubectl logs -l app.kubernetes.io/name=argocd-notifications-controller -n shared-platform-tools
```
4. **Verify secret**:
```bash
kubectl get secret argocd-notifications-secret -n shared-platform-tools -o yaml
```

## Advanced Features

### Custom Channels Per App
```bash
# Different channels for different apps
kubectl patch application uhes-60-frontend -n shared-platform-tools --type='merge' -p='{
  "metadata": {
    "annotations": {
      "notifications.argoproj.io/subscribe.on-sync-failed.slack": "frontend-alerts"
    }
  }
}'
```

### Notification Filters
```bash
# Only critical notifications
kubectl patch application http-proxy-data-ingestion-service -n shared-platform-tools --type='merge' -p='{
  "metadata": {
    "annotations": {
      "notifications.argoproj.io/subscribe.on-health-degraded.slack": "critical-alerts"
    }
  }
}'
```

## Business Value for Demo

**Enterprise Features Demonstrated:**
- **Real-time monitoring** across all deployments
- **Team collaboration** through Slack integration  
- **Immediate alerting** for quick issue resolution
- **Audit trail** of all deployment activities
- **Reduced MTTR** through instant notifications

This integration shows how ArgoCD fits into existing communication workflows and provides enterprise-grade monitoring capabilities.
