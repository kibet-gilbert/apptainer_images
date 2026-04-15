# Building & Testing an Apptainer Image from a Definition File

> Step-by-step instructions for converting a `.def` file into a `.sif` container image and verifying it works correctly.

---

## Table of Contents

1. [Prerequisites](#prerequisites)
2. [Prepare Your Build Directory](#prepare-your-build-directory)
3. [Prepare the conda.yml File](#prepare-the-condayml-file)
4. [Build the Image](#build-the-image)
5. [Test the Image](#test-the-image)
6. [Get the Help Message](#get-the-help-message)
7. [Run a Named App](#run-a-named-app)
8. [Execute an Arbitrary Command](#execute-an-arbitrary-command)
9. [Interactive Shell](#interactive-shell)
10. [Troubleshooting Common Build Errors](#troubleshooting-common-build-errors)

---

## Prerequisites

| Requirement | Minimum version | Check |
|---|---|---|
| Apptainer | 1.2+ | `apptainer --version` |
| Root / fakeroot access | — | See note below |
| Internet access | — | Needed to pull base image & conda packages |
| Free disk space | ≥ 10 GB | `df -h .` |

> **Root vs fakeroot:** Building from a `.def` file normally requires root privileges (`sudo`) because `%post` runs as root. On HPC systems where `sudo` is unavailable, use `apptainer build --fakeroot` (requires the system administrator to configure fakeroot for your user account).

---

## Prepare Your Build Directory

Organise all inputs in a single folder before building. This keeps the build context clean and makes the `%files` section paths predictable.

```
project/
├── package_apptainer.def   # definition file
├── conda.yml               # conda environment specification
└── optional_tarball.tar.gz # (if required by %files)
```

```bash
mkdir -p ~/apptainer_builds/my_tool
cd ~/apptainer_builds/my_tool

# Copy or download your files here
cp /path/to/package_apptainer.def .
cp /path/to/conda.yml .
```

---

## Prepare the conda.yml File

You must have a `conda.yml` file in the build directory before running the build. Choose **one** of the two methods below.

### Method 1 — Seqera Containers (easiest)

1. Open [https://seqera.io/containers/](https://seqera.io/containers/)
2. Search for each tool you need and add it to the container
3. Set **Container type → Singularity**
4. Click **Generate** then **Download conda.yml**
5. Save the file as `conda.yml` in your build directory

### Method 2 — Export a local conda environment

```bash
# Activate the environment you want to containerise
conda activate my_tool_env

# Export (cross-platform, no build strings)
conda env export --no-builds | grep -v "^prefix:" > conda.yml

# Or minimal (only explicitly installed packages — most portable)
conda env export --from-history | grep -v "^prefix:" > conda.yml
```

Verify the file looks reasonable before building:

```bash
head -30 conda.yml
```

---

## Build the Image

### Standard build (with sudo)

```bash
sudo apptainer build my_tool.sif package_apptainer.def
```

### Build with fakeroot (HPC — no sudo)

```bash
apptainer build --fakeroot my_tool.sif package_apptainer.def
```

### Build with a writable sandbox (useful for iterative development)

```bash
# Build a writable directory instead of a .sif
sudo apptainer build --sandbox my_tool_sandbox/ package_apptainer.def

# Shell into the sandbox to debug interactively
sudo apptainer shell --writable my_tool_sandbox/

# Once satisfied, convert the sandbox to a .sif
sudo apptainer build my_tool.sif my_tool_sandbox/
```

### Build with verbose logging

```bash
sudo apptainer build --notest my_tool.sif package_apptainer.def 2>&1 | tee build.log
```

> **Tip:** `--notest` skips the `%test` section during the build (run it manually afterwards). This speeds up iteration when debugging `%post`.

---

## Test the Image

The `%test` section is run automatically at the end of `apptainer build`. To run it manually at any time:

```bash
apptainer test my_tool.sif
```

Expected output (example):

```
=== Verifying tool availability ===
/opt/conda/bin/python
=== Python version ===
Python 3.11.8
=== All checks passed ===
```

If any `command -v` check fails, the test exits with a non-zero code and prints the failing command.

---

## Get the Help Message

Display the `%help` section of the image:

```bash
apptainer run-help my_tool.sif
```

Example output:

```
Apptainer container for my_tool v1.0.0

Usage:
    apptainer run --app <app_name> my_tool.sif [arguments]

Available apps:
    * my_tool_script.sh   – Main entry-point

Available tools (exec directly):
    * python
    * Rscript
```

---

## Run a Named App

If the definition file defines `%apprun` blocks, run a specific app with:

```bash
apptainer run --app <app_name> my_tool.sif [arguments]

# Examples:
apptainer run --app mgs2amr.sh my_tool.sif --help
apptainer run --app blastn my_tool.sif -query input.fa -db nt -out results.txt
```

List all available apps defined in the image:

```bash
apptainer inspect --list-apps my_tool.sif
```

---

## Execute an Arbitrary Command

Run any binary inside the container without a named app:

```bash
apptainer exec my_tool.sif <command> [arguments]

# Examples:
apptainer exec my_tool.sif python --version
apptainer exec my_tool.sif Rscript -e "sessionInfo()"
apptainer exec my_tool.sif blastn -version

# With bind mounts (to access host data):
apptainer exec --bind /data:/data my_tool.sif blastn \
    -query /data/input.fa \
    -db /data/nt \
    -out /data/results.txt
```

---

## Interactive Shell

Drop into a shell inside the container for exploration or debugging:

```bash
apptainer shell my_tool.sif

# With data directory mounted:
apptainer shell --bind /data:/data my_tool.sif
```

Inside the shell, you will see the `Apptainer>` prompt. Type `exit` to leave.

---

## Inspect Image Metadata

```bash
# View %labels metadata
apptainer inspect my_tool.sif

# View the definition file used to build the image
apptainer inspect --deffile my_tool.sif

# View the environment variables (%environment section)
apptainer inspect --environment my_tool.sif

# View the runscript (%runscript section)
apptainer inspect --runscript my_tool.sif
```

---

## Troubleshooting Common Build Errors

| Error | Likely cause | Fix |
|---|---|---|
| `FATAL: Unable to pull docker://...` | No internet access or Docker Hub rate limit | Check connectivity; authenticate with `apptainer remote login docker://docker.io` |
| `PackageNotFoundError` in micromamba | Package name/version not in conda channels | Check the package name on [anaconda.org](https://anaconda.org) and update `conda.yml` |
| `command not found` in `%test` | Binary not installed or not on `PATH` | Verify the package is in `conda.yml`; check `%environment` PATH exports |
| `R CMD INSTALL` fails | R not installed or wrong R version | Add `r-base` to `conda.yml` and pin the version |
| `No space left on device` during build | `/tmp` or working directory is full | Set `APPTAINER_TMPDIR` to a larger partition: `export APPTAINER_TMPDIR=/scratch/tmp` |
| `fakeroot: command not found` | fakeroot not configured by sysadmin | Ask your HPC administrator to enable fakeroot for your account |
| Build hangs on conda solve | Conflicting package versions | Simplify `conda.yml`; pin fewer versions; use `mamba` solver hints |
