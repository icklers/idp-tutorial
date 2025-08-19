# GitOps Target Path Mapping

This repository uses the following target paths when copying exercise files into GitOps-managed locations. Adjust if you mirror these files into a separate GitOps repo.

- ArgoCD bootstrap, health checks, RBAC → `gitops-bootstrap/argocd/`
- Platform core (XRDs, Compositions, functions) → `platform-core/`
- Environment instances (Composite Resources) → `exercises/crossplane-advanced-patterns-01/environments/{env}/infrastructure/`
- GitOps workflow examples (Applications, ApplicationSets) → copy into your infra repo under `applications/` or `applicationsets/` as desired

Examples
```bash
# Health checks ConfigMap
cp exercises/gitops-fundamentals/argocd/argocd-cm-crossplane-health.yaml gitops-bootstrap/argocd/

# RBAC ClusterRole
cp exercises/gitops-fundamentals/rbac/argocd-crossplane-clusterrole.yaml platform-core/

# Composite Resource for dev
cp exercises/gitops-fundamentals/crossplane-integration/xpostgresqlinstance-dev.yaml exercises/crossplane-advanced-patterns-01/environments/dev/infrastructure/

git add .
git commit -m "chore(gitops): add initial health/rbac and sample CR"
git push
```

After push, watch ArgoCD:
```bash
argocd app wait platform-core --timeout 300
argocd app wait infrastructure-dev --timeout 300
```
