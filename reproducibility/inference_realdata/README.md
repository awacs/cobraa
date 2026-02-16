# Real Data Inference Snakefile

## Overview

This Snakefile implements the inference pipeline used in the *cobraa* paper for analyzing real human genomes from the 1000 Genomes Project. It runs cobraa on mhs files to infer population size histories and archaic admixture parameters for individuals from 26 global populations.

## Purpose

This pipeline performs the main analyses presented in the cobraa publication:
1. Infer population size changes over time for modern human populations
2. Test for evidence of archaic admixture (gene flow from an unsampled population)
3. Estimate admixture times (T₁, T₂) and admixture fractions (γ)
4. Compare structured (admixture) models against panmictic (PSMC) models
5. Optionally decode the genome to identify admixture segments

## Pipeline Rules

### Inference Rules

#### 1. `thetafixed_run_cobraa_lambdaBoneval_lambdaAfree_thetafixed`

Runs cobraa with **constant N_B** (population B size constrained).

**Purpose**: Main inference model where population B is constrained to have constant size while population A can vary freely.

**Key Configuration:**
- `theta`: Fixed at 0.0008 (user-specified mutation rate)
- `lambda_B_segments`: Set to `1*D` (one parameter for all of population B)
- `lambda_A_segments`: Free (D parameters)
- `D`: 32 discrete time intervals
- `b`: 100 bp genomic bin size

**Parameters Inferred:**
- Population A size history (N_A(t))
- Population B constant size (N_B)
- Admixture fraction (γ)
- Recombination rate (ρ)

**Input**: mhs files for all autosomes (chr1-chr22) for each individual

**Output**: `final_parameters.txt` containing:
- Time boundaries for each interval
- Coalescence rates for population A
- Coalescence rates for population B

#### 2. `thetafixed_run_cobraa_lambdaBfree_lambdaAfree_thetafixed`

Runs cobraa with **variable N_B** (population B size can vary).

**Purpose**: Alternative model allowing population B to have varying size over time. Commented out in default configuration.

**Configuration**: Same as above except `lambda_B_segments` is free (D parameters)

#### 3. `thetafixed_decode_cobraa_lambdaBoneval_lambdaAfree_thetafixed`

Decodes the genome to infer local ancestry and coalescence times.

**Purpose**: After inferring parameters, decode each chromosome to:
- Identify regions deriving from population A vs. B
- Infer local coalescence times
- Generate posterior probabilities of ancestry states

**Key Features:**
- Uses **composite maximum likelihood (ML) parameters** (ts=13, te=21) - the time pair identified as having the highest composite log-likelihood from the grid search
- Runs on individual chromosomes (1 per job)
- Downsamples output every 10 sites (`-decode_downsample 10`)
- Uses `-path` flag for cobraa-path model

**Input**: 
- Single chromosome mhs file
- Inferred parameters from inference run

**Output**: Compressed decoding file per chromosome with posterior probabilities

## Configuration

### Populations and Samples

Analyzes individuals from **26 populations** (1000 Genomes):
- **African**: ACB, ASW, ESN, GWD, LWK, MSL, YRI
- **European**: CEU, FIN, GBR, IBS, TSI
- **East Asian**: CDX, CHB, CHS, JPT, KHV
- **South Asian**: BEB, GIH, ITU, PJL, STU
- **Admixed American**: CLM, MXL, PEL, PUR

**Samples per population**: 1 (first sample in each population)

### Time Pair Search

The pipeline tests multiple (ts, te) pairs to find maximum likelihood:

**Grid Search:**
- `ts` (T₁, admixture time): Range 6 to 14
- `te` (T₂, split time): Range 17 to 27
- Plus panmictic model: (None, None)

**Composite ML Pair**: (13, 21) - used for decoding

This grid search identifies the best-fitting admixture and split times for each individual.

### Model Parameters

**Fixed Parameters:**
- `D`: 32 discrete time intervals
- `b`: 100 bp genomic bin size
- `theta`: 0.0008 (scaled mutation rate)
- `spread1`: 0.075 (recent time dispersion)
- `spread2`: 50 (ancient time dispersion)
- `iterations`: 150 (EM iterations)
- `thresh`: 1 (convergence threshold)
- `mu_over_rho_ratio`: 1.5 (starting mu/rho ratio)

**Structure Constraints:**
- `lambda_upr_struct`: 10 (upper bound on lambda in structured period)
- `lambda_lwr_struct`: 0.5 (lower bound on lambda in structured period)
- `gamma_upr_bound`: 0.5 (maximum admixture fraction)

**Population B Modeling:**
- Default: Constant size (`lambda_B_segments 1*32`)
- Alternative: Variable size (free, commented out)

## Output Structure

```
/home/tc557/rds/hpc-work/PSMCplus_analysis_231026/231206/
└── D_{D}/b_{b}/spread1_{spread1}/spread2_{spread2}/
    └── muoverr_{muoverr}/iterations{iterations}/thresh_{thresh}/
        └── thetafixed_{theta}/popsample_{population}_{sample}/
            └── pair_{ts}_{te}/
                └── lambdauprstruct_{lambda_upr_struct}/lambdalwrstruct_{lambda_lwr_struct}/
                    └── gammauprbound_{gamma_upr_bound}/
                        ├── lambdaBonevalue_lambdaAfree/
                        │   ├── final_parameters.txt      # Inferred parameters
                        │   ├── log.txt                   # Inference log
                        │   └── decoding_compositeLL240118/
                        │       └── chr{1-22}.txt.gz      # Decoded chromosomes
                        └── lambdaBfree_lambdaAfree/
                            ├── final_parameters.txt      # Alternative model
                            └── log.txt
```

## Usage

### Run Inference for All Populations

```bash
# Run full inference pipeline
snakemake -s Snakefile --cores <num_cores>

# Run inference only (no decoding)
snakemake -s Snakefile thetafixed_run_cobraa_lambdaBoneval_lambdaAfree_thetafixed --cores <num_cores>

# Run decoding only
snakemake -s Snakefile thetafixed_decode_cobraa_lambdaBoneval_lambdaAfree_thetafixed --cores <num_cores>
```

### Run for Specific Population

Modify the `pop_and_sample` list in the Snakefile or use Snakemake's filtering:

```bash
snakemake -s Snakefile --cores <num_cores> -- <specific_output_file>
```

## Required Input Files

### MHS Files

Expects mhs files generated by the mhsfiles/ pipeline:
- Location: `/home/tc557/rds/.../1000Genomes_30X/230213/{pop}/{sample}/mhs/chr{1-22}.mhs`
- Format: Multi-hetsep format
- Coverage: All autosomes

### Sample Metadata

Sample name to ID mapping files for each population:
- Location: `/home/tc557/rds/.../231103_samples_name_ID_{pop}.txt`
- Format: `{sample_name} {sample_ID}` per line

## Key Functions

### `split_popsam(wildcards)`
Parses the combined population_sample identifier into separate population and sample names.

### `get_mhs_files_allchroms(wildcards)`
Retrieves paths to all 22 autosomal mhs files for a given sample.

### `get_params_from_ouput_file_240106(wildcards)`
Extracts inferred parameters (θ, ρ, γ, λ_A, λ_B) from a completed inference run for use in decoding.

## How This Fits in the Bigger Picture

This pipeline represents the **main analysis** of the cobraa project:

**Workflow:**
1. **Data Preparation (mhsfiles/)**: Convert CRAM → BAM → VCF → mhs
2. **This Pipeline (inference_realdata/)**: Analyze real human genomes
   - Infer demographic parameters
   - Compare structured vs. panmictic models
   - Identify admixture signals
   - Decode ancestry along chromosomes
3. **Validation (simulations/)**: Test method accuracy using simulated data

**Scientific Questions Addressed:**
- Is there evidence for archaic admixture in modern humans?
- When did admixture events occur (T₁, T₂)?
- What fraction of ancestry derives from the archaic population (γ)?
- How do population size histories vary across populations?
- Which regions of the genome show archaic ancestry?

## Analysis Strategy

### Model Comparison

For each individual, the pipeline:
1. Tests panmictic model (ts=None, te=None) - PSMC-like
2. Tests structured models across grid of (ts, te) pairs
3. Compares log-likelihoods to identify best model
4. If structured model fits better, estimates admixture parameters

### Parameter Constraints

**Why constrain N_B to be constant?**
- Reduces parameter space for better identifiability
- Archaic population size is difficult to infer from limited introgressed segments
- Conservative approach focusing on well-constrained parameters

**Why search over time pairs?**
- Admixture and split times are not known a priori
- Grid search identifies maximum likelihood values
- Composite likelihood approach combines evidence across genome

## Computational Considerations

The pipeline is designed for HPC environments:
- Each inference job processes ~3 billion bp (22 chromosomes)
- Memory requirements moderate due to binning (b=100)
- Parallelizable across individuals and time pairs
- Decoding can be parallelized across chromosomes

## Output Interpretation

### final_parameters.txt Format

Contains D rows × 4 columns:
1. Left time boundary (coalescent units)
2. Right time boundary (coalescent units)
3. Coalescence rate in population A (λ_A)
4. Coalescence rate in population B (λ_B)

**Conversion to demographic parameters:**
- Time in generations: boundary / μ
- Effective population size: (1 / coalescence_rate) / μ
- Where μ = mutation rate per generation per base

### Log File Contents

Contains iteration-by-iteration updates:
- Log-likelihood values
- Updated parameters (λ_A, λ_B, γ, θ, ρ)
- Convergence information

## Notes

- The pipeline uses fixed theta (0.0008) rather than inferring from data
- Recombination rate (rho) is inferred starting from mu_over_rho_ratio=1.5
- All individuals analyzed independently
- Time boundaries are ~logarithmically spaced via spread1/spread2 parameters
- Decoding uses composite ML pair (13,21) identified from grid search
