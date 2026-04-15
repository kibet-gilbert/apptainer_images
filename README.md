# Apptainer Container Recipes for Bioinformatics

[![Docker Hub](https://img.shields.io/badge/Docker%20Hub-kibetgilbert-2496ED?logo=docker&logoColor=white)](https://hub.docker.com/repositories/kibetgilbert)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Apptainer](https://img.shields.io/badge/Apptainer-1.2+-blue)](https://apptainer.org)

A curated collection of **Apptainer (Singularity) container recipes** for bioinformatics tools focused on antimicrobial resistance (AMR) profiling, metagenomics, whole-genome sequencing (WGS) analysis, variant annotation, single-cell transcriptomics and sequence quality control.

Built images are hosted on **Docker Hub** → [`kibetgilbert`](https://hub.docker.com/repositories/kibetgilbert) and can be pulled directly with Apptainer.

---

## Repository Structure

```
.
├── apptainer_builds/          # One subfolder per tool
│   ├── abricate/
│   │   ├── abricate_apptainer.def   # Apptainer definition file
│   │   ├── abricate_apptainer.sif   # Built image (not tracked by git)
│   │   └── conda.yml                # Conda environment spec
│   ├── fastp/
│   │   └── ...
│   └── ...                          # (one folder per tool)
│
├── docs/                      # Documentation
│   ├── DEF_FILE_GUIDE.md      # Anatomy of a .def file
│   ├── README.md              # Building & testing images
│   ├── DEPOSITING_IMAGES.md   # Pushing images to registries
│   ├── NEXTFLOW_USAGE.md      # Using images in Nextflow pipelines
│   └── template-package_apptainer.def   # Template definition file
│
├── images_registry.tsv        # Machine-readable image database (edit this!)
└── README.md                  # This file
```

> **`.sif` files are not tracked by git** — they are large binary files. Pull them from Docker Hub (see [Pulling Images](#pulling-images)) or build them locally from the `.def` files.

---

## Quick Start

### Pull a pre-built image from Docker Hub

```bash
apptainer pull docker://kibetgilbert/<image_name>:<tag>

# Example:
apptainer pull docker://kibetgilbert/fastp:0.23
apptainer pull docker://kibetgilbert/kraken2:2.1
apptainer pull docker://kibetgilbert/rgi:6.0.3
```

### Build an image locally from a definition file

```bash
cd apptainer_builds/fastp/
sudo apptainer build fastp_apptainer.sif fastp_apptainer.def
```

### Get help from an image

```bash
apptainer run-help fastp_apptainer.sif
```

For full build, test, and usage instructions, see the **[docs/](docs/)** folder.

---

## Image Registry

> The canonical source of truth is [`images_registry.tsv`](images_registry.tsv). The table below is rendered from it for convenience.  
> To update: edit the TSV and regenerate this table, or update both in sync.  
> Columns: **Tool** | **Version** | **Category** | **Description** | **GitHub** | **Citation** | **Docker Hub Image**

---

### AMR Detection & Resistance Gene Identification

| Tool | Version | Description | GitHub | Citation | Docker Hub |
|---|---|---|---|---|---|
| **ABRicate** | 1.0.1 | Mass screening of contigs for AMR and virulence genes | [tseemann/abricate](https://github.com/tseemann/abricate) | Seemann T. *ABRicate*, GitHub (2020) | [`kibetgilbert/abricate:1.0.1`](https://hub.docker.com/r/kibetgilbert/abricate) |
| **AMR++** | 3.0 | Nextflow pipeline for resistome profiling with MEGARes | [Microbial-Ecology-Group/AMRplusplus](https://github.com/Microbial-Ecology-Group/AMRplusplus) | Lakin SM et al. *Nucleic Acids Res* (2017) [doi](https://doi.org/10.1093/nar/gkw1009) | [`kibetgilbert/amrplusplus:latest`](https://hub.docker.com/r/kibetgilbert/amrplusplus) |
| **ARGem** | — | Antibiotic Resistance Gene detection tool | [cezar-sindrilaru/argem](https://github.com/cezar-sindrilaru/argem) | — | [`kibetgilbert/argem:latest`](https://hub.docker.com/r/kibetgilbert/argem) |
| **ARGNet (CPU)** | — | Deep learning-based ARG detection (CPU build) | [Keio-Bioinformatics-Lab/ARGNet](https://github.com/Keio-Bioinformatics-Lab/ARGNet) | — | [`kibetgilbert/argnetcpu:latest`](https://hub.docker.com/r/kibetgilbert/argnetcpu) |
| **ARGNet (GPU)** | — | Deep learning-based ARG detection (GPU build) | [Keio-Bioinformatics-Lab/ARGNet](https://github.com/Keio-Bioinformatics-Lab/ARGNet) | — | [`kibetgilbert/argnetgpu:latest`](https://hub.docker.com/r/kibetgilbert/argnetgpu) |
| **mgs2amr** | — | Metagenome-scale AMR gene detection (BLAST + MetaCherchant + R) | [pieterjanvc/mgs2amr](https://github.com/pieterjanvc/mgs2amr) | van Coillie PJ et al. GitHub | [`kibetgilbert/mgs2amr:latest`](https://hub.docker.com/r/kibetgilbert/mgs2amr) |
| **NCBI AMRFinderPlus** | 3.12.x | NCBI AMRFinderPlus for AMR genes and point mutations | [ncbi/amr](https://github.com/ncbi/amr) | Feldgarden M et al. *Sci Rep* (2021) [doi](https://doi.org/10.1038/s41598-021-91456-0) | [`kibetgilbert/ncbiamrfinderplus:3.12`](https://hub.docker.com/r/kibetgilbert/ncbiamrfinderplus) |
| **ResFinder** | 4.5 / 4.6 | Detection of acquired AMR genes from assembled genomes or reads | [genomicepidemiology/resfinder](https://github.com/genomicepidemiology/resfinder) | Bortolaia V et al. *J Antimicrob Chemother* (2020) [doi](https://doi.org/10.1093/jac/dkaa345) | [`kibetgilbert/resfinder:4.6`](https://hub.docker.com/r/kibetgilbert/resfinder) |
| **RGI** | 6.0.x | Resistance Gene Identifier using CARD database | [arpcard/rgi](https://github.com/arpcard/rgi) | Alcock BP et al. *Nucleic Acids Res* (2023) [doi](https://doi.org/10.1093/nar/gkac920) | [`kibetgilbert/rgi:6.0.3`](https://hub.docker.com/r/kibetgilbert/rgi) |

---

### Metagenomics & Taxonomic Classification

| Tool | Version | Description | GitHub | Citation | Docker Hub |
|---|---|---|---|---|---|
| **CCMetagen** | 1.4.x | Accurate metagenomic species classification using KMA | [vrmarcelino/CCMetagen](https://github.com/vrmarcelino/CCMetagen) | Marcelino VR et al. *Genome Biology* (2020) [doi](https://doi.org/10.1186/s13059-020-02014-2) | [`kibetgilbert/ccmetagen:1.4`](https://hub.docker.com/r/kibetgilbert/ccmetagen) |
| **CZ ID** | — | Chan Zuckerberg infectious disease metagenomics pipeline | [chanzuckerberg/czid-workflows](https://github.com/chanzuckerberg/czid-workflows) | — | [`kibetgilbert/czid:latest`](https://hub.docker.com/r/kibetgilbert/czid) |
| **CZ ID CLI** | latest | Command-line client for uploading samples to CZ ID | [chanzuckerberg/czid-cli](https://github.com/chanzuckerberg/czid-cli) | — | [`kibetgilbert/czid_cli:latest`](https://hub.docker.com/r/kibetgilbert/czid_cli) |
| **Kraken2** | 2.1.x | Ultrafast taxonomic classification using k-mer matching | [DerrickWood/kraken2](https://github.com/DerrickWood/kraken2) | Wood DE et al. *Genome Biology* (2019) [doi](https://doi.org/10.1186/s13059-019-1891-0) | [`kibetgilbert/kraken2:2.1`](https://hub.docker.com/r/kibetgilbert/kraken2) |
| **KrakenTools** | 1.2 | Downstream analysis scripts for Kraken/Bracken output | [jenniferlu717/KrakenTools](https://github.com/jenniferlu717/KrakenTools) | Lu J et al. *Nat Protoc* (2022) [doi](https://doi.org/10.1038/s41596-022-00738-y) | [`kibetgilbert/krakentools:1.2`](https://hub.docker.com/r/kibetgilbert/krakentools) |
| **Krona** | 2.8.x | Interactive hierarchical taxonomy visualisation | [marbl/Krona](https://github.com/marbl/Krona) | Ondov BD et al. *BMC Bioinformatics* (2011) [doi](https://doi.org/10.1186/1471-2105-12-385) | [`kibetgilbert/krona:2.8`](https://hub.docker.com/r/kibetgilbert/krona) |
| **MAGViral / MAGAmr** | — | MAG-based viral and AMR gene characterisation | — | — | [`kibetgilbert/magviral:latest`](https://hub.docker.com/r/kibetgilbert/magviral) |

---

### Whole-Genome Sequencing (WGS) & Assembly

| Tool | Version | Description | GitHub | Citation | Docker Hub |
|---|---|---|---|---|---|
| **BacPipe** | latest | Rapid WGS pipeline for clinical diagnostic bacteriology | [wholeGenomeSequencingAnalysisPipeline/BacPipe](https://github.com/wholeGenomeSequencingAnalysisPipeline/BacPipe) | Dandachi I et al. *iScience* (2020) [doi](https://doi.org/10.1016/j.isci.2019.100769) | [`kibetgilbert/bacpipe:latest`](https://hub.docker.com/r/kibetgilbert/bacpipe) |
| **Dragonflye** | 1.x | Fast bacterial genome assembler for Oxford Nanopore reads | [rpetit3/dragonflye](https://github.com/rpetit3/dragonflye) | Petitjean M. *GitHub* (2022) | [`kibetgilbert/dragonflye:latest`](https://hub.docker.com/r/kibetgilbert/dragonflye) |
| **GAMBIT** | 1.x | Genomic approximation method for bacterial ID and tracking | [jlumpe/gambit](https://github.com/jlumpe/gambit) | Lumpe J et al. *PLOS ONE* (2023) [doi](https://doi.org/10.1371/journal.pone.0277575) | [`kibetgilbert/gambit:latest`](https://hub.docker.com/r/kibetgilbert/gambit) |
| **Kleborate** | 2.x | Genotyping tool for *Klebsiella* (MLST, AMR, virulence, capsule) | [klebgenomics/Kleborate](https://github.com/klebgenomics/Kleborate) | Lam MMC et al. *Nat Commun* (2021) [doi](https://doi.org/10.1038/s41467-021-24448-3) | [`kibetgilbert/kleborate:2`](https://hub.docker.com/r/kibetgilbert/kleborate) |
| **MEGAHIT** | 1.2.x | Ultra-fast memory-efficient metagenomic assembler | [voutcn/megahit](https://github.com/voutcn/megahit) | Li D et al. *Bioinformatics* (2015) [doi](https://doi.org/10.1093/bioinformatics/btv033) | [`kibetgilbert/megahit:1.2`](https://hub.docker.com/r/kibetgilbert/megahit) |
| **SNP-dists** | 0.8.x | Pairwise SNP distance matrix from whole-genome alignments | [tseemann/snp-dists](https://github.com/tseemann/snp-dists) | Seemann T. *snp-dists*, GitHub | [`kibetgilbert/snpdist:0.8`](https://hub.docker.com/r/kibetgilbert/snpdist) |

---

### Quality Control & Preprocessing

| Tool | Version | Description | GitHub | Citation | Docker Hub |
|---|---|---|---|---|---|
| **BBTools** | 39.x | Suite for DNA/RNA-seq QC, trimming and preprocessing (BBDuk, reformat) | [BioInfoTools/BBMap](https://github.com/BioInfoTools/BBMap) | Bushnell B. *DOE JGI* (2014) [url](https://escholarship.org/uc/item/1h3515gn) | [`kibetgilbert/bbtools:39`](https://hub.docker.com/r/kibetgilbert/bbtools) |
| **bcl2fastq2 / SeqDataQC** | 2.20 | Illumina sequencing data QC and demultiplexing pipeline | [Illumina bcl2fastq2](https://support.illumina.com/sequencing/sequencing_software/bcl2fastq-conversion-software.html) | Illumina bcl2fastq2 v2.20 | [`kibetgilbert/seqdataqc:latest`](https://hub.docker.com/r/kibetgilbert/seqdataqc) |
| **fastp** | 0.23.x | Ultra-fast all-in-one FASTQ preprocessor and QC tool | [OpenGene/fastp](https://github.com/OpenGene/fastp) | Chen S et al. *Bioinformatics* (2018) [doi](https://doi.org/10.1093/bioinformatics/bty560) | [`kibetgilbert/fastp:0.23`](https://hub.docker.com/r/kibetgilbert/fastp) |
| **FastQC** | 0.12.x | Quality control tool for high-throughput sequencing data | [s-andrews/FastQC](https://github.com/s-andrews/FastQC) | Andrews S. *Babraham Bioinformatics* (2010) [url](https://www.bioinformatics.babraham.ac.uk/projects/fastqc/) | [`kibetgilbert/fastqc:0.12`](https://hub.docker.com/r/kibetgilbert/fastqc) |

---

### Host Depletion

| Tool | Version | Description | GitHub | Citation | Docker Hub |
|---|---|---|---|---|---|
| **Hostile** | 1.x | Accurate removal of host reads from metagenomic data | [bede/hostile](https://github.com/bede/hostile) | Constantinides B. *Bioinformatics* (2023) [doi](https://doi.org/10.1093/bioinformatics/btad728) | [`kibetgilbert/hostile:latest`](https://hub.docker.com/r/kibetgilbert/hostile) |
| **WiperTools** | — | Host contamination removal tools for sequencing reads | — | — | [`kibetgilbert/wipertools:latest`](https://hub.docker.com/r/kibetgilbert/wipertools) |

---

### Sequence Analysis & Variant Annotation

| Tool | Version | Description | GitHub | Citation | Docker Hub |
|---|---|---|---|---|---|
| **KMA** | 1.4.x | K-mer based alignment for mapping reads to redundant databases | [genomicepidemiology/KMA](https://github.com/genomicepidemiology/KMA) | Clausen PTLC et al. *BMC Bioinformatics* (2018) [doi](https://doi.org/10.1186/s12859-018-2336-6) | [`kibetgilbert/kma:1.4`](https://hub.docker.com/r/kibetgilbert/kma) |
| **SAMtools** | 1.20 | Suite for manipulating SAM/BAM/CRAM alignment files | [samtools/samtools](https://github.com/samtools/samtools) | Danecek P et al. *GigaScience* (2021) [doi](https://doi.org/10.1093/gigascience/giab008) | [`kibetgilbert/samtools:1.20`](https://hub.docker.com/r/kibetgilbert/samtools) |
| **seqtk** | 1.4 | Toolkit for processing sequences in FASTA/FASTQ formats | [lh3/seqtk](https://github.com/lh3/seqtk) | Li H. *seqtk*, GitHub | [`kibetgilbert/seqtk:1.4`](https://hub.docker.com/r/kibetgilbert/seqtk) |
| **SnpEff** | 5.x | Genetic variant annotation and functional effect prediction | [pcingola/SnpEff](https://github.com/pcingola/SnpEff) | Cingolani P et al. *Fly* (2012) [doi](https://doi.org/10.4161/fly.19695) | [`kibetgilbert/snpeff:5`](https://hub.docker.com/r/kibetgilbert/snpeff) |
| **SnpSift** | 5.x | Filter and manipulate annotated VCF files (companion to SnpEff) | [pcingola/SnpEff](https://github.com/pcingola/SnpEff) | Cingolani P et al. *Front Genet* (2012) [doi](https://doi.org/10.3389/fgene.2012.00035) | [`kibetgilbert/snpsift:5`](https://hub.docker.com/r/kibetgilbert/snpsift) |

---

### Data Formats & Utilities

| Tool | Version | Description | GitHub | Citation | Docker Hub |
|---|---|---|---|---|---|
| **BIOM Format** | — | R/Python tools for reading and writing BIOM format microbiome files | [joey711/biom](https://github.com/joey711/biom) | McDonald D et al. *GigaScience* (2012) [doi](https://doi.org/10.1186/2047-217X-1-7) | [`kibetgilbert/biomformat:latest`](https://hub.docker.com/r/kibetgilbert/biomformat) |
| **Bash/AWK** | — | Minimal Bash and AWK utility container for scripting | — | — | [`kibetgilbert/bashawk:latest`](https://hub.docker.com/r/kibetgilbert/bashawk) |
| **Git Tools** | — | Git and developer tools for CI/CD workflows | — | — | [`kibetgilbert/gittools:latest`](https://hub.docker.com/r/kibetgilbert/gittools) |
| **nf-core** | — | nf-core tools for Nextflow pipeline management | [nf-core/tools](https://github.com/nf-core/tools) | Ewels PA et al. *Nat Biotechnol* (2020) [doi](https://doi.org/10.1038/s41587-020-0439-x) | [`kibetgilbert/nf-core:latest`](https://hub.docker.com/r/kibetgilbert/nf-core) |

---

## Documentation

| Document | Description |
|---|---|
| [docs/DEF_FILE_GUIDE.md](docs/DEF_FILE_GUIDE.md) | Anatomy of an Apptainer `.def` file — every section explained |
| [docs/README.md](docs/README.md) | Step-by-step guide to building and testing a `.sif` image |
| [docs/DEPOSITING_IMAGES.md](docs/DEPOSITING_IMAGES.md) | How to push images to Sylabs, Docker Hub, GHCR, Quay.io |
| [docs/NEXTFLOW_USAGE.md](docs/NEXTFLOW_USAGE.md) | Using images locally and from registries in Nextflow pipelines |
| [docs/template-package_apptainer.def](docs/template-package_apptainer.def) | Annotated `.def` template for new images |

---

## Pulling Images

All images can be pulled directly with Apptainer using the `docker://` URI:

```bash
# Generic pattern
apptainer pull docker://kibetgilbert/<tool>:<tag>

# Examples
apptainer pull docker://kibetgilbert/abricate:1.0.1
apptainer pull docker://kibetgilbert/kraken2:2.1
apptainer pull docker://kibetgilbert/rgi:6.0.3
apptainer pull docker://kibetgilbert/fastp:0.23
```

Or use them directly in Nextflow by setting the container URI:

```groovy
process MY_PROCESS {
    container 'docker://kibetgilbert/abricate:1.0.1'
    ...
}
```

---

## Building a New Image

1. Copy the template: `docs/template-package_apptainer.def`
2. Create a folder: `apptainer_builds/<tool_name>/`
3. Add your `conda.yml` (from [Seqera Containers](https://seqera.io/containers/) or a local env)
4. Customise the `.def` file (replace all `<PLACEHOLDER>` values)
5. Build: `sudo apptainer build <tool>.sif <tool>.def`
6. Test: `apptainer test <tool>.sif`
7. Push: `apptainer push <tool>.sif oras://docker.io/kibetgilbert/<tool>:<tag>`

See [docs/README.md](docs/README.md) for the full walkthrough.

---

## Updating the Image Registry

The [`images_registry.tsv`](images_registry.tsv) file is the editable source of truth for all image metadata. It has the following columns:

```
tool | folder | version | category | description | github_url | citation | doi_or_url | docker_hub_image | notes
```

Edit it directly with any spreadsheet application or text editor. The README table above should be kept in sync when new images are added.

---

## Author

**Gilbert Kibet**  
📧 kibet.gilbert.r@gmail.com  
🐳 [hub.docker.com/repositories/kibetgilbert](https://hub.docker.com/repositories/kibetgilbert)

---

## License

![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)   
This repository (definition files and documentation) is released under the [MIT License](LICENSE).  
Individual tools retain their own licenses — please cite the appropriate paper(s) when using each tool in published research.

Users are responsible for:
- Reviewing the licenses of individual tools
- Complying with their terms
- Citing the appropriate publications when using them in research
