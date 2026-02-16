# MHS Files Generation Snakefile

## Overview

This Snakefile automates the conversion of CRAM/BAM files from the 1000 Genomes Project (30X coverage) into multi-hetsep (mhs) format files required by *cobraa*. The mhs format, introduced by Stephan Schiffels, is used for coalescent-based inference methods like PSMC, MSMC, and cobraa.

## Purpose

The mhs file generation pipeline is a crucial preprocessing step that:
1. Converts raw sequencing data (CRAM format) into analysis-ready variant files
2. Applies quality filters and mappability masks
3. Generates the mhs format files needed for downstream population genetic inference

## Pipeline Steps

The Snakefile implements a multi-step pipeline with the following rules:

### 1. `convert_cram_to_bam`
- **Input**: CRAM file from 1000 Genomes 30X dataset
- **Output**: Temporary BAM file
- **Purpose**: Converts CRAM format to BAM format using samtools
- **Reference**: Uses GRCh38 reference genome

### 2. `sort_bam`
- **Input**: Unsorted BAM file
- **Output**: Temporary sorted BAM file
- **Purpose**: Sorts BAM file by coordinate for indexing and downstream processing

### 3. `index_bam`
- **Input**: Sorted BAM file
- **Output**: Temporary BAM index (.bai)
- **Purpose**: Creates index for efficient BAM file access

### 4. `convert_bam_to_vcf_and_bed`
- **Input**: Sorted and indexed BAM file
- **Output**: VCF file (variants) and BED file (callable regions)
- **Purpose**: Calls variants using bcftools and generates mask for callable regions
- **Key parameters**:
  - Mapping quality filter: -q 20
  - Base quality filter: -Q 20
  - Coverage filter: -C 50
- **Tool**: Uses bamCaller.py script (from Stephan Schiffels)

### 5. `convert_vcfs_beds_to_mhs`
- **Input**: VCF file, BED file, mappability mask
- **Output**: Final mhs file (one per chromosome)
- **Purpose**: Generates multi-hetsep format files with masks applied
- **Masks applied**:
  - Callable regions (from BED file)
  - Mappability mask (pre-computed for GRCh38)
- **Tool**: Uses generate_multihetsep.py script (from Aylwyn Scally)

## Configuration

The Snakefile is configured to process:
- **Populations**: 26 populations from 1000 Genomes (ACB, ASW, BEB, CDX, CEU, CHB, CHS, CLM, ESN, FIN, GBR, GIH, GWD, IBS, ITU, JPT, KHV, LWK, MSL, MXL, PEL, PJL, PUR, STU, TSI, YRI)
- **Samples per population**: 4 samples
- **Chromosomes**: All autosomes (chr1-chr22)

## Required Files and Paths

### Reference Files
- **GRCh38 reference**: `/home/tc557/rds/.../GRCh38_full_analysis_set_plus_decoy_hla.fa`
- **Mappability mask**: `/home/tc557/rds/.../GRCh38_ref/strict_perchrom/`
- **Sample metadata**: Files mapping sample names to IDs for each population

### Scripts
- **bamCaller.py**: Calls variants from BAM file (Stephan Schiffels)
- **generate_multihetsep.py**: Generates mhs files from VCF (Aylwyn Scally)

### Input Data
- **CRAM files**: 1000 Genomes 30X coverage data in GRCh38 coordinates

## Output Structure

The output directory structure is:
```
/home/tc557/rds/.../1000Genomes_30X/230213/
├── {population}/
│   └── {sample}/
│       ├── bams/              # Temporary BAM files
│       ├── vcf_beds/          # VCF and BED files per chromosome
│       └── mhs/               # Final mhs files (chr1.mhs - chr22.mhs)
```

## Usage

To run the pipeline for all configured samples and chromosomes:
```bash
snakemake -s Snakefile --cores <num_cores>
```

To process specific samples or chromosomes, modify the `pop_sample` and `chroms` variables in the Snakefile.

## How This Fits in the Bigger Picture

This pipeline is the **first step** in the cobraa reproducibility workflow:

1. **This pipeline (mhsfiles/)**: Generate mhs files from raw sequencing data
2. **Next step (inference_realdata/)**: Run cobraa inference on the generated mhs files to infer population size changes and archaic admixture
3. **Alternative (simulations/)**: Run cobraa on simulated data for validation and testing

The mhs files generated here are used as input for both real data inference and serve as the standard format for cobraa analysis.

## Notes

- The pipeline uses temporary files for BAM files to save disk space
- Mean coverage is calculated dynamically for each sample using chromosome 20
- Quality filters are applied to ensure high-confidence variant calls
- Both accessibility masks (from BAM) and mappability masks (pre-computed) are applied
