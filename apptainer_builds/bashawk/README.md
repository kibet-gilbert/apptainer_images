# Basic Bash commands and Tools for File manipulation

Most bioinformatics analysis input and output are stored in structured text files.    
The most common file formats including databases are generally in the categories below:   
 - TSV, CSV - e.g SAM, VCF
 - JSON
 - XML
 - HTML
 - YAML
 - hd5
 - SQL

Most of these may need quick processing to explore their contents on the commnad line. For this to happen we need a command-line based tool that can intelligentky and quickly abstract the needed data or summaries from the file.

A detailed describtion of these tools has been curated by a team in this GitHub repository: [dbohdan/structured-text-tools](https://github.com/dbohdan/structured-text-tools).

## Container Overview

This container has a list of tools that can do basic file manipulation.    
They can easily be used in genomic data like converting FASTQ.gz files to FASTA format using AWK and gunzip.   
I have included tools like: `vim`, `xz`, `dasel`, `qq`, `xml2`, `xl`, `nushell`, `jq`, `column`, `sqlite`, `mlr`(miller), `perl`, `gzip`, `pigz`, `gawk`, ...

It is designed to be used with Nextflow or standalone via:
        apptainer exec <image.sif> [command] [arguments] ...
        Example:
        apptainer exec <image.sif> bash -c 'gunzip -c input.fastq.gz | awk ...'

```
Example:
apptainer exec bashawk/bash_awk.sif column -h
WARNING: SINGULARITY_TMPDIR and APPTAINER_TMPDIR have different values, using the latter
Usage:
 column [options] [<file>...]
Columnate lists.
Options:
 -t, --table                      create a table
 -n, --table-name <name>          table name for JSON output
 -O, --table-order <columns>      specify order of output columns
 -N, --table-columns <names>      comma separated columns names
 -l, --table-columns-limit <num>  maximal number of input columns
 -E, --table-noextreme <columns>  don't count long text from the columns to column width
 -d, --table-noheadings           don't print header
 -e, --table-header-repeat        repeat header for each page
 -H, --table-hide <columns>       don't print the columns
 -R, --table-right <columns>      right align text in these columns
 -T, --table-truncate <columns>   truncate text in the columns when necessary
 -W, --table-wrap <columns>       wrap text in the columns when necessary
 -L, --keep-empty-lines           don't ignore empty lines
 -J, --json                       use JSON output format for table
 -r, --tree <column>              column to use tree-like output for the table
 -i, --tree-id <column>           line ID to specify child-parent relation
 -p, --tree-parent <column>       parent to specify child-parent relation
 -c, --output-width <width>       width of output in number of characters
 -o, --output-separator <string>  columns separator for table output (default is two spaces)
 -s, --separator <string>         possible table delimiters
 -x, --fillrows                   fill rows before columns
 -h, --help                       display this help
 -V, --version                    display version
```
