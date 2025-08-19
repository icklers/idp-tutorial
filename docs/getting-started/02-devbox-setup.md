# 02: Devbox Setup

Reproducibility is critical for a stable GitOps workflow. We need to ensure that every engineer on the team, as well as our CI/CD pipelines, are using the exact same versions of our command-line tools. We will use DevBox to achieve this.

## What is Devbox?

[Devbox](https://www.jetpack.io/devbox/) provides a user-friendly, JSON-based interface for managing reproducible development environments. It simplifies the process of ensuring consistent tooling across your team and CI/CD pipelines.

We have already defined the necessary tools in the `devbox.json` file at the root of this project. Take a moment to inspect it:

```json
{
  "packages": [
    "git@latest",
    "kubectl@latest",
    "gh@latest", 
    "uv@latest",
    "kind@latest",
    "kubeseal@latest",
    "azure-cli@latest"
  ],
  "shell": {
    "init_hook": [
      "[ ! -d .venv ] && uv venv",
      "source .venv/bin/activate",
      "export UV_LINK_MODE=copy",
      "echo 'Welcome to the GitOps Tutorial Dev Environment!'"
    ],
    "scripts": {
      "setup-tools": [
        "mkdir -p ~/bin/",
        "uv pip install -r pyproject.toml",
        "[ ! -e ~/bin/crossplane ] && curl -sL \"https://raw.githubusercontent.com/crossplane/crossplane/main/install.sh\" | sh > /dev/null",
        "mv crossplane ~/bin/ && chmod +x ~/bin/crossplane",
        "[ ! -e ~/bin/argocd ] && curl -sSL -o argocd-linux-amd64 https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64 && mv argocd-linux-amd64 ~/bin/argocd && chmod +x ~/bin/argocd",
        "echo 'export PATH=~/bin:$PATH' >> ~/.profile",
        "source ~/.profile"
      ]
    }
  }
}
```

**Package Breakdown:**
- **git@latest**: Version control for GitOps workflows
- **kubectl@latest**: Kubernetes command-line interface
- **gh@latest**: GitHub CLI for repository management
- **uv@latest**: Fast Python package installer and virtual environment manager
- **kind@latest**: Kubernetes in Docker for local development
- **kubeseal@latest**: CLI tool for encrypting secrets with Sealed Secrets
- **azure-cli@latest**: Azure command-line interface for cloud resources

**Setup Script Explanation:**
The `setup-tools` script performs several important tasks:

1. **Creates local bin directory**: `mkdir -p ~/bin/` - Safe location for user binaries
2. **Installs Python dependencies**: `uv pip install -r pyproject.toml` - MkDocs and documentation tools
3. **Downloads Crossplane CLI**: Latest stable version from official source
4. **Downloads ArgoCD CLI**: Latest stable version for GitOps management  
5. **Updates PATH**: Ensures tools are available in your shell session

**Note on Tool Versions:** The Crossplane CLI (`crossplane`) and ArgoCD CLI (`argocd`) are installed via the `devbox run setup-tools` command. This ensures you always have the latest compatible versions, which is crucial for compatibility with Crossplane's rapid development cycle.

As you can see, we have pinned the exact versions of `kubectl`, and all our other essential tools.

## Setup Instructions

### 1. Install Devbox

Next, install Devbox.

```bash
curl -fsSL https://get.jetpack.io/devbox | bash
```

### 2. Activate the Devbox Shell

Now, navigate to the root of the `crossplane-gitops-tutorial` directory and run:

```bash
devbox shell
```

This command will:

1.  Read the `devbox.json` file.
2.  Download and install the exact versions of all the packages listed.
3.  Activate a new shell session with all those tools available in your `PATH`.

You are now in a fully reproducible development environment. Every command you run in this shell will use the tools defined in our project, not your globally installed versions.

### 3. Install Project Tools

Run the `setup-tools` script to install the Crossplane CLI and Python dependencies for MkDocs:

```bash
devbox run setup-tools
```

To verify, run:

```bash
kubectl version --client
# Should output v1.29.2 or later

crossplane version --client
# Should output v1.16.0 or later

argocd version --client
# Should output ArgoCD CLI version

kubeseal --version
# Should output 0.24.0 or later

az version
# Should output Azure CLI version information
```

## Troubleshooting Common Issues

**Issue: Command not found after setup**
```bash
# Solution: Ensure PATH is updated
echo 'export PATH=~/bin:$PATH' >> ~/.bashrc  # or ~/.zshrc
source ~/.bashrc  # or ~/.zshrc
```

**Issue: Permission denied on downloaded binaries**
```bash
# Solution: Make binaries executable
chmod +x ~/bin/crossplane ~/bin/argocd
```

**Issue: DevBox shell not activating Python environment**
```bash
# Solution: Manually activate the virtual environment
source .venv/bin/activate
```

With our environment set up, we can now install the core components.

**➡️ [Next: Local Cluster Setup](./03-local-cluster-setup.md)**
