# Quick Start Guide - Configuration Required

## Important Note

**The hard-coded paths from the original repository won't work for you** since they were specific to another user's system (`/home/tc557/...`). You **MUST** configure the `config.yaml` files before running any workflows.

## Setup Steps

### 1. Navigate to a workflow directory

```bash
cd reproducibility/mhsfiles/  # or simulations/ or inference_realdata/
```

### 2. Edit config.yaml

Open `config.yaml` in your text editor:

```bash
nano config.yaml  # or vim, emacs, etc.
```

### 3. Replace ALL placeholder paths

**CRITICAL:** Replace every line that starts with `EDIT_ME/` with your actual paths:

```yaml
# BEFORE (won't work):
cobraa_script: "EDIT_ME/path/to/cobraa/cobraa.py"

# AFTER (replace with your actual path):
cobraa_script: "/home/yourname/software/cobraa/cobraa.py"
```

### 4. Save and verify

After editing all paths, try a dry-run to verify configuration:

```bash
snakemake -s Snakefile --dry-run
```

If you see errors about `EDIT_ME/` or `/path/to/`, you haven't finished editing the config file.

## What Each Config File Needs

### mhsfiles/config.yaml
- `reference_genome` - Path to GRCh38 reference FASTA
- `scripts_dir` - Path to cobraa/reproducibility/mhsfiles
- `mappability_mask_dir` - Path to mappability masks
- `output_dir` - Where to write output files
- Plus several other paths for input data

### simulations/config.yaml
- `cobraa_script` - Path to cobraa.py
- `simulation_script` - Path to msprime_simulations.py
- `data_dir` - Where to write simulation outputs

### inference_realdata/config.yaml
- `cobraa_script` - Path to cobraa.py
- `mhs_files_dir` - Where your MHS files are located
- `data_dir` - Where to write inference outputs
- Plus paths for metadata and input data

## Error Messages

If you try to run without configuring, you'll see:

```
ERROR: Configuration 'cobraa_script' still has placeholder value: EDIT_ME/path/to/cobraa/cobraa.py
       Please edit config.yaml and set the actual path for cobraa.py script path
```

This means you need to edit that specific path in config.yaml.

## Why This Changed

The original Snakefiles had hard-coded paths like:
```python
cobraa_script = '/home/tc557/cobraa/cobraa.py'
```

These paths don't exist on your system, so they would fail anyway. The new system **requires** you to configure your own paths, making it clear what needs to be set up.

## Need Help?

- See `README_CONFIG.md` for detailed documentation
- See `EXAMPLES.md` for example configurations
- See `MIGRATION_GUIDE.md` if migrating from old hard-coded paths
