# pandepth

TAFFISH wrapper for [PanDepth](https://github.com/HuiyangYu/PanDepth), an
ultrafast genomic coverage and depth calculator for SAM, BAM, CRAM, and PAF
alignment files.

This app packages upstream PanDepth `v2.26` as `pandepth 2.26-r2`. The default
TAFFISH command runs the upstream `pandepth` executable directly, and command
mode remains enabled so the same container environment can also be inspected
explicitly with `taf-pandepth pandepth ...`.

Release `2.26-r2` is a help-only TAFFISH update. It keeps the upstream
software, Dockerfile, runtime dependencies, smoke tests, and command behavior
unchanged from `2.26-r1`, and refreshes the terminal `taf-pandepth --help`
text.

Package metadata:

```text
name: pandepth
command: taf-pandepth
version: 2.26-r2
kind: tool
image: ghcr.io/taffish/pandepth:2.26-r2
upstream release: v2.26
upstream runtime version: [v2.26] in help banner
```

## Quick Start

Show the TAFFISH package version:

```sh
taf-pandepth --version
```

Show upstream PanDepth help. PanDepth uses `-h` for help, which conflicts with
the TAFFISH wrapper help flag, so pass it after `--` or use command mode:

```sh
taf-pandepth -- -h
taf-pandepth pandepth -h
```

Calculate whole-reference coverage from an alignment file:

```sh
taf-pandepth -i sample.bam -o sample -t 8
```

Run target-specific modes:

```sh
taf-pandepth -i sample.bam -g genes.gff3 -o sample.genes -t 8
taf-pandepth -i sample.bam -b regions.bed -o sample.regions -t 8
taf-pandepth -i sample.bam -w 100000 -o sample.windows -t 8
taf-pandepth -i sample.bam -a -o sample.sites -t 8
taf-pandepth -i sample.cram -r reference.fa -c -o sample.gc -t 8
```

## Input Notes

`-i` accepts SAM, BAM, CRAM, PAF, or a `.list` file containing one input path per
line. PAF input may be gzip-compressed, but PanDepth expects PAF records to
include CIGAR information. For Minimap2 or Winnowmap-generated PAF, use the
aligner's CIGAR-producing option, such as Minimap2 `-c`.

Sorted and indexed BAM/CRAM files are not required. If matching indexes are
present, PanDepth can use them for multi-threaded acceleration. CRAM decoding
and GC calculation require `-r reference.fa`.

## Main Options

```text
-i FILE       input SAM/BAM/CRAM/PAF file, or a .list file
-o PREFIX     output prefix
-g GFF/GTF    summarize gene or transcript features
-f TYPE       GFF/GTF feature type, default CDS
-b BED        summarize BED regions
-w INT        summarize fixed windows
-a            output per-site depth
-q INT        minimum mapping quality, default 0
-d INT        minimum site depth for statistics, default 1
-x INT        exclude reads with these SAM flag bits, default 1796
-t INT        number of threads, default 3
-r FASTA      reference genome for CRAM decode or GC calculation
-c            calculate GC content, requires -r
-h            upstream help
```

## Outputs

PanDepth writes gzip-compressed tabular outputs using the `-o` prefix:

```text
PREFIX.chr.stat.gz     whole-reference coverage and mean depth
PREFIX.gene.stat.gz    feature summaries when -g is used
PREFIX.bed.stat.gz     BED-region summaries when -b is used
PREFIX.win.stat.gz     fixed-window summaries when -w is used
PREFIX.SiteDepth.gz    per-site depth when -a is used
```

When `-c -r reference.fa` is used, GC columns are added to the relevant summary
files.

## Container Contents

The image contains:

```text
pandepth
gzip
Debian htslib runtime libraries
libdeflate and zlib runtime libraries
```

The app does not bundle aligners or alignment-preparation tools such as
Samtools, Minimap2, Winnowmap, BEDTools, or index-building helpers. Prepare SAM,
BAM, CRAM, PAF, BED, GFF/GTF, and reference FASTA inputs outside this app, then
pass those files to `taf-pandepth`.

## Build Notes

The Dockerfile builds from the upstream `v2.26` source tag and verifies commit
`afc2e5f10c60384dc5574846fadf8dba67476b44`. The upstream release also provides
an x86_64 binary and includes prebuilt static libraries in the source tree, but
this TAFFISH package compiles the source against Debian system `htslib`,
`libdeflate`, and `zlib` libraries. A small build-time include adjustment points
PanDepth at those system headers and libraries; no CLI behavior is changed.

This keeps the app native on both declared platforms:

```text
linux/amd64
linux/arm64
```

## Smoke Coverage

The package smoke checks are self-contained and offline. They verify:

```text
pandepth help and upstream version banner [v2.26]
runtime linkage to libhts and libdeflate
whole-reference coverage from a tiny SAM file
per-site output with -a
BED-region summaries
fixed-window summaries
GFF feature summaries from a .list input
GC-column output with -c -r
```

Smoke tests validate the container and a small functional path. They do not
replace full biological validation on real sequencing or assembly datasets.

## Upstream

- Project: PanDepth
- Source: <https://github.com/HuiyangYu/PanDepth>
- Packaged release: <https://github.com/HuiyangYu/PanDepth/releases/tag/v2.26>
- Upstream license: MIT
- Citation: Yu et al. 2024, *Briefings in Bioinformatics*
- DOI: <https://doi.org/10.1093/bib/bbae197>
- PMID: <https://pubmed.ncbi.nlm.nih.gov/38701418/>

## Maintainer Commands

```sh
taf check
taf build
docker build -t ghcr.io/taffish/pandepth:2.26-r2 -f docker/Dockerfile .
taf publish --release --dry-run
```

## License

The TAFFISH packaging files are licensed under Apache-2.0. Upstream PanDepth is
distributed under the MIT License.
