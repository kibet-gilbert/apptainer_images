Below is a **complete, command-by-command explanation** of **every option you see in `biom -h`**, written from the perspective of **how BIOM is actually used in microbiome / metagenomics pipelines**.

I’ll first explain the **global options**, then **each subcommand**, with **what it does**, **when to use it**, and **typical examples**.

---

# 1. Global options (apply to all commands)

```
Usage: biom [OPTIONS] COMMAND [ARGS]...
```

### `--version`

```bash
biom --version
```

Shows:

* BIOM format version
* CLI toolkit version

**Use case**

* Pipeline reproducibility
* Debugging incompatible BIOM files (JSON vs HDF5)

---

### `-h, --help`

```bash
biom -h
biom convert -h
```

Displays help for:

* The main command, or
* A specific subcommand

---

# 2. Subcommands (core functionality)

---

## 2.1 `add-metadata`

```
biom add-metadata
```

### What it does

Adds **sample metadata** or **observation (feature) metadata** to an existing BIOM table.

### Why it matters

BIOM without metadata is just a matrix.
Metadata enables:

* Faceting
* Group comparisons
* Differential abundance
* Ordination coloring

### Typical metadata types

| Type        | Examples                         |
| ----------- | -------------------------------- |
| Sample      | body site, treatment, day        |
| Observation | taxonomy, gene family, AMR class |

### Common usage

```bash
biom add-metadata \
  -i table.biom \
  -o table_w_meta.biom \
  --sample-metadata-fp sample_metadata.tsv
```

Observation metadata (e.g., taxonomy):

```bash
biom add-metadata \
  -i table.biom \
  -o table_w_tax.biom \
  --observation-metadata-fp taxonomy.tsv \
  --sc-separated taxonomy
```

### Pipeline position

After table construction, before analysis.

---

## 2.2 `convert`

```
biom convert
```

### What it does

Converts **to or from BIOM format**.

### Supported formats

| From      | To                 |
| --------- | ------------------ |
| TSV       | BIOM (HDF5 / JSON) |
| BIOM      | TSV                |
| JSON BIOM | HDF5 BIOM          |

### Most common command (TSV → BIOM)

```bash
biom convert \
  -i table.tsv \
  -o table.biom \
  --table-type="OTU table" \
  --to-hdf5
```

### BIOM → TSV

```bash
biom convert \
  -i table.biom \
  -o table.tsv \
  --to-tsv
```

### Why HDF5 matters

* Smaller
* Faster
* Sparse matrix storage
* Required by QIIME 2

---

## 2.3 `from-uc`

```
biom from-uc
```

### What it does

Creates a BIOM table **directly from clustering output** produced by:

* `vsearch`
* `uclust`
* `usearch`

### Input

`.uc` clustering file

### Output

Feature × sample BIOM table

### Example

```bash
biom from-uc \
  -i clusters.uc \
  -o table.biom
```

### When to use

If you:

* Cluster reads into OTUs
* Want BIOM without intermediate TSV

Less common in modern ASV workflows.

---

## 2.4 `head`

```
biom head
```

### What it does

Shows the **first few rows and columns** of a BIOM table.

### Why useful

* Quick sanity check
* Avoid converting large BIOM → TSV

### Example

```bash
biom head -i table.biom
```

---

## 2.5 `normalize-table`

```
biom normalize-table
```

### What it does

Normalizes counts **per sample**.

### Common normalization

* Relative abundance (sum to 1)

### Example

```bash
biom normalize-table \
  -i table.biom \
  -o table_norm.biom
```

### Warning (important)

This is **simple normalization**, not:

* DESeq2
* TMM
* CLR
* Rarefaction

Use carefully for:

* Visualization
* Exploratory analysis

---

## 2.6 `show-install-info`

```
biom show-install-info
```

### What it does

Prints:

* Python version
* biom-format version
* HDF5 support
* Linked libraries

### Example

```bash
biom show-install-info
```

### When useful

* Debugging cluster installs
* Container validation
* CI/CD pipelines

---

## 2.7 `subset-table`

```
biom subset-table
```

### What it does

Extracts a **subset of samples or observations**.

### Filter by sample IDs

```bash
biom subset-table \
  -i table.biom \
  -o subset.biom \
  --sample-ids sample1,sample2
```

### Filter by observation IDs

```bash
biom subset-table \
  -i table.biom \
  -o subset.biom \
  --observation-ids OTU_1,OTU_2
```

### Pipeline role

* Remove controls
* Focus on specific taxa
* Split datasets

---

## 2.8 `summarize-table`

```
biom summarize-table
```

### What it does

Prints **high-level statistics** about a BIOM table.

### Example

```bash
biom summarize-table -i table.biom
```

### Output includes

* Number of samples
* Number of observations
* Table density
* Counts per sample
* Counts per feature

### First command you should run on any BIOM file.

---

## 2.9 `table-ids`

```
biom table-ids
```

### What it does

Prints IDs from the table.

### Sample IDs

```bash
biom table-ids -i table.biom --samples
```

### Observation IDs

```bash
biom table-ids -i table.biom --observations
```

### Why useful

* Debug mismatched metadata
* Validate joins
* Generate ID lists for filtering

---

## 2.10 `validate-table`

```
biom validate-table
```

### What it does

Checks whether a BIOM file is:

* Structurally valid
* Schema-compliant
* Internally consistent

### Example

```bash
biom validate-table -i table.biom
```

### Critical for

* Detecting corrupted files
* Ensuring QIIME compatibility
* HPC pipelines

---

# 3. Conceptual mapping (mental model)

```
Raw counts (TSV)
   ↓ biom convert
BIOM table
   ↓ biom add-metadata
Annotated BIOM
   ↓ biom subset / normalize
Analysis-ready BIOM
   ↓ QIIME / phyloseq / stats
```

---

# 4. Recommended usage pattern (best practice)

| Step         | Command                 |
| ------------ | ----------------------- |
| Inspect      | `biom summarize-table`  |
| Validate     | `biom validate-table`   |
| Add metadata | `biom add-metadata`     |
| Filter       | `biom subset-table`     |
| Normalize    | `biom normalize-table`  |
| Export       | `biom convert --to-tsv` |

---

# 5. In your context (HPC + containers)

You’re doing this correctly by using:

```bash
apptainer run biomformat.sif biom ...
```

This guarantees:

* Consistent biom-format version
* HDF5 compatibility
* No Python dependency drift

---

