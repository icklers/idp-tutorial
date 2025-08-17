# Troubleshooting

Even in a well-oiled GitOps machine, things can go wrong. This page provides a guide to debugging common issues in your Crossplane and ArgoCD setup.

## 1. My Claim Isn't Becoming `Ready`

This is the most common problem. Here's a systematic approach to debugging it.

### Step 1: `crossplane trace`

This is always your first step. It will show you the entire resource tree and pinpoint which resource is failing.

```bash
crossplane trace <KIND> <NAME>
# e.g., crossplane trace akscluster my-failing-cluster
```

Look for the resource that does not have `READY: True`.

### Step 2: `kubectl describe` the Failing Resource

Once you've identified the failing resource, use `kubectl describe` to get more details.

```bash
kubectl describe <KIND> <NAME>
# e.g., kubectl describe managedresource my-failing-vm
```

Look at the `Events` section at the bottom. This will often contain a specific error message from the Crossplane provider.

**Common Errors:**

-   **`403 Forbidden`:** The Crossplane provider's Service Principal or credentials do not have the required IAM permissions in the cloud provider.
-   **`InvalidRequest`:** You have passed an invalid value in the claim, which was then passed to the cloud provider API. Check your claim's `spec`.
-   **`NotFound`:** The resource was deleted out-of-band (e.g., in the cloud console), and Crossplane is trying to update it.

### Step 3: Check the Provider Pod Logs

If the events aren't clear, look at the logs of the relevant Crossplane provider pod.

```bash
# Find the provider pod
kubectl get pods -n crossplane-system | grep provider-azure

# Tail the logs
kubectl logs -f -n crossplane-system <PROVIDER_POD_NAME>
```

The logs will give you the raw error messages from the cloud provider's API.

## 2. ArgoCD App is `Degraded` or `Progressing`

### `Degraded`

-   **Check the Health Check:** The resource is failing its health check. Click on the resource in the ArgoCD UI to see the health status message.
-   **Crossplane Issue:** Often, an ArgoCD app is `Degraded` because a Crossplane resource it manages is failing. Use the troubleshooting steps above to debug the Crossplane side.

### `Progressing`

-   **Missing Health Check:** The most common reason for a resource to be stuck in `Progressing` is that ArgoCD doesn't have a custom health check for it. The resource might be perfectly healthy, but ArgoCD doesn't know how to verify it.
-   **Long-Running Operation:** The resource is legitimately taking a long time to create (e.g., provisioning a large database).

## 3. My `Composition` Isn't Being Selected

If you create a claim and nothing happens (no `CompositeResource` is created), it means Crossplane could not find a `Composition` to satisfy the claim.

-   **Check Labels/Selectors:** If your `Composition` uses a `claimSelector`, ensure the labels on your claim match the selector.
-   **Check `compositeTypeRef`:** Ensure the `compositeTypeRef` in your `Composition` matches the `group`, `version`, and `kind` of the `CompositeResourceDefinition`.
-   **Check `claimNames`:** Ensure the `claimNames` in your XRD match the `kind` of the claim you are creating.

## 4. Pods are in `ImagePullBackOff`

-   **Check Image Name:** You may have a typo in the container image name in your `Deployment` or `Composition`.
-   **Private Registry:** If the image is in a private registry, you need to create an `imagePullSecret` and attach it to the `Pod`'s `ServiceAccount`.
-   **Kyverno Policy:** If you are using an image signature policy, the pod may be blocked because the image is not signed or the signature is invalid. Check the Kyverno pod logs.

By following these steps, you should be able to diagnose and resolve the vast majority of issues you encounter in your GitOps journey.
