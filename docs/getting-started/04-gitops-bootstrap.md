# 04: GitOps Bootstrap with ArgoCD

Now that we have a blank Kubernetes cluster, we will kick off our entire platform using a single command. This is the essence of a true GitOps workflow.

## The GitOps Bootstrap Process

We will apply a single file to our cluster: `gitops-bootstrap/argocd/install.yaml`. This manifest contains the Custom Resource Definitions for ArgoCD.

Immediately after, we will apply our `platform-core.yaml`. This special `ApplicationSet` resource instructs ArgoCD to manage its own configuration and all other platform components from our Git repository.

### 1. Create Your Platform GitOps Repository

This tutorial provides the content for your "Platform GitOps Repo". You will create a new GitHub repository and push this content to it.

#### 1.1 Prepare your environment

First, set your GitHub username as an environment variable. This is used to authenticate with the repository when ArgoCD pulls the manifests.

```bash
export GITHUB_USERNAME="" # Replace with your GitHub username
```

Next set a name for your Platform GitOps repository. This is the repository you just created.

```bash
export GITHUB_REPO_NAME="" # Replace with your Platform GitOps repository name
```

Next, set the environment variable for your Platform GitOps repository URL:

```bash
export PLATFORM_GITOPS_REPO_URL="https://github.com/$GITHUB_USERNAME/$GITHUB_REPO_NAME.git"
```

Validate the environment variables:

```bash
echo "GITHUB_USERNAME: $GITHUB_USERNAME"
echo "GITHUB_REPO_NAME: $GITHUB_REPO_NAME"
echo "PLATFORM_GITOPS_REPO_URL: $PLATFORM_GITOPS_REPO_URL"
```

If the output looks correct, you are ready to proceed.

#### 1.2 Create your Platform GitOps Repository

Create your new Platform GitOps Repository using the tutorial repository as a template and clone it to your working directory. This gives you a clean copy of all the necessary files in your own repository.

```bash
# Create new repository from the tutorial template and clone it
gh repo create $GITHUB_USERNAME/$GITHUB_REPO_NAME --template icklers/idp-tutorial --public --clone

cd $GITHUB_REPO_NAME
```

Your platform repository is now ready and contains all the necessary GitOps configurations.

> If you want to create a private repository, refer to the ArgoCD Documentation about [Private Repositories in GitHub](https://argo-cd.readthedocs.io/en/stable/user-guide/private-repositories/#github-app-credential)

### 2. Prepare the `platform-core.yaml`

The `platform-core.yaml` manifest needs to point to your Platform GitOps repository. We will use a simple `sed` command to replace the placeholder with your repository URL.

```bash
sed -i "s|__YOUR_PLATFORM_GITOPS_REPO_URL__|$PLATFORM_GITOPS_REPO_URL|g" gitops-bootstrap/argocd/platform-core.yaml
```

> **Note on ApplicationSet Path:**
> Ensure that the `path` in the `platform-core.yaml`'s Git generator is set to `platform-core` if your `crossplane.yaml` is directly within that directory in your Git repository. If you intend to have subdirectories within `platform-core` for different applications, then the path should be `platform-core/*`.


### 3. Apply the Bootstrap Manifests

This is the **only manual `kubectl apply`** we will perform. First, we create the namespace for ArgoCD. Then, we apply the official installation manifest directly from the ArgoCD project's GitHub repository. This is the recommended way to ensure you are installing a stable and complete version of ArgoCD.

Finally, we apply our `platform-core.yaml` to tell our new ArgoCD instance to start managing our platform from Git.

```bash
# Validate cluster context
kubectl config current-context
```

Valid output: `kind-kind`

```bash
# Create the namespace for ArgoCD
kubectl create namespace argocd

# Apply the official ArgoCD installation manifest
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# Apply the Platform Core ApplicationSet to start the GitOps sync
kubectl apply -f gitops-bootstrap/argocd/platform-core.yaml
```

#### Validate ArgoCD Installation

```bash
kubectl -n argocd get pods,ApplicationSet
```

Wait about 60-100 seconds and check the status with the command above.
Once all pods are in the `Running` state and show "1/1" for the `READY` column, you can proceed to the next step.

### 4. Register Your GitOps Repository with ArgoCD

ArgoCD needs to know about your GitOps repository to pull manifests from it. Before adding the repository, you need to log in to the ArgoCD server using the `argocd` CLI. You will use the admin password you retrieved earlier.


Start a new shell and port-forward the ArgoCD server and keep the shell open:

```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

In the devbox terminal, retrieve the initial admin password for ArgoCD:

```bash
export ARGOCD_ADMIN_PASSWORD=$(kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d)
```

Then, log in to the ArgoCD server using the `argocd` CLI.:

```bash
argocd login localhost:8080 --username admin --password $ARGOCD_ADMIN_PASSWORD
```

> You will see a warning about insecure connections:"
>
> ```plaintext
> WARNING: server certificate had error: tls: failed to verify certificate: x509: certificate signed by unknown authority. Proceed insecurely (y/n)?
> ```
>
> This is expected since we are using port-forwarding and did not set up TLS. You can safely ignore this warning for local development.

Once logged in, you will need a GitHub Personal Access Token (PAT) with `repo` scope to allow ArgoCD to access your repository.

Follow the prompts in your browser to complete the authentication. Once authenticated, you can close the browser tab. The `gh` CLI will store the PAT securely. Use the `argocd repo add` command to register your repository. We will use the `gh auth token` command to securely retrieve your PAT.

```bash
argocd repo add $PLATFORM_GITOPS_REPO_URL --username $GITHUB_USERNAME --password $(gh auth token)
```

> If you encounter an authentication error, create a new **GitHub Personal Access Token (classic)** with `repo` scope in your [GitHub settings](https://github.com/settings/tokens). Set an expiration date for the token, copy it, and use it like this:
>
> ```bash
> export GITHUB_PAT="your-personal-access-token"
> argocd repo add $PLATFORM_GITOPS_REPO_URL --username $GITHUB_USERNAME --password $GITHUB_PAT
> ```

Validate the repository is added successfully:

```bash
argocd repo list
```

### 5. Observe the Magic

ArgoCD is now installing and configuring itself, Crossplane, and all the providers and compositions defined in your Git repository. To watch this happen, access the ArgoCD UI.

Echo the admin password for convenience, to use it when logging into the ArgoCD WebUI:

```bash
echo "ArgoCD Admin Password: $ARGOCD_ADMIN_PASSWORD"
```

Log in to [`https://localhost:8080`](https://localhost:8080) with username `admin` and the retrieved password. You will see the `platform-core` and all the other applications syncing automatically.

## Congratulations!

You have successfully bootstrapped a fully functional GitOps workflow. From this point forward, we will interact with our system exclusively through Git.

## Next Steps

Your GitOps foundation is now ready. Next, we'll add secure secret management capabilities that will be essential for managing cloud credentials and application secrets.

**➡️ [Next Section: Sealed Secrets Setup](05-sealed-secrets-setup.md)**