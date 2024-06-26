---
title: "Siletti 2022 snRNAseq Dataset - Get Specificity Matrix"
author: "Tayden Li"
output:
  html_document:
    keep_md: yes
---

Code is adapted from [Bryois (2020)](https://github.com/jbryois/scRNA_disease/tree/master).
The script produces continuous specificity MAGMA inputs.

# Load Data

The Siletti data was downloaded from [here](https://github.com/linnarsson-lab/adult-human-brain).

### Load necessary libraries


```r
library(tidyverse)
library("rhdf5")
```

### Load single cell dataset


```r
file="/oak/stanford/groups/laramied/LDSC/Bryois2020/Code_Paper/Data/Siletti/Siletti_L2-cluster-log1p_matrix.h5"
h5f <- H5Fopen(file)
exp <- as.data.frame(t(h5f$matrix))
colnames(exp) <- h5f$Cluster
exp$Gene <- sub("\\..*", "", h5f$Accession)
h5closeAll()
```

Keep certain clusters only (if needed).


```r
exp <- exp %>% select(Cluster83:Cluster205, Gene)
```


Only keep genes with a unique name and tidy data.


```r
exp <- exp %>% add_count(Gene) %>% 
  filter(n==1) %>%
  select(-n) %>%
  gather(key = column,value=Expr,-Gene) %>%
  as.tibble()
```

### Load gene coordinates

File downloaded from MAGMA website (https://ctg.cncr.nl/software/magma).

Filtered to remove extended MHC (chromosome 6, 25Mb to 34Mb).


```r
gene_coordinates <- 
  read_tsv("../Data/NCBI/NCBI37.3.gene.loc.extendedMHCexcluded",
           col_names = FALSE,col_types = 'cciicc') %>%
  select(1:4) %>% 
  rename(end="X4", start="X3", chr="X2", ENTREZ="X1") %>% 
  mutate(chr=paste0("chr",chr))
```


### Get table for ENTREZ and ENSEMBL gene names.


```r
entrez_ensembl <- AnnotationDbi::toTable(org.Hs.eg.db::org.Hs.egENSEMBL)
```

Only keep genes with a unique ENTREZ and ENSEMBL ID.


```r
entrez_ensembl_unique_genes_entrez <- entrez_ensembl %>% count(gene_id) %>% filter(n==1)
entrez_ensembl_unique_genes_ens <- entrez_ensembl %>% count(ensembl_id) %>% filter(n==1)
entrez_ensembl <- filter(entrez_ensembl,gene_id%in%entrez_ensembl_unique_genes_entrez$gene_id & ensembl_id %in% entrez_ensembl_unique_genes_ens$ensembl_id)
colnames(entrez_ensembl)[1] <- "ENTREZ"
colnames(entrez_ensembl)[2] <- "Gene"
gene_coordinates <- inner_join(entrez_ensembl,gene_coordinates) %>% as.tibble()
```


# Transform Data

### Remove genes not expressed


```r
exp_agg <- exp %>% rename(ClusterID=column, Expr_sum_mean=Expr)

not_expressed <- exp_agg %>% 
  group_by(Gene) %>% 
  summarise(total_sum=sum(Expr_sum_mean)) %>% 
  filter(total_sum==0) %>% 
  select(Gene) %>% unique() 

exp_agg <- filter(exp_agg,!Gene%in%not_expressed$Gene)
```

### Each cell type is scaled to the same total number of (transformed) molecules. 


```r
exp_agg <- exp_agg %>% 
  group_by(ClusterID) %>% 
  mutate(Expr_sum_mean=Expr_sum_mean*1000/sum(Expr_sum_mean)) %>% 
  ungroup()
```


# Specificity Calculation

Specificity is defined as the proportion of total expression in a cell type of interest (x/sum(x)).

### Calculate specificity


```r
exp_agg <- exp_agg %>% 
  group_by(Gene) %>% 
  mutate(specificity=Expr_sum_mean/sum(Expr_sum_mean)) %>% 
  ungroup()
```

### Get MAGMA genes

Only keep genes in the MAGMA gene coordination list


```r
exp_agg <- inner_join(exp_agg,gene_coordinates,by="Gene")
```

### Write MAGMA input file


```r
exp_conti_spe <- exp_agg %>% select(ENTREZ, ClusterID, specificity) %>% spread(ClusterID, specificity)
colnames(exp_conti_spe)[1] <- "GENE"

exp_conti_spe %>% write_tsv("MAGMA/conti_specificity_matrix.txt")
```
