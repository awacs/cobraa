# Snakemake Configuration Guide

This directory contains Snakefiles for processing genomic data with *cobraa*. The Snakefiles have been updated to use configuration files instead of hard-coded paths, making them portable across different systems.

## Quick Start

1. Copy the `config.yaml` file in each reproducibility subdirectory
2. Edit the paths in `config.yaml` to match your local system
3. Run Snakemake as usual: `snakemake -s Snakefile --cores <N>`

## Configuration Files

Each subdirectory (`mhsfiles/`, `simulations/`, `inference_realdata/`) contains its own `config.yaml` file.

### reproducibility/mhsfiles/config.yaml

This configuration is for processing BAM/CRAM files into multi-hetsep (mhs) format files:

```yaml
# Reference genome for GRCh38 (hg38)
reference_genome: "/path/to/GRCh38_full_analysis_set_plus_decoy_hla.fa"

# Scripts directory (change to your local cobraa installation)
scripts_dir: "/path/to/cobraa/reproducibility/mhsfiles"

# Mappability mask directory for GRCh38
mappability_mask_dir: "/path/to/GRCh38_ref/strict_perchrom"

# Mean coverage file
meancoverage_file: "/path/to/meancoverage.txt"

# Base data directory for storing outputs
data_dir: "/path/to/data"

# 1000 Genomes data directory
genomes_data_dir: "/path/to/1000Genomes_30X"

# CRAM files directory
cram_files_dir: "/path/to/1000Genomes_30X/crumble/GRCh38"

# Sample metadata directory
sample_metadata_dir: "/path/to/1000Genomes_30X/sample_metadata"

# Output directory for processing
output_dir: "/path/to/output/230213"
```

### reproducibility/simulations/config.yaml

This configuration is for running simulations:

```yaml
# Scripts directory (change to your local cobraa installation)
cobraa_script: "/path/to/cobraa/cobraa.py"

# Simulation script
simulation_script: "/path/to/cobraa/reproducibility/simulations/msprime_simulations.py"

# Base directory for simulation outputs
data_dir: "/path/to/cobraa_simulations_and_inference"
```

### reproducibility/inference_realdata/config.yaml

This configuration is for running inference on real data:

```yaml
# Scripts directory (change to your local cobraa installation)
cobraa_script: "/path/to/cobraa/cobraa.py"

# 1000 Genomes data directory
genomes_data_dir: "/path/to/1000Genomes_30X"

# MHS files base directory
mhs_files_dir: "/path/to/1000Genomes_30X/mhs_files"

# Sample metadata directory
sample_metadata_dir: "/path/to/1000Genomes_30X/sample_metadata"

# Base directory for inference outputs
data_dir: "/path/to/PSMCplus_analysis"
```

## Backward Compatibility

The Snakefiles maintain backward compatibility with the original hard-coded paths. If a path is not found in the config file, it will fall back to the original hard-coded value. This allows gradual migration to the new configuration system.

## Example Usage

### 1. Processing mhs files from BAM/CRAM

```bash
cd reproducibility/mhsfiles/
# Edit config.yaml with your paths
snakemake -s Snakefile --cores 8
```

### 2. Running simulations

```bash
cd reproducibility/simulations/
# Edit config.yaml with your paths
snakemake -s Snakefile --cores 20
```

### 3. Running inference on real data

```bash
cd reproducibility/inference_realdata/
# Edit config.yaml with your paths
snakemake -s Snakefile --cores 20
```

## Required External Data

Before running the workflows, you need to download:

1. **Reference genome (GRCh38)**: Download from NCBI or similar source
2. **Mappability masks**: Required for filtering genomic regions
3. **Sample data**: 1000 Genomes Project data or your own sequencing data

## Important Notes

- All paths in `config.yaml` should be absolute paths
- Ensure all directories in the paths exist or have write permissions
- The scripts referenced (bamCaller.py, generate_multihetsep.py, msprime_simulations.py) should be present in your cobraa installation
- Some workflows require significant computational resources (see `resources` sections in Snakefiles)

## Troubleshooting

If you encounter path-related errors:

1. Check that all paths in `config.yaml` are correct and absolute
2. Verify that you have read/write permissions for all directories
3. Ensure that required input files exist at the specified locations
4. Check that the cobraa scripts are in the paths specified in the config

## Migration from Hard-coded Paths

If you're migrating from the original hard-coded version:

1. The Snakefiles will work without config files (using original hard-coded paths)
2. Create `config.yaml` files gradually
3. Test each workflow after adding configuration
4. Once all paths are configured, the workflows become fully portable
