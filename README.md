# Mapping the Cellular Etiology of Schizophrenia and Diverse Brain Phenotypes
This is the code for the following publication:
Laramie E Duncan*, Tayden Li*, Madeleine Salem, Will Li, Leili Mortazavi, Hazal Senturk, Naghmeh Shargh, Sam Vesuna, Hanyang Shen, Jong Yoon, Gordon Wang, Jacob Ballon, Longzhi Tan, Brandon Scott Pruett, Brian Knutson, Karl Deisseroth, William J Giardino. Mapping the Cellular Etiology of Schizophrenia and Diverse Brain Phenotypes (in revision).


## Software Requirements
1. Python libraries: `h5py`, `numexpr`
2. R libraries: `tidyverse`, `rhdf5`, `AnnotationDbi`, `org.Hs.eg.db`, `dplyr`, `readr` 
3. [MAGMA v1.10](https://cncr.nl/research/magma/)

## Data Requirements
1. GWAS summary statistics
2. [Siletti et al.'s single-cell RNAseq dataset](https://github.com/linnarsson-lab/adult-human-brain)
3. [MAGMA auxiliary files](https://cncr.nl/research/magma/) of the same genome build and ancestry as GWAS summary statistics to be used

## Get MAGMA Inputs
Follow the following steps:
1. [Get the ln(1+x)-transformed cluster-by-gene matrix.](Preprocessing_Siletti/create_matrices/Siletti_create_L2-log_dataset.py)
2. [Preprocess the matrix and calculate specificity.](Preprocessing_Siletti/create_magma_inputs/get_Siletti_continuous_input.md)

## Run MAGMA
First, confirm the following items:
1. The summary statistics are from a single population that matches MAGMA's auxiliary data.
2. The summary statistics are the same genome build as MAGMA's auxiliary files.
3. If the summary statistics do not contain a SNP ID column, obtain the SNP IDs from the chromosomal and base pair positions using a reference file of the same genome build.

Then, follow the steps below to run MAGMA (scripts to be modified accordingly):
1. Create a SNP location file (`snploc_{GWAS_file_name}`) that contains three columns of the GWAS summary statistics in the following order: SNP ID, chromosome, and base pair position.
2. Annotate and conduct a gene analysis.
     Example code is provided [here](MAGMA/1.annotationAndGeneAnalysis.sh). The annotation step requires SNP location files created earlier, while the gene analysis step requires original GWAS files. Please refer to the MAGMA manual for different specification options for sample size and more.
4. Run a gene property analysis.
     Example code is provided [here](MAGMA/2.genePropertyAnalysis.sh). Please also refer to the MAGMA manual for the usage of different flag options.

### Conditional Analysis
To run a pairwise conditional analysis on clusters after the steps above, follow these steps (scripts to be modified accordingly):
1. To limit the computation time, [create a MAGMA input file with only top clusters](MAGMA/3.create_top_results_matrix.md), as indicated by the results from the previous step.
2. [Run a pairwise conditional analysis.](MAGMA/4.conditionalAnalysis.sh)
3. [Conduct a forward stepwise selection](MAGMA/5.forward_selection_condition_results.md) to arrive at a list of independent clusters.
