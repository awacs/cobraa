# Simulations Snakefile

## Overview

This Snakefile implements a comprehensive simulation and inference workflow for testing and validating the *cobraa* method. It simulates genetic data under various demographic models with population structure and admixture, then runs inference to evaluate the method's performance in recovering the true parameters.

## Purpose

This pipeline serves two critical functions:
1. **Simulation**: Generate synthetic genetic data under controlled demographic scenarios using msprime
2. **Inference Validation**: Test cobraa's ability to recover known demographic parameters (population sizes, admixture times, admixture fractions)

This is essential for:
- Validating the cobraa method's accuracy
- Understanding parameter identifiability
- Exploring the method's behavior under different demographic scenarios
- Comparing structured (cobraa) vs. unstructured (PSMC) models

## Pipeline Components

### Simulation Rule: `simulate`

Simulates genetic data using msprime with specified demographic parameters.

**Key Parameters:**
- `L`: Sequence length (default: 1.5e8 bp)
- `N`: Effective population size (default: 1.6e4)
- `mu`: Mutation rate per base per generation (default: 1.25e-8)
- `p`: Recombination rate per base (default: 1e-8)
- `D`: Number of discrete time intervals (default: 32)
- `ts`: Time index of admixture event (T₁)
- `te`: Time index of population split (T₂)
- `gamma`: Admixture fraction from population B
- `intensity`: Additional parameter for admixture intensity
- `sim_model`: Simulation model (default: 'hudson')

**Demographic Models:**
- `pop_split_psc_pop_240604`: Panmictic model (no structure)
- `pop_split_psc_pop_240604_true`: Structured model with admixture

**Output:**
- Compressed mhs files (`.mhs.gz`) containing simulated variants
- One file per chromosome (20 chromosomes simulated)
- One set per sample (configurable number of samples)

### Inference Rules

#### 1. `PSMC_inference_PSMC`

Runs PSMC (unstructured/panmictic) inference on simulated data.

**Purpose**: Provides a baseline comparison by fitting a standard PSMC model (no admixture) to the data.

**Key Parameters:**
- `binsize`: Genomic bin size (default: 100 bp)
- `thresh`: Convergence threshold
- `iterations`: Number of EM iterations (default: 100)
- `theta`: Fixed at 0.0008

**Output**: `final_parameters.txt` with inferred population size history

#### 2. `cobraa_inference_cobraa`

Runs cobraa (structured) inference on simulated data using the **true** admixture times.

**Purpose**: Tests cobraa's ability to recover population sizes and admixture fraction when ts and te are known.

**Key Parameters:**
- Same as PSMC, plus:
- `gamma_fg`: First guess for admixture fraction
- `lambda_B_segments`: Set to `32*0` (constrains N_B to be constant)
- `gamma_upr`: Upper bound on gamma (0.5)

**Dependencies**: Requires PSMC inference to complete first

**Output**: `final_parameters.txt` with inferred population sizes and admixture parameters

#### 3. `alltimepairs_cobraa_inference_cobraa_alltimepairs`

Runs cobraa inference searching over **all possible time pairs** (ts, te).

**Purpose**: Tests the method's ability to identify the correct admixture and split times from data.

**Search Space:**
- `ts` range: 1 to 30
- `te` range: ts+1 to 31
- All possible combinations are tested

**Key Parameters:**
- `infts`, `infte`: Inferred time indices being tested
- Other parameters same as structured cobraa

**Output**: Separate inference results for each (ts, te) pair tested

## Simulation Scenarios

### Default Configuration

The Snakefile is configured to test:

**Demographic Parameters:**
- Admixture fractions (`gamma`): 0.2
- Time pairs (`ts_te_sim_pairs`): (13, 21)
- Number of samples: 1

**Search Configuration:**
- Tests all possible (ts, te) pairs from (1,2) to (30,31)
- This creates a grid search to find the maximum likelihood time pair

### Historical Configurations (Commented Out)

The file includes configurations for broader parameter sweeps:
- Multiple gamma values: 0.05, 0.1, 0.2, 0.3, 0.4
- Multiple time pairs: (6,13), (6,15), (9,16), (9,19), (13,18), (13,21), (13,23)
- Multiple samples: up to 10
- Multiple gamma first guesses: 0.05, 0.2, 0.4

## Output Structure

```
/home/tc557/rds/hpc-work/cobraa_snakemakes/cobraa_simulations_and_inference_240604/
└── model_{model}/
    └── L_{L}/N_{N}/mu_{mu}/p_{p}/D_{D}/
        └── ts_{ts}/te_{te}/intensity_{intensity}/gamma_{gamma}/
            └── simmodel_{sim_model}/spread1_{spread1}/spread2_{spread2}/
                ├── sample{sample}_chrom{chrom}.mhs.gz  # Simulated data
                └── inference_240607/
                    ├── unstructure/                     # PSMC results
                    │   └── binsize{b}_thresh{t}_iterations{i}_fixedtheta0.0008_sample{s}_final_parameters.txt
                    └── structure/                       # Cobraa results
                        ├── binsize{b}_thresh{t}_iterations{i}_gammafg{g}_fixedtheta0.0008_sample{s}_final_parameters.txt
                        └── inferts{ts}_infte{te}/       # Time pair search
                            └── binsize{b}_thresh{t}_iterations{i}_gammafg{g}_fixedtheta0.0008_sample{s}_final_parameters.txt
```

## Usage

To run simulations and inference:

```bash
# Simulate data only
snakemake -s Snakefile simulate --cores <num_cores>

# Run full pipeline (simulate + infer)
snakemake -s Snakefile --cores <num_cores>

# Run specific inference
snakemake -s Snakefile cobraa_inference_cobraa --cores <num_cores>
```

## How This Fits in the Bigger Picture

This pipeline is used for **method validation and development**:

1. **Simulations (this pipeline)**: 
   - Generate data with known demographic history
   - Test inference under controlled conditions
   - Validate parameter recovery
   - Compare structured vs. unstructured models

2. **Real Data Analysis (inference_realdata/)**:
   - Apply validated method to actual human genomes
   - Infer unknown demographic parameters
   - Requires mhs files from mhsfiles/ pipeline

3. **Method Development**:
   - Results from this pipeline informed parameter choices
   - Helped establish convergence criteria
   - Validated the statistical framework

## Key Insights from Simulations

The simulation framework tests:
- **Identifiability**: Can we distinguish admixture from other demographic events?
- **Time resolution**: How precisely can we identify ts and te?
- **Admixture fraction recovery**: How accurately can we estimate gamma?
- **Model comparison**: When does structured model outperform panmictic model?

## Computational Resources

The Snakefile includes resource specifications for HPC:
- Partition: icelake-himem
- Memory: 200-355G depending on task
- Time: 2 hours per task
- Account: DURBIN-SL2-CPU

These can be modified for different computing environments.

## Notes

- The pipeline uses msprime for coalescent simulations
- Simulations use the SMC' approximation (appropriate for PSMC/cobraa)
- Both panmictic and structured models are simulated with identical coalescence rates
- The `spread1` and `spread2` parameters control time interval spacing (same as in the main cobraa inference pipeline)
- Multiple samples allow assessment of variance in parameter estimates
