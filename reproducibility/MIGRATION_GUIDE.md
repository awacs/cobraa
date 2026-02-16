# Migration Guide: From Hard-coded Paths to Configurable Paths

This document provides a guide for users migrating from the old hard-coded paths to the new configurable system.

## What Changed?

The Snakefiles in the `reproducibility/` directory have been updated to use configurable paths instead of hard-coded ones. This makes the workflows portable across different systems.

## Before (Hard-coded)

```python
# Old way - hard-coded paths
reference_HGDP = '/home/tc557/rds/rds-durbin-group-8b3VcZwY7rY/projects/human/HGDP/downloaded_220412/GRCh38_full_analysis_set_plus_decoy_hla.fa'
```

## After (Configurable)

```python
# New way - configurable with fallback
configfile: "config.yaml"
reference_HGDP = config.get('reference_genome', '/home/tc557/rds/rds-durbin-group-8b3VcZwY7rY/projects/human/HGDP/downloaded_220412/GRCh38_full_analysis_set_plus_decoy_hla.fa')
```

## Migration Steps

### Option 1: Quick Start (Use Defaults)
If you have the same directory structure as the original setup, you don't need to do anything! The Snakefiles will continue to work with the original hard-coded paths as fallback values.

### Option 2: Configure for Your System

1. **Navigate to the workflow directory**
   ```bash
   cd reproducibility/mhsfiles/  # or simulations/ or inference_realdata/
   ```

2. **Edit config.yaml**
   ```bash
   nano config.yaml  # or use your preferred editor
   ```

3. **Update paths to match your system**
   ```yaml
   # Example for mhsfiles/config.yaml
   reference_genome: "/your/path/to/GRCh38_reference.fa"
   scripts_dir: "/your/path/to/cobraa/reproducibility/mhsfiles"
   output_dir: "/your/output/directory"
   # ... etc
   ```

4. **Run your workflow**
   ```bash
   snakemake -s Snakefile --cores 8
   ```

## Configuration Files

Each subdirectory has its own `config.yaml`:

- `reproducibility/mhsfiles/config.yaml` - For BAM/CRAM to mhs conversion
- `reproducibility/simulations/config.yaml` - For running simulations
- `reproducibility/inference_realdata/config.yaml` - For inference on real data

## Tips

1. **Use absolute paths** - All paths in config.yaml should be absolute paths
2. **Check permissions** - Ensure you have read/write access to all configured directories
3. **Test with dry-run** - Before running the full workflow, test with:
   ```bash
   snakemake -s Snakefile --dry-run
   ```

## Common Issues

### Issue: "No such file or directory"
**Solution**: Check that all paths in your config.yaml exist and are correctly spelled.

### Issue: "Permission denied"
**Solution**: Ensure you have write permissions for output directories.

### Issue: Workflow uses old hard-coded paths
**Solution**: Make sure `config.yaml` exists in the same directory as the Snakefile and contains the correct keys.

## Example: Complete Setup

Here's a complete example for setting up the mhsfiles workflow:

1. **Clone the repository**
   ```bash
   git clone https://github.com/awacs/cobraa.git
   cd cobraa
   ```

2. **Edit the config**
   ```bash
   cd reproducibility/mhsfiles
   nano config.yaml
   ```

3. **Update with your paths**
   ```yaml
   reference_genome: "/data/references/GRCh38.fa"
   scripts_dir: "/home/username/cobraa/reproducibility/mhsfiles"
   mappability_mask_dir: "/data/masks/GRCh38_strict"
   output_dir: "/data/output/1000genomes"
   cram_files_dir: "/data/1000genomes/cram"
   sample_metadata_dir: "/data/1000genomes/metadata"
   ```

4. **Run the workflow**
   ```bash
   snakemake -s Snakefile --cores 16
   ```

## Need Help?

- See `reproducibility/README_CONFIG.md` for detailed documentation
- Check the Snakefile comments for information about each path
- Original hard-coded paths are preserved as fallback values in the code

## Backward Compatibility

The old workflows will continue to work without any changes. The config files are optional and provide a cleaner way to manage paths across different systems.
