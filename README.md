# IoT_cgorin - Application Repository

This repository contains the Kubernetes manifests for the Inception-of-Things Part 3 project.

## Repository Structure

```
IoT_cgorin/
├── .git/                    # Git version control
├── README.md               # This file
├── apps/
│   └── playground.yaml     # Application deployment and service (watched by Argo CD)
└── argocd/
    └── application.yaml    # Argo CD Application manifest (GitOps configuration)
```

## Important Files

### `apps/playground.yaml`
This is the **source of truth** for your application deployment. It contains:
- **Deployment**: Defines the playground application pod
  - Image: `wil42/playground:v1` (or `v2`)
  - Port: 80 (exposed on 8888 externally via port-forward)
  - Replica: 1
- **Service**: Exposes the deployment
  - ClusterIP service listening on port 80
  - Selector: `app: playground`
  - Deployed to namespace: `dev`

### `argocd/application.yaml`
This manifest defines how Argo CD should manage the deployment:
- Watches: `https://github.com/K1000e/IoT_cgorin.git`
- Source path: `apps/`
- Auto-sync: Enabled with self-healing and auto-pruning
- Target namespace: `dev`

## How to Update the Application Version

### Method 1: Direct Edit

```bash
# Edit the file locally
nano apps/playground.yaml

# Change the image line:
# From: image: wil42/playground:v1
# To:   image: wil42/playground:v2

# Commit and push
git add apps/playground.yaml
git commit -m "Update playground to v2"
git push
```

### Method 2: Using sed (Script)

```bash
sed -i 's/wil42\/playground:v1/wil42\/playground:v2/g' apps/playground.yaml
git add apps/playground.yaml
git commit -m "Update playground to v2"
git push
```

### Method 3: Version Switching Alias (Optional)

Add to your `.bashrc`:
```bash
# Git shortcut
alias gup='git add -A && git commit -m'
alias gpu='git push'

# Playground version switching
alias pg-v1="sed -i 's/wil42\/playground:v2/wil42\/playground:v1/g' apps/playground.yaml && gup 'Switch to v1' && gpu"
alias pg-v2="sed -i 's/wil42\/playground:v1/wil42\/playground:v2/g' apps/playground.yaml && gup 'Switch to v2' && gpu"
```

Then simply:
```bash
pg-v1  # Switch to version 1
pg-v2  # Switch to version 2
```

## Verification After Update

### 1. Check Git Log
```bash
git log --oneline
# Should show your recent commits
```

### 2. Check GitHub
Visit: `https://github.com/K1000e/IoT_cgorin.git`
- Verify your changes are pushed
- Check the raw content of `apps/playground.yaml`

### 3. Verify Deployment in Cluster

```bash
# Check Argo CD status
argocd app get playground
# Look for "SYNC STATUS" - should be "Synced"

# Check running pods
kubectl get pods -n dev -o wide

# Check deployment
kubectl describe deployment playground -n dev
# Look for the image version in "Container Image"

# Test the application
curl http://localhost:8888/
# v1: {"status":"ok", "message": "v1"}
# v2: {"status":"ok", "message": "v2"}
```

### 4. Check Argo CD UI
- Open https://localhost:8080
- Look at the "playground" application
- Check the "SYNC STATUS" - should show "Synced"
- Check pod details and image version

## Troubleshooting

### Changes Not Syncing
```bash
# Check Argo CD detection (waits up to 3 minutes)
argocd app get playground

# Manually trigger sync
argocd app sync playground

# Force sync if needed
argocd app sync playground --force

# Monitor sync progress
watch -n 1 'argocd app get playground | grep SYNC'
```

### Image Not Updating in Pod
```bash
# Check if image is available
k3d image list

# Check current pod
kubectl get pods -n dev -o wide
kubectl describe pod <pod-name> -n dev | grep -A 5 "Container ID"

# Check deployment status
kubectl get deployment -n dev
kubectl describe rs -n dev  # ReplicaSet (shows image being used)

# Check logs
kubectl logs -n dev -l app=playground
```

### Git Push Issues
```bash
# Verify remote is set correctly
git remote -v
# Should show: origin  git@github.com:K1000e/IoT_cgorin.git

# Verify you have SSH or HTTPS access
git config user.email
git config user.name

# Test connection
ssh -T git@github.com  # For SSH
# or
curl -I https://github.com  # For HTTPS
```

## Understanding the Sync Cycle

```
1. You push changes to GitHub
         │
         ▼
2. Argo CD detects the change (within 3 minutes by default)
         │
         ▼
3. Argo CD compares Git state with cluster state
         │
         ├─ If different → Sync needed
         │
         ▼
4. Argo CD applies the new manifests
         │
         ▼
5. Kubernetes creates new pod with new image
         │
         ▼
6. Old pod is terminated
         │
         ▼
7. Argo CD reports "Synced" ✅
```

## Best Practices

1. **Always commit before pushing**: `git add` → `git commit` → `git push`
2. **Use meaningful commit messages**: Helps track what changed and why
3. **Avoid manual kubectl edits**: All changes should go through Git
4. **Check Argo CD UI regularly**: Verify synchronization status
5. **Test locally first** (if possible): Before pushing to production
6. **Keep images stable**: Use version tags (v1, v2) instead of `latest`

## For Your Defense

Prepare to demonstrate:
1. ✅ Show this repository on GitHub
2. ✅ Show the `apps/playground.yaml` file
3. ✅ Change the image version in Git
4. ✅ Show Argo CD detecting the change
5. ✅ Show the new version running in the cluster
6. ✅ Verify with `curl http://localhost:8888/`

## Reference Commands

```bash
# Quick status check
watch 'echo "=== Argo CD ===" && argocd app get playground | grep SYNC && echo "=== Pod ===" && kubectl get pods -n dev && echo "=== App ===" && curl http://localhost:8888/'

# Complete audit
argocd app get playground && \
kubectl get deployment -n dev && \
kubectl get pods -n dev -o wide && \
kubectl logs -n dev -l app=playground && \
curl http://localhost:8888/
```

## Links

- **GitHub**: https://github.com/K1000e/IoT_cgorin.git
- **Argo CD UI**: https://localhost:8080 (admin user)
- **App Access**: http://localhost:8888
- **Argo CD CLI**: `argocd` command
- **Kubectl**: `kubectl` or `k` (alias)
