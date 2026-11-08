# Bioinformatics 101

A short field guide for people starting computational work on cancer genomics: what to learn first, how to set up, and the habits that keep an analysis trustworthy. It is a map rather than a course; every section points to where to go deeper.

---

## 1. The working setup

**The shell.** Nearly every tool runs from a terminal. Learn to move around (`cd`, `ls`, `pwd`), look at files without opening them (`head`, `less`, `wc -l`, `cut`, `sort | uniq -c`), search (`grep`, `find`), and chain commands with pipes. Large genomic files are compressed: `zcat file.gz | head` reads them without unpacking.
→ [The Unix Shell (Software Carpentry)](https://swcarpentry.github.io/shell-novice/)

**Remote machines.** Real data lives on a cluster. Connect with `ssh`, copy with `rsync` (resumable, unlike `scp`), and keep long sessions alive in `tmux`. Heavy work goes through the scheduler, never on the login node:

```bash
sbatch job.sh          # submit a batch job
squeue -u $USER        # see your jobs
sacct -j <jobid>       # what happened after it ended (state, memory, time)
```

Ask for the memory and time a job actually needs; `sacct` tells you what it used last time.

**Environments.** Install tools per project, never into the system. Use [pixi](https://pixi.sh) or conda/mamba with the `conda-forge` and `bioconda` channels, and commit the lock file so anyone can rebuild the same environment. Containers (Apptainer/Singularity on clusters) are the next step when a pipeline has to run unchanged elsewhere.

**Version control.** Every analysis is a git repository: code, configs and small results tracked, raw data never. Commit small and often, with messages that say *why*.
→ [Version Control with Git (Software Carpentry)](https://swcarpentry.github.io/git-novice/)

## 2. Languages

- **Python**: `pandas`/`polars` for tables, `numpy`/`scipy` for numerics, `scanpy` for single-cell, `pysam` for alignments, `matplotlib`/`seaborn` for figures.
- **R / Bioconductor**: the reference implementations for most statistics in genomics: `DESeq2`, `edgeR`/`limma`, `GenomicRanges`, `minfi`, `survival`; `ggplot2` for figures.

Learn one well and read the other. Work in scripts, and keep notebooks for exploration.

## 3. The data, from sample to signal

| Stage | Format | What it holds | Typical tools |
|---|---|---|---|
| Raw reads | FASTQ | sequence + per-base quality | FastQC, MultiQC, fastp |
| Reference | FASTA (+ GTF/GFF) | genome sequence (+ gene models) | GENCODE, Ensembl |
| Alignments | BAM / CRAM | reads placed on the genome | BWA-MEM2, STAR, minimap2, samtools |
| Variants | VCF / MAF | SNVs, indels, structural variants | GATK Mutect2, Strelka2, bcftools |
| Intervals | BED | regions (peaks, targets, CpGs) | bedtools |
| Counts | matrix (TSV, H5AD, MTX) | features × samples / cells | featureCounts, salmon, Cell Ranger |
| Methylation | bedMethyl, IDAT | per-CpG methylation | modkit, Bismark, minfi |

Coordinate systems trip everyone once: **BED is 0-based, half-open; VCF and GTF are 1-based.** Always know which genome build you are on (GRCh38 vs GRCh37/hg19) and never mix them. Chromosome names differ too (`chr1` vs `1`).

## 4. Core analyses in cancer genomics

- **Somatic variants**: tumour vs matched normal; filter germline and artefacts; annotate with VEP or OncoKB.
- **Copy number**: from WGS/WES (e.g. ASCAT, CNVkit) or low-pass sequencing (ichorCNA). Purity and ploidy are estimated jointly and can be ambiguous at low tumour fraction, so read relative changes with care.
- **Expression**: bulk RNA-seq → counts → normalisation → differential expression → pathway or signature scores.
- **Single-cell and spatial**: QC → normalisation → clustering → annotation; the tumour microenvironment ("ecosystem") view comes from here.
- **Methylation and fragmentomics**: cfDNA from plasma carries tissue-of-origin and tumour signals in its methylation and fragment patterns.
- **Integration**: linking layers (genome, transcriptome, epigenome, clinical outcome) is where most of the biology is, and also where most errors creep in.

## 5. Statistics you cannot skip

- Many tests mean many false positives: correct with Benjamini–Hochberg (FDR), and report q-values.
- **Batch effects** often dominate biology. Plot a PCA coloured by batch, centre and date before anything else.
- Effect size plus confidence interval tells more than a p-value alone.
- Survival: Kaplan–Meier to look, Cox models to adjust; check proportional hazards.
- Separate exploration from confirmation: decide the test before you look at the answer.

## 6. Habits that keep an analysis trustworthy

1. **Raw data is read-only.** Never edit it in place; every derived file is produced by code.
2. **Freeze your inputs.** Record where each reference file came from (URL, version, date) and its checksum (`sha256sum`).
3. **Test small first.** Run every step on a small slice (one chromosome, 1% of reads) before the full job.
4. **Sanity-check every step.** Read counts, mapping rates, number of variants, distributions; a number that looks too good usually is.
5. **Make it rerunnable.** A workflow manager ([Snakemake](https://snakemake.readthedocs.io), [Nextflow](https://www.nextflow.io) / [nf-core](https://nf-co.re)) turns a pile of scripts into a pipeline anyone can rerun.
6. **Figures carry the finding.** A title that states the result, labelled axes with units, the numbers behind every figure saved alongside it.

## 7. Patient data

Genomic and clinical data are personal data, even when "de-identified". Work only where your approvals allow, keep controlled-access data (dbGaP, EGA) on the approved systems, never copy rows to a laptop, email or chat tool (AI assistants included), and share only aggregate results.

## 8. Where to go next

- [Biostar Handbook](https://www.biostarhandbook.com): practical genomics, from the shell up
- [Modern Statistics for Modern Biology](https://www.huber.embl.de/msmb/) (Holmes & Huber)
- [Orchestrating Single-Cell Analysis with Bioconductor](https://bioconductor.org/books/release/OSCA/)
- [Rosalind](https://rosalind.info): bioinformatics problems to practise on
- Public data: [cBioPortal](https://www.cbioportal.org), [GDC / TCGA](https://portal.gdc.cancer.gov), [GEO](https://www.ncbi.nlm.nih.gov/geo/), [UCSC Genome Browser](https://genome.ucsc.edu)

---

<sub>© 2026 Rohi Taitou Maimon (RTM).</sub>
