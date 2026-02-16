# Example Configuration Files

This directory contains example configuration files showing realistic path structures. These are templates you can copy and modify for your system.

## Example 1: University HPC System

```yaml
# reproducibility/mhsfiles/config.yaml
# Example for a university HPC cluster

reference_genome: "/cluster/project/genetics/references/GRCh38/GRCh38_full_analysis_set_plus_decoy_hla.fa"
scripts_dir: "/home/username/software/cobraa/reproducibility/mhsfiles"
mappability_mask_dir: "/cluster/project/genetics/references/GRCh38/mappability_masks/strict_perchrom"
meancoverage_file: "/scratch/username/1000genomes/meancoverage.txt"
data_dir: "/scratch/username/projects"
genomes_data_dir: "/cluster/project/genetics/1000Genomes/phase3_30X"
cram_files_dir: "/cluster/project/genetics/1000Genomes/phase3_30X/crumble/GRCh38"
sample_metadata_dir: "/cluster/project/genetics/1000Genomes/metadata"
output_dir: "/scratch/username/output/1000genomes_mhs"
```

## Example 2: Local Workstation

```yaml
# reproducibility/simulations/config.yaml
# Example for a local workstation

cobraa_script: "/home/username/software/cobraa/cobraa.py"
simulation_script: "/home/username/software/cobraa/reproducibility/simulations/msprime_simulations.py"
data_dir: "/mnt/data/simulations/cobraa_output"
```

## Example 3: Cloud Instance (AWS)

```yaml
# reproducibility/inference_realdata/config.yaml
# Example for an AWS EC2 instance

cobraa_script: "/opt/cobraa/cobraa.py"
genomes_data_dir: "/data/1000genomes"
mhs_files_dir: "/data/1000genomes/mhs_files"
sample_metadata_dir: "/data/1000genomes/metadata"
data_dir: "/results/inference_analysis"
```

## Path Naming Conventions

When setting up your paths, consider these conventions:

### Reference Data (Read-Only)
- Store in a shared, read-only location
- Examples: `/cluster/shared/references/`, `/data/references/`, `/opt/references/`

### Input Data (Read-Only)
- Store in a project-specific or shared location
- Examples: `/cluster/project/myproject/input/`, `/data/1000genomes/`

### Output Data (Read-Write)
- Use scratch or fast storage for outputs
- Examples: `/scratch/username/`, `/results/`, `/mnt/fast_storage/output/`

### Software/Scripts
- Install in home directory or shared software location
- Examples: `/home/username/software/`, `/opt/`, `/usr/local/`

## Quick Setup Script

Here's a bash script to help set up your config file:

```bash
#!/bin/bash
# setup_config.sh - Interactive config setup

echo "Setting up cobraa configuration..."
echo

# Get base paths
read -p "Enter your cobraa installation directory: " COBRAA_DIR
read -p "Enter your data directory: " DATA_DIR
read -p "Enter your reference genome path: " REF_GENOME

# Create config for mhsfiles
cat > reproducibility/mhsfiles/config.yaml << EOF
reference_genome: "${REF_GENOME}"
scripts_dir: "${COBRAA_DIR}/reproducibility/mhsfiles"
mappability_mask_dir: "${DATA_DIR}/references/mappability"
meancoverage_file: "${DATA_DIR}/meancoverage.txt"
data_dir: "${DATA_DIR}"
genomes_data_dir: "${DATA_DIR}/1000genomes"
cram_files_dir: "${DATA_DIR}/1000genomes/cram"
sample_metadata_dir: "${DATA_DIR}/1000genomes/metadata"
output_dir: "${DATA_DIR}/output"
EOF

echo "Config file created at reproducibility/mhsfiles/config.yaml"
```

## Validation

After creating your config, validate it with:

```bash
# Check YAML syntax
python -c "import yaml; yaml.safe_load(open('config.yaml'))"

# Check if paths exist
for path in $(grep ': "/' config.yaml | cut -d'"' -f2); do
  if [ ! -e "$path" ]; then
    echo "Warning: Path does not exist: $path"
  fi
done
```

## Tips for Different Environments

### HPC Clusters
- Use `$SCRATCH` or `$WORK` environment variables if available
- Consider using Lmod or modules for software paths
- Check storage quotas before setting output directories

### Cloud Instances
- Use mounted volumes for data storage
- Consider using S3 buckets for reference data
- Use instance storage for temporary/scratch files

### Local Workstations
- Use absolute paths (avoid `~` for home directory)
- Consider using SSDs for output directories
- Monitor disk space when running large workflows

## See Also

- `README_CONFIG.md` - Detailed configuration documentation
- `MIGRATION_GUIDE.md` - Migration guide from hard-coded paths
- Each `config.yaml` - Template configuration files
