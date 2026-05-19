taf-pandepth 2.26-r2

TAFFISH wrapper for PanDepth 2.26, an ultrafast depth and coverage calculator
for SAM, BAM, CRAM, and PAF alignment files.

Usage:
  taf-pandepth [-h | --help]
  taf-pandepth [-v | --version]
  taf-pandepth [PANDEPTH_OPTIONS...]
  taf-pandepth -- [PANDEPTH_OPTIONS...]
  taf-pandepth --compile [PANDEPTH_OPTIONS...]

Wrapper options:
  -h, --help       Show this TAFFISH help
  -v, --version    Show TAFFISH package version
  --compile        Print generated shell code instead of running it
  --               Stop parsing TAFFISH wrapper options

Upstream help:
  taf-pandepth -- -h
  taf-pandepth pandepth -h

Examples:
  taf-pandepth -i sample.bam -o sample -t 8
  taf-pandepth -i sample.bam -g genes.gff3 -o sample.genes -t 8
  taf-pandepth -i sample.bam -b regions.bed -o sample.regions -t 8
  taf-pandepth -i sample.bam -w 100000 -o sample.windows -t 8
  taf-pandepth -i sample.bam -a -o sample.sites -t 8
  taf-pandepth -i sample.cram -r reference.fa -c -o sample.gc -t 8

Common options:
  -i FILE       input SAM/BAM/CRAM/PAF file, or a .list file
  -o PREFIX     output prefix
  -g GFF/GTF    summarize gene or transcript features
  -f TYPE       GFF/GTF feature type, default CDS
  -b BED        summarize BED regions
  -w INT        summarize fixed windows
  -a            output per-site depth
  -q INT        minimum mapping quality
  -t INT        number of threads
  -r FASTA      reference genome for CRAM decode or GC calculation
  -c            calculate GC content, requires -r

Outputs:
  PREFIX.chr.stat.gz, PREFIX.gene.stat.gz, PREFIX.bed.stat.gz,
  PREFIX.win.stat.gz, PREFIX.SiteDepth.gz depending on selected modes.

Notes:
  PanDepth does not provide a separate --version command. The upstream version
  is checked through the -h help banner, which reports [v2.26].
  PAF input must include CIGAR information; for Minimap2 PAF, use minimap2 -c.
  CRAM decoding and GC calculation require -r reference.fa.
  This image includes pandepth, gzip, htslib runtime libraries, libdeflate, and
  zlib. It does not include Samtools, Minimap2, Winnowmap, BEDTools, or other
  alignment-preparation tools.

Container:
  image: ghcr.io/taffish/pandepth:2.26-r2
  platforms: linux/amd64, linux/arm64

Upstream:
  source:  https://github.com/HuiyangYu/PanDepth
  release: v2.26
  license: MIT
  citation: Yu et al. 2024, doi:10.1093/bib/bbae197
