
Documentation for PRS-CSx
=========================

# Module Overview


PRScsx is a tool used to calculate polygenic scores. It integrates GWAS summary statistics and LD panels across multiple ancestry groups to improve global polygenic prediction. This nextflow pipeline seemlessly takes GWAS summary statistics, makes the input files for PRScsx, runs PRScsx, concatenates the chromosome-separated outcomes, and produces a summary violin plot, all in a parallelized fashion.

[Paper Link for Reference](https://www.nature.com/articles/s41588-022-01054-7)

[Tool Documentation Link](https://github.com/getian107/PRScsx/tree/master)

[Example Module Config File](https://github.com/PMBB-Informatics-and-Genomics/pmbb-geno-pheno-toolkit/tree/main/Example_Configs/prscsx.config)

[Example nextflow.config File](https://github.com/PMBB-Informatics-and-Genomics/pmbb-geno-pheno-toolkit/tree/main/Example_Configs/nextflow.config)
## Cloning Github Repository:


* Command: `git clone https://github.com/PMBB-Informatics-and-Genomics/geno_pheno_workbench.git`

* Navigate to relevant workflow directory run commands (our pipelines assume all of the nextflow files/scripts are in the current working directory)
## Software Requirements:


* [Nextflow version 23.04.1.5866](https://www.nextflow.io/docs/latest/cli.html)

* [Singularity version 3.8.3](https://sylabs.io/docs/) OR [Docker version 4.30.0](https://docs.docker.com/)
## Commands for Running the Workflow


* Singularity Command: `singularity build prscsx.sif docker://pennbiobank/prscsx:latest`

* Docker Command: `docker pull pennbiobank/prscsx:latest`

* Command to Pull from Google Container Registry: `docker pull gcr.io/verma-pmbb-codeworks-psom-bf87/prscsx:latest`

* Run Command: `nextflow run /path/to/toolkit/module/prscsx.nf`

* Common `nextflow run` flags:

    * `-resume` flag picks up the workflow where it left off, otherwise, the workflow will rerun from the beginning

    * `-stub` performs a sort of dry run of the whole workflow, checks channels without executing any code

    * `-profile` selects the compute profiles we set up in nextflow.config (see nextflow.config file below)

    * `-profile` selects the compute profiles we set up in nextflow.config (see nextflow.config file below)

    * `-profile standard` uses the docker image to executes the processes

    * `-profile cluster` uses the singularity container and submits processes to a queue- optimal for HPC or LPC computing systems

    * `-profile all_of_us` uses the docker image to execute pipelines on the All of Us Researcher Workbench

* for more information visit the [Nextflow documentation](https://www.nextflow.io/docs/latest/cli.html)
# Configuration Parameters and Input File Descriptions

## Workflow


* `LD_panel_type` (Type: String)

    * LD panel types: 1000 genomes or UKBiobank. Label 1000 genomes as “1kg” and UKBB as “ukbb”

* `prscsx_group_cohort_map` (Type: Map (Dictionary))

    * prscsx computes scores for multiple ancestry groups (defined by LD reference panels) at the same time. it is possible that cohort names can include ancestry labels (especially when stitching a GWAS workflow to PRScsx). this maps a "score group nickname" (key) to cohort names (values) included in descriptor table. because prscsx can only include one analysis from each ancestry group, DO NOT duplicate ancestry groups in analysis groups. thus, prscsx will compute scores simultaneously by score group and phenotype. keys should be score groups and values should be a list of cohorts in each score group

* `input_descriptor_table_colnames` (Type: Map (Dictionary))

    * dictionary including names of essential columns in input descriptor table. specifies cohort (nickname or dataset), ancestry of LD reference panel (AFR, AMR, EAS, EUR, SAS), phenotype GWAS sample size, and summary statistic FULL FILE PATH AND FILENAME column names (If you want to use a relative path, add "${launchDir}/" in front of the path)

* `input_descriptor_table_filename (PRS-CSx)` (Type: File Path)

    * file that specifies the cohort, ancestry, phenotype, GWAS sample size, and summary statistics filename. must have five columns in this order: cohort, ancestry, phenotype, GWAS sample size, summary statistics filename. MUST BE A CSV (comma delimited)

* `gwas_meta_workflow_stitching` (Type: Bool (Java: true or false))

    * whether an individual is stitching another workflow to this one, specifically, stitching a GWAS or meta analysis to this workflow. DO NOT put true if stitching PRScsx to a workflow AFTER. Only put true if stitching to a workflow BEFORE PRScsx

* `my_python` (Type: File Path)

    * Path to the python executable to be used for python scripts - often it comes from the docker/singularity container (/opt/conda/bin/python)
## Pre-Processing


* `genome_build` (Type: String)

    * genome build of summary statistics, can be “b37” (build 37/hg19) or “b38” (build 38/hg38)

* `sumstats_colnames` (Type: Map (Dictionary))

    * dictionary that specifies the column names in your sumstats files. specifies chromosome column, position column, A1 column, A2 column, quantitative phenotype effect size column (beta or odds ratio), binary phenotype effect size column (beta or odds ratio), and significance column (pvalue or standard error). this assumes that all sumstats files have the same column names. do not change the keys (chr_colname, pos_colname, etc), only change the value accordingly.
## PRScsx


* `my_prscsx` (Type: File Path)

    * Path to the 

* `seed` (Type: String)

    * Non-negative integer which seeds the random number generator. This can be any number, I always pick 7.

* `write_posterior_samples` (Type: Bool (Java: true or false))

    * If True and META_FLAG also True, write all posterior samples of meta-analyzed SNP effect sizes after thinning. Default is False.

* `meta_flag` (Type: Bool (Java: true or false))

    * If True, return combined SNP effect sizes across populations using an inverse-variance-weighted meta-analysis of the population-specific posterior effect size estimates. Default is False.

* `mcmc_thinning_factor` (Type: String)

    * Thinning factor of the Markov chain. Default is 5

* `mcmc_burnin` (Type: String)

    * Number of burnin iterations. Default is 500 * the number of discovery populations

* `mcmc_iterations` (Type: String)

    * Total number of MCMC iterations. Default is 1,000 * the number of discovery populations

* `param_phi` (Type: String)

    * if NOT using, specify as 'auto'. if using: Global shrinkage parameter phi. If phi is not specified, it will be learnt from the data using a fully Bayesian approach. This usually works well for polygenic traits with very large GWAS sample sizes (hundreds of thousands of subjects). For GWAS with limited sample sizes (including most of the current disease GWAS), fixing phi to 1e-02 (for highly polygenic traits) or 1e-04 (for less polygenic traits), or doing a small-scale grid search (e.g., phi = 1e-6, 1e-4, 1e-2, 1) to find the optimal phi value in the validation dataset often improves perdictive performance. format should be 1e-02 (make sure to include the zero after the minus)

* `param_b` (Type: String)

    * Parameter b in the gamma-gamma prior. Default is 0.5

* `param_a` (Type: String)

    * Parameter a in the gamma-gamma prior. Default is 1

* `LD_directory` (Type: File Path)

    * FULL FILE PATH to directory with LD panels and Multi-Ancestry SNP info file. you MUST use LD panels from the PRScsx github because they have RSIDs matching the multi-ancestry SNP file that is specific to PRScsx. You must include a slash (/) after the directory name. If you want to use a relative path, add "${launchDir}/" in front of the path
# Output Files from PRS-CSx


* Violinplots

    * Type: Distribution Plot

    * Format: png

* SNP Weight Files

    * Variant-level scores produced by PRScsx, for all chromosomes

    * Type: PGS SNP Weights

    * Format: txt

    * Parallel By: Cohort, Phenotype, Ancestry

    * Output File Header:





    ```
    CHR     RSID    POS     A1      A2      PGS
    ```
# Current Dockerfile for the Container/Image


```docker
FROM continuumio/miniconda3
WORKDIR /app

# copy local SNP maps (match SNPs in LD panels) into docker image 
COPY snpinfo_mult_1kg_hm3_map_b37_b38 /app/snpinfo_mult_1kg_hm3_map_b37_b38
COPY snpinfo_mult_ukbb_hm3_map_b37_b38 /app/snpinfo_mult_ukbb_hm3_map_b37_b38

# give SNP maps read permissions
RUN chmod +r /app/snpinfo_mult_1kg_hm3_map_b37_b38 \
    && chmod +r /app/snpinfo_mult_ukbb_hm3_map_b37_b38 \    
    # install tools needed to install PRScsx
    && apt-get update \
    && apt-get install -y --no-install-recommends git \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/* \
    # download PRScsx
    && git clone --depth 1 -b master https://github.com/getian107/PRScsx.git \
    # install python packages needed for pipeline
    && conda install -y -n base -c conda-forge -c bioconda scipy=1.11.4 h5py pandas seaborn matplotlib pathlib python=3.11 \
    && conda clean --all --yes

USER root

```