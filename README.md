
Documentation for PRS-CSx
=========================

# Module Overview


PRScsx is a tool used to calculate polygenic scores. It integrates GWAS summary statistics and LD panels across multiple ancestry groups to improve global polygenic prediction. This nextflow pipeline seemlessly takes GWAS summary statistics, makes the input files for PRScsx, runs PRScsx, concatenates the chromosome-separated outcomes, and produces a summary violin plot, all in a parallelized fashion.

[Paper Link for Reference](https://www.nature.com/articles/s41588-022-01054-7)

[Tool Documentation Link](https://github.com/getian107/PRScsx/tree/master)
## Cloning Github Repository:


* Command: `git clone https://github.com/PMBB-Informatics-and-Genomics/geno_pheno_workbench.git`

* Navigate to relevant workflow directory run commands (our pipelines assume all of the nextflow files/scripts are in the current working directory)
## Software Requirements:


* [Nextflow version 23.04.1.5866](https://www.nextflow.io/docs/latest/cli.html)

* [Singularity version 3.8.3](https://sylabs.io/docs/) OR [Docker version 4.30.0](https://docs.docker.com/)
## Commands for Running the Workflow


* Singularity Command: `singularity build prscsx.sif docker://pennbiobank/prscsx:latest`

* Docker Command: `docker pull pennbiobankprscsx:latest`

* Command to Pull from Google Container Registry: `docker pull gcr.io/verma-pmbb-codeworks-psom-bf87/need-to-update`

* Run Command: `nextflow run prscsx.nf -resume -profile cluster`

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

# Output Files from PRS-CSx


* SNP Weight Files

    * Type: PGS SNP Weights

    * Format: txt

    * Parallel By: Cohort, Phenotype
# Example Config File Contents


```
// set up params
// can be customized by user
params {
    // python path
    my_python = "/opt/conda/bin/python" // path to python in docker containe
    
    // whether an individual is stitching another workflow to this one, specifically, stitching a GWAS or meta analysis to this workflow
    // true or false
    // DO NOT put true if stitching PRScsx to a workflow AFTER. Only put true if stitching to a workflow BEFORE PRScsx
    gwas_meta_workflow_stitching = false
    
    // list of chromosomes
    chromosome_list = (21..22)

    /*
    file that specficies the cohort, ancestry, phenotype, GWAS sample size, and summary statistics filename
    must have five columns in this order: cohort, ancestry, phenotype, GWAS sample size, summary statistics filename
    cohort: a nickname, for example, "AFR_ALL", "AFR_M", "PMBB"
    ld_ref_ancestry: LD reference panel to be used in PRScsx, specified by 1000 genomes (1KG) genetically inferred ancestry group (AFR, AMR, EAS, EUR, SAS)
    sumstats_filename: specify summary statistics filenames with RELATIVE file path
    descriptor filename:
    */
    descriptor_table_filename='test_descriptor_file.txt'
    // delimiter in summary table
    descriptor_table_delim='\t'
    /*
    dictionary including names of essential columns in sumstats summary table
    specifies cohort, ancestry, phenotype sample size, and summary statistic filename column names
    */
    descriptor_table_colnames = [cohort_colname: 'COHORT',
                                        ld_ref_ancestry_colname: 'LD_PANEL_ANCESTRY',
                                        phenotype_colname: 'PHENOTYPE',
                                        sample_size_colname: 'SAMPLE_SIZE',
                                        sumstats_filename_colname: 'SUMSTATS_FILENAME']

    /*
    prscsx computes scores for multiple ancestry groups (defined by LD reference panels) at the same time
    it is possible that cohort names can include ancestry labels (especially when stitching a GWAS workflow to PRScsx)
    this maps a "score group nickname" (key) to cohort names (values) included in descriptor table
    because prscsx can only include one analysis from each ancestry group, DO NOT duplicate ancestry groups in analysis groups
    thus, prscsx will compute scores simultaneously by score group and phenotype
    keys should be score groups and values should be a list of cohorts in each score group
    */
    prscsx_group_cohort_map = ['PRScsx_test_data': ['PRScsx_test_data1','PRScsx_test_data2','PRScsx_test_data3','PRScsx_test_data4']]
    
    /*
    dictionary that specifies the column names in your sumstats files
    specifies chr column, pos column, A1 column, A2 column, beta or odds ration column, and pvalue or standard error column
    this assumes that all sumstats files have the same column names
    do not change the keys (chr_colname, pos_colname, etc), only change the value accordingly
    */
    colnames = [chr_colname: '#CHROM', // chromosome column name in sumstats file
                pos_colname: 'BP', // base pair/position column name in sumstats file
                a1_colname: 'A1', // A1 or alternate (ALT) allele column name in sumstats file
                a2_colname: 'A2', // A2 or reference (REF) allele column name in sumstats file
                quant_beta_or_colname: 'BETA', // beta or odds ratio column name for quantitative phenotypes in sumstats file (either works)
                bin_beta_or_colname: 'OR', // beta or odds ratio column name for binary phenotypes in sumstats file (either works)
                pval_se_colname: 'P'] // p-value or standard error column name in sumstats file (either works)
    /*
    genome build of summary statistics
    b37 or b38
    */
    genome_build='b38'
    /*
    directory with LD panels and multi-ancestry snp info file
    you MUST use LD panels from the PRScsx github
    because they have RSIDs matching a multi-ancestry SNP file that is specific to PRScsx
    (https://github.com/getian107/PRScsx)
    you must include a slash (/) after the directory name and include the RELATIVE file path
    */
    LD_directory = 'ID_Reference/'

    // LD panel types: 1000 genomes or UKBB
    // 1000 genomes label: 1kg
    /// UKBB label: ukbb
    LD_panel_type = '1kg'

    // other prscsx parameters
    param_a = '1' // Parameter a in the gamma-gamma prior. Default is 1
    param_b = '0.5' // Parameter b in the gamma-gamma prior. Default is 0.5
    param_phi = 'auto' /* if NOT using, specify as 'auto'.
    if using: Global shrinkage parameter phi.
    If phi is not specified, it will be learnt from the data using a fully Bayesian approach.
    This usually works well for polygenic traits with very large GWAS sample sizes (hundreds of thousands of subjects).
    For GWAS with limited sample sizes (including most of the current disease GWAS),
    fixing phi to 1e-02 (for highly polygenic traits)
    or 1e-04 (for less polygenic traits),
    or doing a small-scale grid search (e.g., phi = 1e-6, 1e-4, 1e-2, 1)
    to find the optimal phi value in the validation dataset often improves perdictive performance.
    format should be 1e-02 (make sure to include the zero after the minus) */
    mcmc_iterations = '2000' // Total number of MCMC iterations. Default is 1,000 * the number of discovery populations
    mcmc_burnin = '1000' // Number of burnin iterations. Default is 500 * the number of discovery populations
    mcmc_thinning_factor = '5' // Thinning factor of the Markov chain. Default is 5
    meta_flag = 'False' /* If True,
    return combined SNP effect sizes across populations
    using an inverse-variance-weighted meta-analysis of the population-specific posterior effect size estimates.
    Default is False. */
    seed = '7' // Non-negative integer which seeds the random number generator. This can be any number, I always pick 7.
}

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
# Current `nextflow.config` contents


```
includeConfig "${launchDir}/prscsx.config"

// set up profile
// change these parameters as needed
profiles {
    non_docker_dev {
        process.executor = awsbatch-or-lsf-or-slurm-etc
    }

    standard {
        process.executor = awsbatch-or-lsf-or-slurm-etc
        process.container = 'pennbiobank/prscsx:latest'
        docker.enabled = true
    }
    cluster {
        process.executor = awsbatch-or-lsf-or-slurm-etc
        process.queue = 'epistasis_normal'
        process.memory = '30GB'
        process.container = 'prscsx.sif'
        singularity.enabled = true
        singularity.runOptions = '-B /root/,/directory/,/names/'
    }

    all_of_us {
        process.executor = awsbatch-or-lsf-or-slurm-etc
        process.memory = '15GB'
        process.container = 'gcr.io/ritchie-aou-psom-9015/prscsx:latest'
        google.zone = "us-central1-a"
        google.project = 'terra-vpc-sc-bb404549' // change to your project id
        google.lifeSciences.debug = true
        google.lifeSciences.network = "network"
        google.lifeSciences.subnetwork = "subnetwork"
        google.lifeSciences.usePrivateAddress = false
        google.lifeSciences.serviceAccountEmail = 'pet-2666723902222ba8b8580@terra-vpc-sc-bb404549.iam.gserviceaccount.com' // change to your service email
        google.lifeSciences.copyImage = "gcr.io/google.com/cloudsdktool/cloud-sdk:alpine"
        google.enableRequesterPaysBuckets = true
        workDir='gs://fc-secure-f3e7d01e-18fa-40ba-bb3e-4d7497ba7d5b/work/' // change to your working directory in your workspace bucket
    }
}

```