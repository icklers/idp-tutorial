# 01: Extending Crossplane with Composition Functions

Patch-and-transform is a powerful way to create `Compositions`, but it has its limits. What if you need to add complex logic, conditional resource creation, or loop over a list to generate resources? For this, we use **Composition Functions**.

## What are Composition Functions?

A Composition Function is a custom program that you write, package as a container image, and reference in your `Composition`. When Crossplane reconciles your `CompositeResource`, it sends the `Observed` and `Desired` state to your function. Your function then returns a new `Desired` state, which Crossplane applies.

This allows you to inject any logic you can imagine into the composition process.

### Why Use Them?

-   **Conditional Logic:** Only create a resource if a certain field in the claim is set (e.g., `if claim.spec.ha == true, create a read replica`).
-   **Looping:** Create a variable number of resources based on a list in the claim (e.g., `for each port in claim.spec.firewallPorts, create a FirewallRule`).
-   **External Lookups:** Call an external API to fetch data and use it to enrich your resources.
-   **Complex Validation:** Implement validation logic that is too complex for OpenAPI schema validation.

## How They Work

Your `Composition` is modified to run in `Pipeline` mode, which consists of a series of steps. One of these steps can be your function.

**Example `Composition` with a Function:**

```yaml
apiVersion: apiextensions.crossplane.io/v1
kind: Composition
metadata:
  name: my-function-composition
spec:
  compositeTypeRef: # ...
  mode: Pipeline
  pipeline:
    - step: patch-and-transform
      functionRef:
        name: function-patch-and-transform
    - step: my-custom-logic
      functionRef:
        name: my-awesome-function
      input:
        apiVersion: my-fn.example.org/v1alpha1
        kind: MyFunctionInput
        spec:
          someValue: "hello world"
```

In this example:

1.  The `mode` is set to `Pipeline`.
2.  The first step is the standard `patch-and-transform` function.
3.  The second step calls our custom function, `my-awesome-function`.
4.  We can pass custom `input` to our function.

## Writing a Composition Function

Composition Functions can be written in any language that can be packaged in a container. Go is a popular choice due to its strong typing and Kubernetes ecosystem tooling.

Crossplane provides libraries (`crossplane-runtime/pkg/function` and `crossplane-runtime/pkg/function/proto`) to simplify the process.

### The `FunctionRunner`

Your function's `main.go` will typically use a `FunctionRunner`.

```go
package main

import (
	"context"
	"fmt"

	"github.com/crossplane/crossplane-runtime/pkg/errors"
	"github.com/crossplane/crossplane-runtime/pkg/function"
	fnv1beta1 "github.com/crossplane/function-sdk-go/proto/v1beta1"
)

func main() {
	function.Run(function.HandlerFunc(func(ctx context.Context, req *fnv1beta1.RunFunctionRequest) (*fnv1beta1.RunFunctionResponse, error) {
		// 1. Get the observed and desired state from the request.
		observed, err := function.ParseObservedResources(req)
		if err != nil {
			return nil, errors.Wrap(err, "cannot parse observed resources")
		}
		desired, err := function.ParseDesiredResources(req)
		if err != nil {
			return nil, errors.Wrap(err, "cannot parse desired resources")
		}

		// 2. Add your custom logic here.
		// For example, loop over a field in the claim and add new resources to the `desired` map.

		// 3. Return the modified desired state.
		if err := function.ComposeDesiredResources(req, desired); err != nil {
			return nil, errors.Wrap(err, "cannot compose desired resources")
		}

		return req, nil
	}))
}
```

## Exercise: A Looping Function

**Objective:** Create a Composition Function that creates a set of Azure Firewall rules based on a list in the claim.

**Tasks:**

1.  **Define the XRD:** Create a `CompositeFirewall` with a claim, `Firewall`, that has a field `allowedPorts` which is an array of strings.
2.  **Write the Function:**
    -   Create a new Go project for your function.
    -   The function should get the `allowedPorts` array from the claim.
    -   It should loop through the array.
    -   For each port, it should create a new Azure `FirewallRule` resource and add it to the desired state.
3.  **Package and Push:** Build the function's Docker image and push it to a registry.
4.  **Create the `Function` resource:** Define a `Function` resource in your `platform` repository that points to your new image.
5.  **Create the `Composition`:** Create a `Composition` that uses your new function in its pipeline.
6.  **Claim a `Firewall`:** Create a `Firewall` claim with a list of ports.
7.  **Verify:** Check in the Azure portal that a firewall rule was created for each port in your claim.

Composition Functions are an advanced topic, but they unlock the full potential of Crossplane, allowing you to build a truly powerful and customized Internal Developer Platform.

**➡️ [Next: Multi-Cluster Management](./02-multi-cluster-management.md)**
