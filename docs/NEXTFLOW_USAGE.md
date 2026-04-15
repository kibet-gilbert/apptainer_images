# Using Apptainer Images in Nextflow Pipelines

> A guide to configuring and using `.sif` images — stored locally or in public registries (Quay.io / ghcr.io / BioContainers) — inside Nextflow pipelines.

---

## Table of Contents

1. [How Nextflow Runs Containers](#how-nextflow-runs-containers)
2. [Enabling Apptainer in Nextflow](#enabling-apptainer-in-nextflow)
3. [Using a Local .sif Image](#using-a-local-sif-image)
4. [Using an Image from a Registry](#using-an-image-from-a-registry)
   - [Docker Hub / Quay.io / GHCR](#docker-hub--quayio--ghcr)
   - [BioContainers (Quay.io)](#biocontainers-quayio)
   - [Seqera Wave Containers](#seqera-wave-containers)
   - [Sylabs Cloud Library](#sylabs-cloud-library)
5. [Process-level Container Directives](#process-level-container-directives)
6. [Global Container Configuration in nextflow.config](#global-container-configuration-in-nextflowconfig)
7. [Bind Mounts (Accessing Host Data)](#bind-mounts-accessing-host-data)
8. [Caching Pulled Images](#caching-pulled-images)
9. [Using Named Apps (%apprun) in Nextflow](#using-named-apps-apprun-in-nextflow)
10. [nf-core Pipelines and Apptainer](#nf-core-pipelines-and-apptainer)
11. [HPC / SLURM Example](#hpc--slurm-example)
12. [Troubleshooting](#troubleshooting)

---

## How Nextflow Runs Containers

When a Nextflow process has a `container` directive and the `apptainer` (or `singularity`) scope is enabled, Nextflow:

1. Pulls or locates the image (`.sif`)
2. Wraps the process `script` block in an `apptainer exec <image.sif> bash -c "..."` call
3. Automatically bind-mounts the working directory and any paths listed in `apptainer.runOptions`

No changes to your process scripts are needed — Nextflow handles the container invocation transparently.

---

## Enabling Apptainer in Nextflow

In `nextflow.config`:

```groovy
apptainer {
    enabled    = true
    autoMounts = true          // auto-mount CWD and common paths
}
```

Or pass it on the command line:

```bash
nextflow run main.nf -profile apptainer
```

> **Note:** `singularity.enabled = true` also works — Nextflow treats Apptainer and Singularity interchangeably since Apptainer is the direct successor.

---

## Using a Local .sif Image

If you have already built a `.sif` file on disk, reference it by absolute path in the process `container` directive.

```nextflow
process RUN_BLAST {
    container '/home/user/images/blast_2.15.0.sif'

    input:
    path query_fa

    output:
    path 'results.txt'

    script:
    """
    blastn -query ${query_fa} -db nt -out results.txt -outfmt 6
    """
}
```

Or set a global default in `nextflow.config` and override per-process:

```groovy
// nextflow.config
process.container = '/home/user/images/blast_2.15.0.sif'
```

---

## Using an Image from a Registry

### Docker Hub / Quay.io / GHCR

Nextflow automatically pulls OCI images and converts them to `.sif` when a `docker://` URI is given:

```nextflow
process RUN_BLAST {
    container 'docker://quay.io/gilykibe/blast:2.15.0'
    // or: container 'docker://ghcr.io/gilykibe/blast:2.15.0'
    // or: container 'docker://docker.io/gilykibe/blast:2.15.0'

    script:
    """
    blastn -query input.fa -db nt -out results.txt
    """
}
```

### BioContainers (Quay.io)

BioContainers images are hosted at `quay.io/biocontainers/`. Use the exact tag shown on [https://biocontainers.pro](https://biocontainers.pro):

```nextflow
process RUN_SAMTOOLS {
    container 'docker://quay.io/biocontainers/samtools:1.19--h50ea8bc_1'

    script:
    """
    samtools view -bS input.sam > output.bam
    """
}
```

Browse available tags:

```bash
# Using Apptainer directly to inspect available tags:
apptainer pull docker://quay.io/biocontainers/samtools:1.19--h50ea8bc_1
```

Or visit: `https://quay.io/repository/biocontainers/<tool_name>?tab=tags`

### Seqera Wave Containers

[Wave](https://seqera.io/containers/) provides on-demand container builds. After generating a container, copy the `docker://` URI shown in the Wave UI and use it directly:

```nextflow
process RUN_TOOL {
    container 'docker://community.wave.seqera.io/library/blast:2.15.0--<hash>'

    script:
    """
    blastn --version
    """
}
```

### Sylabs Cloud Library

```nextflow
process RUN_TOOL {
    container 'library://gilykibe/biotools/blast:2.15.0'

    script:
    """
    blastn --version
    """
}
```

---

## Process-level Container Directives

You can specify different containers for different processes in the same pipeline:

```nextflow
process ALIGN {
    container 'docker://quay.io/biocontainers/bwa:0.7.17--hed695b0_7'
    // ...
}

process CALL_VARIANTS {
    container 'docker://quay.io/biocontainers/gatk4:4.4.0.0--py36hdfd78af_0'
    // ...
}
```

Or use a `params` map for easy version management:

```groovy
// nextflow.config
params {
    containers {
        blast   = 'docker://quay.io/biocontainers/blast:2.15.0--h7d5a4b4_1'
        samtools = 'docker://quay.io/biocontainers/samtools:1.19--h50ea8bc_1'
    }
}
```

```nextflow
process RUN_BLAST {
    container params.containers.blast
    // ...
}
```

---

## Global Container Configuration in nextflow.config

```groovy
apptainer {
    enabled    = true
    autoMounts = true

    // Cache directory for pulled .sif files (prevents re-downloading)
    cacheDir   = '/scratch/apptainer_cache'

    // Extra flags passed to every `apptainer exec` call
    runOptions = '--bind /data:/data --bind /ref:/ref'
}
```

### Profiles for flexible deployment

```groovy
profiles {
    standard {
        process.executor = 'local'
    }

    apptainer {
        apptainer.enabled    = true
        apptainer.autoMounts = true
    }

    slurm_apptainer {
        process.executor     = 'slurm'
        apptainer.enabled    = true
        apptainer.autoMounts = true
        apptainer.cacheDir   = '/scratch/$USER/apptainer_cache'
    }
}
```

Run with a profile:

```bash
nextflow run main.nf -profile apptainer
nextflow run main.nf -profile slurm_apptainer
```

---

## Bind Mounts (Accessing Host Data)

Apptainer containers have read-only access to the filesystem by default. To make host directories accessible inside the container, use bind mounts.

### Automatic mounts

```groovy
apptainer {
    enabled    = true
    autoMounts = true    // mounts CWD, $HOME, /tmp automatically
}
```

### Manual bind mounts

```groovy
apptainer {
    enabled    = true
    runOptions = '--bind /data:/data --bind /reference_db:/ref'
}
```

Inside a process script, refer to the mounted paths directly:

```nextflow
process BLAST_SEARCH {
    container 'docker://quay.io/biocontainers/blast:2.15.0--h7d5a4b4_1'

    script:
    """
    blastn -query input.fa -db /ref/nt -out results.txt
    """
}
```

---

## Caching Pulled Images

Every time Nextflow pulls a `docker://` image, it converts it to a `.sif` and caches it. Set a persistent cache directory to avoid repeated downloads:

```groovy
// nextflow.config
apptainer {
    enabled  = true
    cacheDir = '/scratch/apptainer_cache'   // shared across pipeline runs
}
```

Or set the environment variable before running:

```bash
export NXF_APPTAINER_CACHEDIR=/scratch/apptainer_cache
nextflow run main.nf -profile apptainer
```

---

## Using Named Apps (%apprun) in Nextflow

If your container defines `%apprun` entry-points, call them via `apptainer run --app` inside the process `script` block:

```nextflow
process RUN_MGS2AMR {
    container '/home/user/images/mgs2amr.sif'

    input:
    path input_dir

    output:
    path 'output/'

    script:
    """
    apptainer run --app mgs2amr.sh ${task.container} \\
        --input ${input_dir} \\
        --output output/
    """
}
```

> **Note:** When Nextflow wraps a script in `apptainer exec`, `task.container` holds the resolved container path/URI. For `%apprun` you need `apptainer run --app`, so call it explicitly inside the script block as shown above.

---

## nf-core Pipelines and Apptainer

All [nf-core](https://nf-co.re) pipelines support Apptainer/Singularity out of the box. Run any nf-core pipeline with:

```bash
nextflow run nf-core/<pipeline> \
    -profile singularity \
    --input samplesheet.csv \
    --outdir results/
```

nf-core pipelines pull BioContainers images automatically. To use your own custom image, override the container for a specific process in your local `nextflow.config`:

```groovy
process {
    withName: 'CUSTOM_PROCESS' {
        container = '/home/user/images/my_tool.sif'
    }
}
```

---

## HPC / SLURM Example

A complete `nextflow.config` for an HPC cluster running SLURM with Apptainer:

```groovy
process {
    executor = 'slurm'
    queue    = 'normal'
    memory   = '16 GB'
    cpus     = 4
    time     = '4h'

    withName: 'RUN_BLAST' {
        container = 'docker://quay.io/biocontainers/blast:2.15.0--h7d5a4b4_1'
        cpus      = 8
        memory    = '32 GB'
    }
}

apptainer {
    enabled    = true
    autoMounts = true
    cacheDir   = '/scratch/$USER/apptainer_cache'
    runOptions = '--bind /lustre:/lustre'
}

executor {
    $slurm {
        queueSize = 50
    }
}
```

Submit the pipeline:

```bash
nextflow run main.nf \
    -profile slurm_apptainer \
    -resume \
    --input samplesheet.csv \
    --outdir results/
```

---

## Troubleshooting

| Problem | Likely cause | Fix |
|---|---|---|
| `FATAL: Unable to resolve image` | Wrong URI or registry not accessible | Verify the URI with `apptainer pull <uri>` manually |
| `Bind mount ... does not exist` | Host path in `runOptions` is missing | Create the directory or remove the bind option |
| `Permission denied` on `.sif` file | `.sif` in a `noexec` mounted filesystem | Move the cache to a filesystem that allows execution (`exec`) |
| Pipeline re-downloads image every run | `cacheDir` not set | Set `apptainer.cacheDir` in `nextflow.config` |
| `command not found` inside container | Binary not on container PATH | Check `%environment` in the `.def` file; add PATH export |
| `%apprun` entry-point not working | Using `apptainer exec` instead of `run` | Switch to `apptainer run --app <name>` in the script block |
| OOM / Java heap errors | Insufficient JVM memory | Increase `_JAVA_OPTIONS` in `%environment` and `memory` in process |
