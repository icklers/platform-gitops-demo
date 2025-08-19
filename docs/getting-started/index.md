# Getting Started

Welcome to the comprehensive GitOps and Crossplane tutorial! This section will guide you through setting up a complete Internal Developer Platform (IDP) using GitOps principles.

## What You'll Build

By the end of this section, you'll have:
- ✅ A reproducible development environment using DevBox
- ✅ A local Kubernetes cluster running on KinD
- ✅ ArgoCD managing your platform through GitOps
- ✅ Secure secret management with Sealed Secrets
- ✅ Foundation for Crossplane infrastructure automation

## Repository Structure

This tutorial repository is organized as follows:

```
idp-tutorial/
├── docs/                    # Tutorial documentation (you're reading this)
├── exercises/               # Hands-on exercise files
├── gitops-bootstrap/        # Ready-to-use bootstrap configurations
├── platform-core/          # Pre-populated platform configurations
└── repository-templates/    # Templates for creating new repositories
```

- **docs/**: Step-by-step tutorial content
- **exercises/**: Practice files organized by module
- **gitops-bootstrap/**: Production-ready GitOps configurations
- **platform-core/**: Platform components managed by ArgoCD

## Prerequisites

Before starting, ensure you have:
- **macOS or Linux** (Windows with WSL2 also works)
- **Git** installed and configured
- **GitHub account** with Personal Access Token
- **Azure subscription** (free tier is sufficient for most exercises)
- **8GB+ RAM** recommended for local cluster
- **10GB+ free disk space**

## Prerequisites Validation

Before starting, validate your environment:

### 1. Git Configuration
```bash
git --version
# Expected: git version 2.40.0 or later

git config --global user.name
git config --global user.email
# Expected: Your name and email configured
```

### 2. GitHub Access
```bash
gh auth status
# Expected: ✓ Logged in to github.com as <username>
```

### 3. Azure CLI
```bash
az account show
# Expected: JSON output showing your active subscription
```

## Learning Path Overview

### Module 1: Introduction (5 minutes)
- Understand the core components and architectural vision
- Learn about GitOps principles and IDP concepts

### Module 2: DevBox Setup (15 minutes)  
- Install DevBox for reproducible development environment
- Set up all required tools and dependencies

### Module 3: Local Cluster Setup (10 minutes)
- Create a KinD cluster for local development
- Verify cluster connectivity and basic operations

### Module 4: GitOps Bootstrap (30 minutes)
- Deploy ArgoCD using GitOps principles
- Connect your platform repository
- Experience the "magic" of GitOps automation

### Module 5: Sealed Secrets Setup (20 minutes)
- Add secure secret management capabilities
- Learn to encrypt secrets for safe Git storage
- Test the complete secret management workflow

**Total estimated time: 80 minutes**

## Next Steps

Ready to begin? Start with understanding the foundational concepts and architectural vision.

**➡️ [Begin: Introduction to GitOps Workflow](01-introduction.md)**