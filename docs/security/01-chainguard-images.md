# 01: Securing the Supply Chain with Chainguard Images

Our infrastructure is defined by code, but it runs on container images. The security of our control plane (Crossplane, ArgoCD, and all the Crossplane Providers) is paramount. A vulnerability in one of these components could compromise our entire system.

This is why we use **Chainguard Images**.

## What are Chainguard Images?

[Chainguard](https://www.chainguard.dev/) produces minimalist, hardened container images with a near-zero vulnerability count. They are built from source, signed, and continuously scanned.

Key features:

-   **Minimalism:** Images contain only the application and its direct dependencies. There is no shell, package manager, or other unnecessary tooling that could be exploited.
-   **SBOMs:** Every image comes with a Software Bill of Materials (SBOM), giving you a complete inventory of every component in the image.
-   **Signed:** Images are signed with Sigstore, allowing you to verify their integrity and provenance.

By using Chainguard images for our control plane components, we drastically reduce our attack surface.

## How We Use Chainguard Images

We have configured our Helm charts for Crossplane and ArgoCD to use Chainguard images instead of the default upstream images.

**Example: Overriding the Crossplane image in its Helm Chart**

When installing the Crossplane Helm chart, you can override the image registry and repository:

```bash
helm install crossplane crossplane-stable/crossplane \
  --namespace crossplane-system \
  --set image.repository=cgr.dev/chainguard/crossplane \
  --set image.tag=v1.15.0
```

We have pre-configured these overrides in our bootstrap process. When you installed Crossplane and ArgoCD in the "Getting Started" module, you were already using the hardened Chainguard versions.

## Verifying Image Signatures with Kyverno

It's not enough to just use secure images; we need to enforce that *only* secure, signed images can run in our cluster. We can use a policy engine like [Kyverno](https://kyverno.io/) to do this.

Here is an example Kyverno `ClusterPolicy` that enforces image signature verification:

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: check-image-signature
spec:
  validationFailureAction: Enforce
  rules:
    - name: check-chainguard-signature
      match:
        any:
        - resources:
            kinds:
              - Pod
      verifyImages:
      - imageReferences:
        - "cgr.dev/chainguard/*"
      attestors:
      - count: 1
        entries:
        - keyless:
            subject: "https://github.com/chainguard-images/images/.github/workflows/release.yaml@refs/heads/main"
            issuer: "https://token.actions.githubusercontent.com"
```

This policy instructs the Kubernetes API server to:

1.  Intercept all `Pod` creation requests.
2.  If the pod uses an image from `cgr.dev/chainguard/`, verify its signature.
3.  The signature must come from the official Chainguard release pipeline on GitHub.
4.  If the signature is invalid or missing, **reject** the pod creation request.

This provides a strong guarantee that only authorized, unmodified container images are running in our cluster.

## Exercise: Enforcing Policy

**Objective:** Add a Kyverno policy to the `platform` repository to enforce that all Crossplane Provider pods must also come from the Chainguard registry.

**Tasks:**

1.  Install Kyverno into your KinD cluster.
2.  Create a new `ClusterPolicy` manifest.
3.  The policy should match all pods in the `crossplane-system` namespace.
4.  It should verify that any image matching `cgr.dev/chainguard/provider-*` is properly signed.
5.  Add the policy to the `gitops-bootstrap/apps/platform` directory.
6.  Commit and push the change.
7.  Verify in the ArgoCD UI that the policy is synced and active.

This exercise demonstrates how to use GitOps to manage not just your infrastructure, but your security policies as well.

**➡️ [Next: Secret Management](./02-secret-management.md)**
