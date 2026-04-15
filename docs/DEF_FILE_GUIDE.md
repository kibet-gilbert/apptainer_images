# Apptainer Definition File: Anatomy & Reference

> A plain-language guide to every section of [`package_apptainer.def`](./template-package_apptainer.def) — the template used for building bioinformatics Apptainer (Singularity) images with `micromamba`.

---

## Table of Contents

1. [Overview](#overview)
2. [Header — Bootstrap & Base Image](#header--bootstrap--base-image)
3. [%files](#files)
4. [%post](#post)
5. [%environment](#environment)
6. [%test](#test)
7. [%runscript](#runscript)
8. [%apprun](#apprun)
9. [%help](#help)
10. [%labels](#labels)
11. [conda.yml — Creating the Conda Environment File](#condayml--creating-the-conda-environment-file)
12. [Placeholder Reference](#placeholder-reference)

---

## Overview

An Apptainer **definition file** (`.def`) is a recipe that describes how to build a portable, reproducible container image (`.sif`). It is divided into named **sections** (prefixed with `%`). Each section serves a distinct purpose during the build or runtime lifecycle.

```
Build time:   %files  →  %post
Runtime:      %environment  →  %runscript / %apprun / %test
Metadata:     %labels  %help
```

---

## Header — Bootstrap & Base Image

```singularity
BootStrap: docker
From: mambaorg/micromamba:1.5.10-noble
```

| Field | Description |
|---|---|
| `BootStrap` | The source type for the base layer. `docker` pulls from Docker Hub (or any OCI-compatible registry). Other options: `library`, `localimage`, `scratch`. |
| `From` | The base Docker image. `mambaorg/micromamba` provides a minimal Ubuntu-based image with `micromamba` pre-installed — ideal for conda-managed bioinformatics stacks. The tag `1.5.10-noble` pins both the micromamba version and the Ubuntu codename (24.04 Noble Numbat) for reproducibility. |

**Why micromamba?** It is a fast, lightweight reimplementation of `conda`/`mamba` with no base Python dependency, making it ideal for slim container builds.

---

## %files

```singularity
%files
    conda.yml /scratch/conda.yml
    optional_tarball.tar.gz /scratch/optional_tarball.tar.gz
```

Copies files from the **build host** into the container **at build time only**. Files placed in `/scratch/` inside the container are typically staging artefacts used during `%post` and are not needed at runtime.

| Column | Description |
|---|---|
| Left-hand path | Path on the host machine (relative to the build directory, or absolute). |
| Right-hand path | Destination inside the container. `/scratch/` is a conventional staging area. |

**Common files to include:**

- `conda.yml` — the conda environment specification (see [conda.yml section](#condayml--creating-the-conda-environment-file))
- Source tarballs for tools not available via conda (e.g. `gfaTools_v0.9.0-beta.tar.gz`)
- Patch files or small configuration files needed only during the build

> **Tip:** Do not copy large reference databases here. Mount them at runtime with `--bind`.

---

## %post

This is the most important section. It runs as **root** inside the container after the base image is bootstrapped. All software installation happens here.

### Typical build steps

#### 1 — Initialise micromamba

```bash
micromamba shell init -s bash -p /opt/conda
```

Configures the shell so that `micromamba activate` works inside the container.

#### 2 — Install system-level conda packages

```bash
micromamba install -y -n base \
    conda-forge::procps-ng \
    conda-forge::libiconv=1.17 \
    conda-forge::libidn2=2.3.2
```

Installs low-level system libraries that are often needed as implicit dependencies but may not be listed in `conda.yml`.

#### 3 — Install from conda.yml

```bash
micromamba install -y -n base -f /scratch/conda.yml
```

Installs all packages declared in the environment file into the `base` environment.

#### 4 — Export a lock file (reproducibility)

```bash
micromamba env export -n base --explicit > /environment.lock
```

Captures the exact resolved URLs of every installed package. This lock file can be committed to version control for bit-for-bit reproducibility.

#### 5 — Clean the package cache

```bash
micromamba clean -a -y
```

Removes downloaded tarballs and extracted package files, significantly reducing the final image size.

#### 6 — Install the main tool

Choose the method that fits your tool:

| Method | When to use |
|---|---|
| `git clone` | Tool lives on GitHub and has no conda package |
| Extract tarball | Pre-compiled binary distributed as an archive |
| `pip install` | Pure-Python package not in conda channels |
| Build from source | No pre-built binary available |

#### 7 — Language-specific extras

```bash
# R package from CRAN
Rscript -e "install.packages('pkg', repos='https://cloud.r-project.org')"

# R package from local tarball
R CMD INSTALL /scratch/pkg.tar.gz
```

#### 8 — Symlinks and helper directories

```bash
mkdir -p /opt/tool/tools
ln -s /opt/conda/bin/binary.sh /opt/tool/tools/binary.sh
```

Useful when a wrapper script expects dependencies at a specific relative path.

---

## %environment

```singularity
%environment
    export PATH="$MAMBA_ROOT_PREFIX/bin:/opt/conda/bin:$PATH"
    export CONDA_PREFIX="/opt/conda"
    export _JAVA_OPTIONS="-XX:CompressedClassSpaceSize=256m -XX:MaxMetaspaceSize=512m -Xss1m"
```

Variables set here are exported into the shell **every time the container starts** (both interactive and batch runs). They are written to `/.singularity.d/env/` inside the image.

| Variable | Purpose |
|---|---|
| `PATH` | Ensures conda-installed binaries and tool scripts are found. |
| `CONDA_PREFIX` | Tells R and Python where the conda installation lives. |
| `_JAVA_OPTIONS` | Reduces JVM memory overhead — important for tools that wrap Java (e.g. GATK, MetaCherchant). |

> **Note:** Variables set only in `%post` (without `export` to `%environment`) are **not** available at runtime.

---

## %test

```singularity
%test
    command -v python
    command -v Rscript
    python --version
    Rscript -e "library(pkg); packageVersion('pkg')"
```

Run automatically when you execute:

```bash
apptainer test image.sif
```

`command -v <binary>` returns exit code `1` if the binary is not found, causing the test to fail visibly. This is preferable to `which` in portable scripts.

**Best practices:**

- Check every critical binary with `command -v`
- Add at least one functional smoke-test (e.g. `tool --version`)
- Test language packages explicitly (R `library()`, Python `import`)

---

## %runscript

```singularity
#%runscript
#    exec <TOOL_BINARY> "$@"
```

Executed when the container is invoked with `apptainer run image.sif [args]`. When commented out, running the container drops you to a shell or does nothing, depending on the base image default.

Uncomment and set to the primary tool binary when there is a single obvious entry-point.

---

## %apprun

```singularity
%apprun mgs2amr.sh
    /opt/mgs2amr/mgs2amr.sh $*
```

Named application entry-points. Each `%apprun <name>` block defines a separate runnable app inside the same image. Invoked with:

```bash
apptainer run --app mgs2amr.sh image.sif [args]
```

This pattern is ideal for **multi-tool images** where a single container wraps several scripts or utilities. Define one `%apprun` block per exposed script.

> Use `$*` (or preferably `"$@"`) to forward all arguments to the script.

---

## %help

```singularity
%help
    Apptainer container for <TOOL_NAME> v<TOOL_VERSION>
    ...
```

Displayed by:

```bash
apptainer run-help image.sif
```

Write this section as you would a concise man page — include available apps, key binaries, and a usage example. This is the first thing users see.

---

## %labels

```singularity
%labels
    author="Your Name"
    author_email="you@example.com"
    tool="tool_name"
    tool_version="1.0.0"
```

Key-value metadata stored inside the image. Inspectable with:

```bash
apptainer inspect image.sif
```

Recommended labels: `author`, `author_email`, `tool`, `tool_version`, `base_image`, `description`.

---

## conda.yml — Creating the Conda Environment File

The `conda.yml` (or `conda.yaml`) file lists all conda packages to install. There are two primary ways to create it:

### Method 1 — Seqera Containers (recommended for reproducibility)

[https://seqera.io/containers/](https://seqera.io/containers/) is a free web service that resolves conda packages and generates a ready-to-use environment file and container.

**Steps:**

1. Go to [https://seqera.io/containers/](https://seqera.io/containers/)
2. Search for your tool(s) by name (e.g. `blast`, `bwa`, `samtools`)
3. Add each package to the container
4. Set **Container type** to `Singularity`
5. Click **Generate**
6. Download the `conda.yml` file (and optionally the pre-built `.sif`)

The generated file looks like:

```yaml
channels:
  - conda-forge
  - bioconda
  - defaults
dependencies:
  - bioconda::blast=2.15.0
  - bioconda::samtools=1.19
  - conda-forge::python=3.11
```

### Method 2 — Export a local conda environment

If you already have a working local environment:

```bash
# Full export (platform-specific, good for local reuse)
conda env export -n myenv > conda.yml

# Cross-platform export (no build strings — more portable)
conda env export -n myenv --no-builds > conda.yml

# Minimal export (only explicitly installed packages)
conda env export -n myenv --from-history > conda.yml
```

> **Tip:** The `--from-history` flag produces the most portable and readable file, but may under-specify versions. The `--no-builds` flag is a good middle ground for container builds.

**Trim the prefix line** before using the file in a container build:

```bash
grep -v "^prefix:" conda.yml > conda_clean.yml
```

---

## Placeholder Reference

Replace every `<PLACEHOLDER>` in the template before building:

| Placeholder | Replace with |
|---|---|
| `<TOOL_NAME>` | Short name of the tool (e.g. `blast`, `bwa`) |
| `<TOOL_VERSION>` | Version string (e.g. `2.15.0`) |
| `<TOOL_BINARY>` | The executable name (e.g. `blastn`) |
| `<YOUR_NAME>` | Your full name |
| `<YOUR_EMAIL>` | Your contact email |
| `<YYYY-MM-DD>` | Build date |
| `<ORG>` | GitHub organisation or username |
| `<REPO>` | GitHub repository name |
| `<PKG>` | Package name (R or Python) |
| `<RPACKAGE>` | R package name |
| `<OTHER_BINARY>` | Any additional tool binaries to expose |
