# `snp-dists` Overview

`snp-dists` converts a FASTA alignment to a SNP distance matrix.

The code can be found on github: [tseemann/snp-dists](https://github.com/tseemann/snp-dists).


```
apptainer exec snp-dists_apptainer.sif snp-dists -h
WARNING: SINGULARITY_TMPDIR and APPTAINER_TMPDIR have different values, using the latter
SYNOPSIS
  Pairwise SNP distance matrix from a FASTA alignment
USAGE
  snp-dists [opts] aligned.fasta[.gz] > matrix.tsv
OPTIONS
  -h       Show this help
  -v       Print version and exit
  -j CPUS  Threads to use [1]
  -q       Quiet mode; no progress messages
  -a       Count all differences not just [AGTC]
  -k       Keep case, don't uppercase all letters
  -m       Output MOLTEN instead of TSV
  -c       Use comma instead of tab in output
  -b       Blank top left corner cell
URL
  https://github.com/tseemann/snp-dists
```
