# 02: Nix & Devbox Setup

Reproducibility is critical for a stable GitOps workflow. We need to ensure that every engineer on the team, as well as our CI/CD pipelines, are using the exact same versions of our command-line tools. We will use Nix and Devbox to achieve this.

## What is Nix?

Nix is a powerful package manager for Linux and macOS that makes package management reliable and reproducible. It allows you to create isolated, declarative, and version-pinned development environments.

-   **Declarative:** You define the exact packages you need in a configuration file (`flake.nix`).
-   **Reproducible:** Nix guarantees that anyone who uses this file will get the exact same version of every package, down to the last bit.
-   **Isolated:** Projects can have their own set of dependencies without interfering with each other or your global system state.

## What is Devbox?

While Nix is incredibly powerful, its syntax can be steep. [Devbox](https://www.jetpack.io/devbox/) provides a user-friendly, JSON-based interface on top of Nix. It simplifies the process of managing your development environment.

We have already defined the necessary tools in the `devbox.json` file at the root of this project. Take a moment to inspect it:

```json
```json
{
  "packages": [
    "argocd@latest",
    "mkdocs@latest",
    "oh-my-zsh@latest",
    "zsh@latest",
    "kubectl@latest",
    "gh@latest",
    "uv@latest"
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
        "uv pip sync",
        "curl -sL \"https://raw.githubusercontent.com/crossplane/crossplane/main/install.sh\" | sh"
      ]
    }
  }
}
```

**Note on the `up` CLI:** You may notice `up` in our packages list. This is the official command-line tool from Upbound, the creators of Crossplane. It provides helpful commands for managing Crossplane packages, configurations, and providers. While we will primarily use `kubectl` in this tutorial, `up` is an essential tool for advanced Crossplane development and is included here for completeness.

**Note on the Crossplane CLI and Python Dependencies:** The Crossplane CLI (`crossplane`) and all Python dependencies for MkDocs are installed via the `devbox run setup-tools` command. This ensures you always have the latest compatible versions, which is crucial for compatibility with Crossplane's rapid development cycle and for building the documentation.

As you can see, we have pinned the exact versions of `kubectl`, and all our other essential tools.

## Setup Instructions

### 1. Install Nix

If you don't have Nix installed, follow the [official installation guide](https://nixos.org/download.html). We recommend the multi-user installation.

```bash
sh <(curl -L https://nixos.org/nix/install) --daemon
```

### 2. Install Devbox

Next, install Devbox.

```bash
curl -fsSL https://get.jetpack.io/devbox | bash
```

### 3. Activate the Devbox Shell

Now, navigate to the root of the `crossplane-gitops-tutorial` directory and run:

```bash
devbox shell
```

This command will:

1.  Read the `devbox.json` file.
2.  Use Nix to download and install the exact versions of all the packages listed.
3.  Activate a new shell session with all those tools available in your `PATH`.

You are now in a fully reproducible development environment. Every command you run in this shell will use the tools defined in our project, not your globally installed versions.

### 4. Install Project Tools

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
