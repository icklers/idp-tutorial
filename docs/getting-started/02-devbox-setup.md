# 02: Devbox Setup

Reproducibility is critical for a stable GitOps workflow. We need to ensure that every engineer on the team, as well as our CI/CD pipelines, are using the exact same versions of our command-line tools. We will use Nix and Devbox to achieve this.

## What is Devbox?

[Devbox](https://www.jetpack.io/devbox/) provides a user-friendly, JSON-based interface on top of Nix. It simplifies the process of managing your development environment.

We have already defined the necessary tools in the `devbox.json` file at the root of this project. Take a moment to inspect it:

```json
```json
{
  "packages": [
    "kubectl@latest",
    "gh@latest",
    "uv@latest"
  ],
  "shell": {
    "init_hook": [
      "echo 'Welcome to the GitOps Tutorial Dev Environment!'"
    ],
    "scripts": {
      "setup-tools": [
        "uv pip sync",
        "curl -sL \"https://raw.githubusercontent.com/crossplane/crossplane/main/install.sh\" | sh"
      ]
    }
  }
}
```

**Note on the Crossplane CLI and Python Dependencies:** The Crossplane CLI (`crossplane`) and all Python dependencies for MkDocs are installed via the `devbox run setup-tools` command. This ensures you always have the latest compatible versions, which is crucial for compatibility with Crossplane's rapid development cycle and for building the documentation.

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
# Should output v1.29.2 or the version pinned in devbox.json

crossplane version --client
# Should output v2.0.0 or the version pinned in devbox.json
```

With our environment set up, we can now install the core components.

**➡️ [Next: Local Cluster Setup](./03-local-cluster-setup.md)**
