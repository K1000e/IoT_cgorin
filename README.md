# IoT_cgorin

This repository was created for the Inception of Things project at 42, part 3.
It contains only [apps/playground.yaml](apps/playground.yaml) and this README.
Its purpose is to demonstrate Argo CD with auto-sync, using GitHub as the source of truth.

## File Purpose

[apps/playground.yaml](apps/playground.yaml) contains the Kubernetes manifests for the playground:
- a `Deployment` to run the application
- a `Service` to expose it inside the cluster
- the `wil42/playground` image version to change when needed

## Key Points

- The file remains the source of truth for the application.
- Changes must be made in Git and then pushed.
- Argo CD automatically applies the new version afterward.

## Minimal Verification

```bash
git diff apps/playground.yaml
kubectl get pods -n dev
curl http://localhost:8888/
```
