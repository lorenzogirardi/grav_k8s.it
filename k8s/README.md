# Grav on izanagi (k3s)

## Architecture

```
Browser (LAN) → grav-192.168.1.15.dyn.k8s.it → Traefik → grav-svc:80 → Grav pod
                                                               PVC: grav-config (2Gi)

Argo CronWorkflow (every 6h):
  linuxserver/grav pod (mounts same PVC)
  → php bin/plugin static-site-generator export
  → git push --force → lorenzogirardi/grav_k8s.it:gh-pages

GitHub Pages → lorenzogirardi.github.io/grav_k8s.it (or custom domain)
```

## First-time deploy

```bash
export KUBECONFIG=~/.kube/izanagi.conf

# 1. Namespace + storage + RBAC
kubectl apply -f k8s/namespace.yaml
kubectl apply -f k8s/pvc.yaml
kubectl apply -f k8s/rbac.yaml

# 2. GitHub PAT secret (copy example, fill token, apply)
cp k8s/secret.example.yaml k8s/secret.yaml
# edit k8s/secret.yaml — set GITHUB_TOKEN
kubectl apply -f k8s/secret.yaml

# 3. Deploy Grav
kubectl apply -f k8s/deployment.yaml
kubectl apply -f k8s/service.yaml
kubectl apply -f k8s/ingressroute.yaml

# 4. Install SSG plugin via Grav admin or CLI
kubectl exec -n grav deploy/grav -- php /config/www/bin/gpm install static-site-generator -n

# 5. Deploy Argo CronWorkflow
kubectl apply -f k8s/argo-ssg-workflow.yaml
```

## Manual publish trigger

```bash
# Via Argo CLI
argo submit -n grav --from cronwf/grav-ssg-publish --watch

# Or Argo UI: grav namespace → CronWorkflows → grav-ssg-publish → Submit
```

## GitHub Pages setup

1. Go to repo Settings → Pages
2. Source: Deploy from branch → `gh-pages` → `/ (root)`
3. Custom domain: optional

## SSG plugin notes

- Output dir: `/config/www/static-export` (configured in `user/config/plugins/static-site-generator.yaml`)
- Argo workflow expects files there — if plugin changes the path, update `argo-ssg-workflow.yaml` `OUTPUT_DIR`
- Plugin must be installed inside the PVC (step 4 above)
