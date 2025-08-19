# GitOps Fundamentals: Verification Notes

This file documents minor items to verify after the initial extraction.

## Placeholders to confirm

- Repository URLs in ArgoCD resources:
  - `https://github.com/your-org/infrastructure-gitops.git` in:
    - `workflow-patterns/argocd-app-infrastructure-dev.yaml`
    - `workflow-patterns/applicationset-application-infrastructure.yaml`
  Replace with your actual GitOps repository URLs.

- API group and composition names in Composite Resource example:
  - `apiVersion: database.example.org/v1alpha1`
  - `compositionRef.name: azure-postgresql-server`
  Validate these match your platform API and available Composition names.

## Relative link validations

- Docs references use paths like `../../exercises/...` from files under `docs/gitops-fundamentals/`. From the doc location, `../../` correctly resolves to repository root. Links manually checked for:
  - `argocd/argocd-cm-crossplane-health.yaml`
  - `rbac/argocd-crossplane-clusterrole.yaml`
  - `crossplane-integration/xpostgresqlinstance-dev.yaml`
  - `workflow-patterns/argocd-app-infrastructure-dev.yaml`
  - `workflow-patterns/applicationset-application-infrastructure.yaml`

## Scope intentionally deferred (next pass)

- Extracting GitHub Actions workflow YAML blocks from `03-workflow-patterns.md` into exercises.
- Extracting monitoring/alerting rules and other non-Kubernetes resources into exercises.

## Notes

- All added doc reference blocks follow the required command prefix format: `user@host idp-tutorial> $`.
- No original tutorial content was modified beyond adding reference blocks immediately after the relevant YAML snippets.
