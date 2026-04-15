# Depositing Apptainer Images in Public Repositories

> A guide to uploading and sharing your `.sif` container images on Sylabs Cloud Library, Docker Hub, GitHub Packages, Quay.io, and BioContainers.

---

## Table of Contents

1. [Overview of Repository Options](#overview-of-repository-options)
2. [Sylabs Cloud Library (library://)](#sylabs-cloud-library-library)
3. [Docker Hub (via OCI push)](#docker-hub-via-oci-push)
4. [GitHub Container Registry (ghcr.io)](#github-container-registry-ghcrio)
5. [Quay.io](#quayio)
6. [BioContainers / Bioconductor](#biocontainers--bioconductor)
7. [Naming & Tagging Conventions](#naming--tagging-conventions)
8. [Automating Uploads with GitHub Actions](#automating-uploads-with-github-actions)

---

## Overview of Repository Options

| Repository | Best for | Protocol | Free tier |
|---|---|---|---|
| [Sylabs Cloud Library](#sylabs-cloud-library-library) | Singularity/Apptainer-native storage | `library://` | Yes (11 GB) |
| [Docker Hub](#docker-hub-via-oci-push) | Wide compatibility, OCI images | `docker://` | Yes (public repos) |
| [GitHub Container Registry](#github-container-registry-ghcrio) | CI/CD integration with GitHub | `oras://` / `docker://` | Yes (public repos) |
| [Quay.io](#quayio) | Red Hat / bioinformatics community | `docker://` | Yes (public repos) |
| [BioContainers](#biocontainers--bioconductor) | Bioinformatics-specific, auto-built | PR to bioconda | Yes (community) |

---

## Sylabs Cloud Library (library://)

Sylabs Cloud Library is the native registry for Singularity/Apptainer images. Images are pushed and pulled using the `library://` protocol.

### 1. Create an account

Go to [https://cloud.sylabs.io](https://cloud.sylabs.io) and sign up.

### 2. Generate an access token

```
Account → Access Tokens → Create Token → Copy
```

### 3. Authenticate Apptainer

```bash
apptainer remote login
# Paste your token when prompted
```

### 4. Push the image

```bash
apptainer push my_tool.sif library://<username>/<collection>/<image_name>:<tag>

# Example:
apptainer push blast_2.15.0.sif library://gilykibe/biotools/blast:2.15.0
```

### 5. Pull the image (by anyone)

```bash
apptainer pull library://gilykibe/biotools/blast:2.15.0
```

### 6. Sign the image (optional but recommended)

```bash
# Generate a PGP key (once)
apptainer key newpair

# Sign the image
apptainer sign blast_2.15.0.sif

# Push public key to Sylabs keystore
apptainer key push <FINGERPRINT>
```

---

## Docker Hub (via OCI push)

Apptainer can push images to any OCI-compatible registry (Docker Hub, Quay, GHCR) using the `oras://` or OCI-SIF format.

### 1. Create a Docker Hub account

Go to [https://hub.docker.com](https://hub.docker.com) and sign up.

### 2. Log in via Apptainer

```bash
apptainer registry login --username <dockerhub_username> docker://docker.io
# Enter your password or access token
```

### 3. Push as an OCI artifact

```bash
apptainer push my_tool.sif oras://docker.io/<username>/<image_name>:<tag>

# Example:
apptainer push blast_2.15.0.sif oras://docker.io/gilykibe/blast:2.15.0
```

### 4. Pull the image

```bash
apptainer pull oras://docker.io/gilykibe/blast:2.15.0
```

---

## GitHub Container Registry (ghcr.io)

GitHub Container Registry (ghcr.io) integrates seamlessly with GitHub repositories and GitHub Actions CI/CD.

### 1. Generate a GitHub Personal Access Token (PAT)

Go to:
```
GitHub → Settings → Developer Settings → Personal Access Tokens → Generate new token
```

Grant scopes: `write:packages`, `read:packages`, `delete:packages`.

### 2. Log in via Apptainer

```bash
echo $GITHUB_TOKEN | apptainer registry login --username <github_username> --password-stdin oras://ghcr.io
```

### 3. Push the image

```bash
apptainer push my_tool.sif oras://ghcr.io/<github_username>/<image_name>:<tag>

# Example:
apptainer push blast_2.15.0.sif oras://ghcr.io/gilykibe/blast:2.15.0
```

### 4. Make the package public

By default, packages on ghcr.io are private. To make public:
```
GitHub → Your profile → Packages → <package_name> → Package Settings → Change visibility → Public
```

### 5. Pull the image

```bash
apptainer pull oras://ghcr.io/gilykibe/blast:2.15.0
```

---

## Quay.io

Quay.io is maintained by Red Hat and is popular in the bioinformatics community (especially for BioContainers).

### 1. Create a Quay.io account

Go to [https://quay.io](https://quay.io) and sign up.

### 2. Create a repository

```
Quay.io → + Create New Repository → Container Image Repository → Public → Create
```

### 3. Log in via Apptainer

```bash
apptainer registry login --username <quay_username> docker://quay.io
# Enter your Quay.io password or robot token
```

### 4. Push the image

```bash
apptainer push my_tool.sif oras://quay.io/<username>/<image_name>:<tag>

# Example:
apptainer push blast_2.15.0.sif oras://quay.io/gilykibe/blast:2.15.0
```

### 5. Pull the image

```bash
apptainer pull oras://quay.io/gilykibe/blast:2.15.0
```

---

## BioContainers / Bioconductor

BioContainers ([https://biocontainers.pro](https://biocontainers.pro)) automatically builds and hosts containers for every package in [Bioconda](https://bioconda.github.io/). If your tool is in Bioconda, a Docker and Singularity image is likely **already available**.

### Check if a BioContainer already exists

```bash
# Pull directly — BioContainers are hosted on Quay.io and ghcr.io
apptainer pull docker://quay.io/biocontainers/blast:2.15.0--h7d5a4b4_1

# Or browse: https://biocontainers.pro/tools/<tool_name>
```

### Submit your tool to Bioconda (to get a BioContainer automatically)

1. Fork [https://github.com/bioconda/bioconda-recipes](https://github.com/bioconda/bioconda-recipes)
2. Add a recipe under `recipes/<tool_name>/`
3. Open a Pull Request — CI will build and test the package
4. Once merged, BioContainers builds Docker and Singularity images automatically

### Seqera Containers (Wave)

[https://seqera.io/containers/](https://seqera.io/containers/) builds on-demand Singularity containers from conda packages and hosts them for free. Images are immediately available via a URL returned by the Wave service — no upload needed.

---

## Naming & Tagging Conventions

Consistent naming makes images easy to find and cite:

```
<registry>/<namespace>/<tool_name>:<tool_version>--<build_hash>

# Examples:
quay.io/biocontainers/samtools:1.19--h50ea8bc_1
ghcr.io/gilykibe/blast:2.15.0
library://gilykibe/biotools/mgs2amr:1.0.0
```

| Component | Recommendation |
|---|---|
| `tool_name` | Lowercase, hyphens instead of underscores |
| `tool_version` | Semantic version (e.g. `1.2.3`) — avoid `latest` |
| `build_hash` | Short git SHA or conda build string for traceability |

---

## Automating Uploads with GitHub Actions

Place this workflow in `.github/workflows/build_push.yml` to automatically build and push your image on every tagged release.

```yaml
name: Build and Push Apptainer Image

on:
  push:
    tags:
      - 'v*'          # triggers on tags like v1.0.0

jobs:
  build-push:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Install Apptainer
        run: |
          sudo apt-get update
          sudo apt-get install -y apptainer

      - name: Build image
        run: |
          sudo apptainer build my_tool.sif package_apptainer.def

      - name: Log in to GHCR
        run: |
          echo "${{ secrets.GITHUB_TOKEN }}" | \
          apptainer registry login --username ${{ github.actor }} \
            --password-stdin oras://ghcr.io

      - name: Push image to GHCR
        run: |
          TAG=${GITHUB_REF_NAME}          # e.g. v1.0.0
          apptainer push my_tool.sif \
            oras://ghcr.io/${{ github.repository_owner }}/my_tool:${TAG}
```

> **Secret:** `GITHUB_TOKEN` is automatically available in GitHub Actions — no manual setup required for ghcr.io pushes.
