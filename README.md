# Mapping the Cellular Neuropathology of Psychiatric Disorders

# Requirements
1. Python libraries: `h5py`, `numexpr`
2. R libraries: `tidyverse`, `rhdf5`, `AnnotationDbi`, `org.Hs.eg.db`
3. [MAGMA](https://ctg.cncr.nl/software/magma)

# Data Requirements
1. The [Siletti single-cell RNAseq dataset](https://github.com/linnarsson-lab/adult-human-brain)
2. [MAGMA](https://ctg.cncr.nl/software/magma) auxillary files

# Get MAGMA Inputs
Follow the follwoing steps:
1. [Get a $\ln(1+x)$-transformed Cluster $\times$ Gene matrix.](Preprocessing_Siletti/create_matrices/Siletti_create_L2-log_dataset.py)
2. [Get continous specificity MAGMA input files.](Preprocessing_Siletti/create_magma_inputs/get_Siletti_continuous_input.md)

# Run MAGMA
First, confirm the following items:
1. The summary statistics is from a single population which matches MAGMA's auxillary data.
2. The summary statistics is of the same genome build as MAGMA's auxillary files.
3. If the summary statistics does not contain a SNP ID column, infer the SNP IDs from the chromosomal and base pair positions using a reference file of the same genome build. See [example](MAGMA/0.get_rsid.Rmd).

Then, follow the steps below to run MAGMA (scripts to be modified accordingly):
1. [Annotate and conduct gene analysis.](MAGMA/1.step1and2.sh)
2. [Run MAGMA's continous gene property analysis](MAGMA/2.step3-conti.sh)

## Conditional Analysis
To run pairwise conditional analysis on clusters after the steps above, follow these steps (scipts to be modified accordingly):
1. To limit the computational time, [create a MAGMA input file with only top clusters](MAGMA/3.create_top_results_matrix.Rmd), as indicated by the results from gene property analysis.
2. [Run the pairwise conditional analysis.](MAGMA/4.step3-joint_conti.sh)
3. [Forward selection](5.forward_selection_condition_results.Rmd) to arrive at a list of relatively conditionally independent clusters.