
Documentation for PRS-CSx
=========================

# Module Overview


PRScsx is a tool used to calculate polygenic scores. It integrates GWAS summary statistics and LD panels across multiple ancestry groups to improve global polygenic prediction. This nextflow pipeline seemlessly takes GWAS summary statistics, makes the input files for PRScsx, runs PRScsx, concatenates the chromosome-separated outcomes, and produces a summary violin plot, all in a parallelized fashion.
- [Tool Paper Link for Reference](https://www.nature.com/articles/s41588-022-01054-7)
- [Tool Documentation Link for Reference](https://github.com/getian107/PRScsx/tree/master)
- [Example Config File](https://github.com/PMBB-Informatics-and-Genomics/pmbb-geno-pheno-toolkit/tree/main/Example_Configs/prscsx.config)
- [Example nextflow.config File](https://github.com/PMBB-Informatics-and-Genomics/pmbb-geno-pheno-toolkit/tree/main/Example_Configs/nextflow.config)

## Software Requirements


* [Nextflow version 24.04.3](https://www.nextflow.io/docs/latest/cli.html)

* [Singularity 3.8.3](https://sylabs.io/docs/) OR [Docker 4.30.0](https://docs.docker.com/)
## Commands for Running the Workflow


* Singularity Command: `singularity build prscsx.sif docker://pennbiobank/prscsx:latest`

* Docker Command: `docker pull pennbiobank/prscsx:latest`

* Pull from Google Container Registry: `docker pull gcr.io/verma-pmbb-codeworks-psom-bf87/prscsx:latest`

* Run Command: `nextflow run /path/to/toolkit/module/prscsx.nf`

* Common `nextflow run` flags:

    * `-resume` flag picks up workflow where it left off

    * `-stub` performs a dry run, checks channels without executing code

    * `-profile` selects the compute profiles in nextflow.config

    * `-profile standard` uses the Docker image to execute processes

    * `-profile cluster` uses the Singularity container and submits processes to a queue

    * `-profile all_of_us` uses the Docker image on All of Us Workbench

* More info: [Nextflow documentation](https://www.nextflow.io/docs/latest/cli.html)
# Detailed Pipeline Steps

## Part I: Setup


1. Start your own tools directory and go there. You may do this in your project analysis directory, but it often makes sense to clone into a general `tools` location

```sh
# Make a directory to clone the pipeline into
TOOLS_DIR="/path/to/tools/directory"
mkdir $TOOLS_DIR
cd $TOOLS_DIR
```

2. Download the source code by cloning from git

```sh
git clone None
cd $TOOLS_DIR/pmbb-nf-toolkit-prscsx
```

3. Build the singularity image
    - you may call the image whatever you like, and store it wherever you like. Just make sure you specify the name in `nextflow.conf`
    - this does NOT have to be done for every saige-based analysis, but it is good practice to re-build every so often as we update regularly.


```sh
cd $TOOLS_DIR/pmbb-nf-toolkit-prscsx
singularity build prscsx.sif docker://pennbiobank/prscsx:latest
```
## Part II: Configure your run


1. Make a separate analysis/run/working directory.
    - The quickest way to get started, is to run the analysis in the folder the pipeline is run. However, subsequent analyses will over-write results from previous analyses.
    - ❗This step is optional, but We Highly recommend making a `tools` directory separate from your `run` directory. We recommend storing the `nextflow.conf` in here as it shouldn't change between runs.


```sh
WDIR="/path/to/analysis/run1"
mkdir -p $WDIR
cd $WDIR
```

2. Fill out the `nextflow.config` file for your system.
    - See [Nextflow configuration documentation](https://www.nextflow.io/docs/latest/config.html) for information on how to configure this file. An example can be found on our GitHub: [Nextflow Config](https://github.com/PMBB-Informatics-and-Genomics/pmbb-geno-pheno-toolkit/blob/main/Example_Configs/nextflow.config).
    - ❗IMPORTANTLY, you must configure a user-defined profile for your run environments (local, docker, saige, cluster, etc.). If multiple profiles are specified, run with a specific profile using `nextflow run -profile $MY_PROFILE`.
    - For singularity, The profile's attribute `process.container` should be set to `'/path/to/prscsx.sif'` (replace `/path/to` with the location where you built the image above). See [Nextflow Executor Information](https://www.nextflow.io/docs/latest/executor.html) for more details.
    - ⚠️As this file remains mostly unchanged for your system, We recommend storing this file in the `tools/pipeline` directory and passing it to the pipeline with `-c /path/to/nextflow.config`.


3. Create a pipeline-specific `.config` file specifying your run parameters and input files. See Below for workflow-specific parameters and what they mean.
    - Everything in here can be configured in `nextflow.config`, however we find it easier to separate the system-level profiles from the individual run parameters.
    - Examples can be found in our Pipeline-Specific [Example Config Files](https://github.com/PMBB-Informatics-and-Genomics/pmbb-geno-pheno-toolkit/tree/main/Example_Configs).
    - you can compartamentalize your config file as much as you like by passing
    - There are 2 ways to specify the config file during a run:

        - with the `-c` option on the command line: `nextflow run -c PRScsx/prscsx.config`
        - in the `nextflow.config`: at the top of the file add: `includeConfig PRScsx/prscsx.config`

## Part III: Run your analysis


❗We HIGHLY recommend doing a STUB run to test the analysis using the `-stub` flag. This is a dry run to make sure your environment, parameters, and input_files are specified and formatted correctly.❗We also HIGHLY recommend doing a TEST run with the included test data in `$TOOLS_DIR/pmbb-nf-toolkit-prscsx/test_data`we have several pre-configured analyses runs with input data and fully-specified config files.

```sh
# run an exwas stub
nextflow run $TOOLS_DIR/pmbb-nf-toolkit-prscsx/prscsx.nf \
   -profile cluster \
   -c /path/to/nextflow.config \
   -c PRScsx/prscsx.config \
   -stub

# run an exwas for real
nextflow run $TOOLS_DIR/pmbb-nf-toolkit-prscsx/prscsx.nf \
   -profile cluster \
   -c /path/to/nextflow.config \
   -c PRScsx/prscsx.config

# resume an exwas run if it was interrupted or ran into an error
nextflow run $TOOLS_DIR/pmbb-nf-toolkit-prscsx/prscsx.nf \
   -profile cluster \
   -c /path/to/nextflow.config \
   -c PRScsx/prscsx.config \
   -resume
```
# Pipeline Parameters

## Input Files for PRS-CSx


* LD Panels

    * Genetic ancestry specific linkage disequilibrium (LD) panels. These can be generated from 1000 genomes (1KG) data or UK Biobank (UKBB data). They must be for one of the five 1000 genomes super populations (AFR, AMR, EAS, EUR, SAS). These can be downloaded from the PRS-CSx GitHub: 

    * Type: LD Panels

    * Format: hdf5

* my_prscsx

    * Path to the 

    * Type: Script

    * Format: py

* GWAS Summary Statistics

    * Genome-wide association study summary statistic files.

    * Type: Summary Statistics

    * Format: txt

    * File Header:


    ```
    chromosome      base_pair_location      variant_id      effect_allele   other_allele    beta    standard_error  p_value effect_allele_frequency ci_lower  odds_ratio       ci_upper
    ```

* Multi-ancestry SNP file

    * Multi-ancestry SNP information file with ancestry-specific allele frequencies and allele flips. The SNPs match those in the LD panels. This can be downloaded from the PRS-CSx GitHub: 

    * Type: LD Panels

    * Format: txt

    * File Header:


    ```
    CHR     SNP     BP      A1      A2      FRQ_AFR FRQ_AMR FRQ_EAS FRQ_EUR FRQ_SAS FLP_AFR FLP_AMR FLP_EAS FLP_EUR FLP_SAS
    ```
## Output Files for PRS-CSx


* Violinplots

    * Type: Distribution Plot

    * Format: png

* SNP Weight Files

    * Variant-level scores produced by PRScsx, for all chromosomes

    * Type: PGS SNP Weights

    * Format: txt

    * File Header:


    ```
    CHR     RSID    POS     A1      A2      PGS
    ```

        * Parallel By: Cohort, Phenotype, Ancestry
## Other Parameters for PRS-CSx

### PRScsx


* `write_posterior_samples` (Type: Bool (Java: true or false))

    * If True and META_FLAG also True, write all posterior samples of meta-analyzed SNP effect sizes after thinning. Default is False.

* `mcmc_burnin` (Type: String)

    * Number of burnin iterations. Default is 500 * the number of discovery populations

* `seed` (Type: String)

    * Non-negative integer which seeds the random number generator. This can be any number, I always pick 7.

* `param_b` (Type: String)

    * Parameter b in the gamma-gamma prior. Default is 0.5

* `meta_flag` (Type: Bool (Java: true or false))

    * If True, return combined SNP effect sizes across populations using an inverse-variance-weighted meta-analysis of the population-specific posterior effect size estimates. Default is False.

* `mcmc_thinning_factor` (Type: String)

    * Thinning factor of the Markov chain. Default is 5

* `my_prscsx` (Type: File Path)

    * Path to the 

* `LD_directory` (Type: File Path)

    * FULL FILE PATH to directory with LD panels and Multi-Ancestry SNP info file. you MUST use LD panels from the PRScsx github because they have RSIDs matching the multi-ancestry SNP file that is specific to PRScsx. You must include a slash (/) after the directory name. If you want to use a relative path, add "${launchDir}/" in front of the path

* `param_a` (Type: String)

    * Parameter a in the gamma-gamma prior. Default is 1

* `param_phi` (Type: String)

    * if NOT using, specify as 'auto'. if using: Global shrinkage parameter phi. If phi is not specified, it will be learnt from the data using a fully Bayesian approach. This usually works well for polygenic traits with very large GWAS sample sizes (hundreds of thousands of subjects). For GWAS with limited sample sizes (including most of the current disease GWAS), fixing phi to 1e-02 (for highly polygenic traits) or 1e-04 (for less polygenic traits), or doing a small-scale grid search (e.g., phi = 1e-6, 1e-4, 1e-2, 1) to find the optimal phi value in the validation dataset often improves perdictive performance. format should be 1e-02 (make sure to include the zero after the minus)

* `mcmc_iterations` (Type: String)

    * Total number of MCMC iterations. Default is 1,000 * the number of discovery populations
### Pre-Processing


* `genome_build` (Type: String)

    * genome build of summary statistics, can be “b37” (build 37/hg19) or “b38” (build 38/hg38)

* `sumstats_colnames` (Type: Map (Dictionary))

    * dictionary that specifies the column names in your sumstats files. specifies chromosome column, position column, A1 column, A2 column, quantitative phenotype effect size column (beta or odds ratio), binary phenotype effect size column (beta or odds ratio), and significance column (pvalue or standard error). this assumes that all sumstats files have the same column names. do not change the keys (chr_colname, pos_colname, etc), only change the value accordingly.
### Workflow


* `LD_panel_type` (Type: String)

    * LD panel types: 1000 genomes or UKBiobank. Label 1000 genomes as “1kg” and UKBB as “ukbb”

* `input_descriptor_table_colnames` (Type: Map (Dictionary))

    * dictionary including names of essential columns in input descriptor table. specifies cohort (nickname or dataset), ancestry of LD reference panel (AFR, AMR, EAS, EUR, SAS), phenotype GWAS sample size, and summary statistic FULL FILE PATH AND FILENAME column names (If you want to use a relative path, add "${launchDir}/" in front of the path)

* `my_python` (Type: File Path)

    * Path to the python executable to be used for python scripts - often it comes from the docker/singularity container (/opt/conda/bin/python)

* `input_descriptor_table_filename (PRS-CSx)` (Type: File Path)

    * file that specifies the cohort, ancestry, phenotype, GWAS sample size, and summary statistics filename. must have five columns in this order: cohort, ancestry, phenotype, GWAS sample size, summary statistics filename. MUST BE A CSV (comma delimited)

* `prscsx_group_cohort_map` (Type: Map (Dictionary))

    * prscsx computes scores for multiple ancestry groups (defined by LD reference panels) at the same time. it is possible that cohort names can include ancestry labels (especially when stitching a GWAS workflow to PRScsx). this maps a "score group nickname" (key) to cohort names (values) included in descriptor table. because prscsx can only include one analysis from each ancestry group, DO NOT duplicate ancestry groups in analysis groups. thus, prscsx will compute scores simultaneously by score group and phenotype. keys should be score groups and values should be a list of cohorts in each score group

* `gwas_meta_workflow_stitching` (Type: Bool (Java: true or false))

    * whether an individual is stitching another workflow to this one, specifically, stitching a GWAS or meta analysis to this workflow. DO NOT put true if stitching PRScsx to a workflow AFTER. Only put true if stitching to a workflow BEFORE PRScsx
# Configuration and Advanced Workflow Files

## Example Config File Contents (From Path)


```
// set up params
// can be customized by user
params {
    // python path
    my_python = "/opt/conda/bin/python" // path to python in docker/singularity container
    // PRScsx path
    my_prscsx = "/app/PRScsx/PRScsx.py" // path to PRScsx.py script in docker/singularity container (If you use your own, please use v1.1.0 or later to match the options used in the script)
    
    // whether an individual is stitching another workflow to this one, specifically, stitching a GWAS or meta analysis to this workflow
    // true or false
    // DO NOT put true if stitching PRScsx to a workflow AFTER. Only put true if stitching to a workflow BEFORE PRScsx
    gwas_meta_workflow_stitching = false
    
    // list of chromosomes
    chromosome_list = (21..22)

    /*
    file that specficies the cohort, ancestry, phenotype, GWAS sample size, and summary statistics filename
    must have five columns: cohort, ancestry, phenotype, GWAS sample size, summary statistics filename
    cohort: a nickname, for example, "AFR_ALL", "AFR_M", "PMBB"
    ld_ref_ancestry: LD reference panel to be used in PRScsx, specified by 1000 genomes (1KG) genetically inferred ancestry group (AFR, AMR, EAS, EUR, SAS)
    sumstats_full_filepath: specify summary statistics filenames with FULL file path (If you want to use a relative path, add "${launchDir}/" in front of the path)
    MUST BE A CSV (comma delimited)
    descriptor filename:
    */
    input_descriptor_table_filename='test_descriptor_file.csv'
    /*
    dictionary including names of essential columns in input descriptor table
    specifies cohort, ancestry of LD reference panel, phenotype, sample size, and summary statistic FULL FILE PATH AND FILENAME column names
    */
    input_descriptor_table_colnames = [cohort_colname: 'COHORT',
                                        ld_ref_ancestry_colname: 'LD_PANEL_ANCESTRY',
                                        phenotype_colname: 'PHENOTYPE',
                                        sample_size_colname: 'SAMPLE_SIZE',
                                        sumstats_full_filepath_colname: 'SUMSTATS_FILEPATH']

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
    specifies chromosome column, position column, A1 column, A2 column, quantitative phenotype effect size column (beta or odds ratio), binary phenotype effect size column (beta or odds ratio), and significance column (pvalue or standard error)
    this assumes that all sumstats files have the same column names
    do not change the keys (chr_colname, pos_colname, etc), only change the value accordingly
    */
    sumstats_colnames = [chr_colname: '#CHROM', // chromosome column name in sumstats file
                pos_colname: 'BP', // base pair/position column name in sumstats file
                a1_colname: 'A1', // A1 or alternate (ALT) allele column name in sumstats file
                a2_colname: 'A2', // A2 or reference (REF) allele column name in sumstats file
                quant_beta_or_colname: 'BETA', // beta or odds ratio column name for quantitative phenotypes in sumstats file (either works)
                bin_beta_or_colname: 'OR', // beta or odds ratio column name for binary phenotypes in sumstats file (either works)
                pval_se_colname: 'P'] // p-value or standard error column name in sumstats file (either works)
    /*
    genome build of summary statistics
    can be build 37 (b37/hg19) or build 38 (b38/hg38)
    specify build 37 as "b37" and build 38 as "b38"
    */
    genome_build='b38'
    /*
    FULL FILE PATH to directory with LD panels and Multi-Ancestry SNP info file
    you MUST use LD panels from the PRScsx github because they have RSIDs matching a multi-ancestry SNP file that is specific to PRScsx
    (https://github.com/getian107/PRScsx)
    you must include a slash (/) after the directory name
    If you want to use a relative path, add "${launchDir}/" in front of the path
    */
    ld_directory = 'ID_Reference/'

    // LD panel types: 1000 genomes or UKBiobank
    // 1000 genomes label: 1kg
    /// UKBB label: ukbb
    ld_panel_type = '1kg'

    // true/false statement to update output genome build to b38 (PRScsx outputs it to b37)
    // must be all lower case with no quotes (true or false)
    b38_output_genome_build = true

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
    write_posterior_samples = 'False' // If True and META_FLAG also True, write all posterior samples of meta-analyzed SNP effect sizes after thinning. Default is False.
    seed = '7' // Non-negative integer which seeds the random number generator. This can be any number, I always pick 7.
}

```
## Current `nextflow.config` contents


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
        // CHANGE EVERY TIME! These are specific for each user, see docs
        google.lifeSciences.serviceAccountEmail = service@email.gservicaaccount.com
        workDir = /path/to/workdir/ // can be gs://
        google.project = terra project id

        // These should not be changed unless you are an advanced user
        process.container = 'gcr.io/verma-pmbb-codeworks-psom-bf87/prscsx:latest' // GCR SAIGE docker container (static)

        // these are AoU, GCR parameters that should NOT be changed
        process.memory = '15GB' // minimum memory per process (static)
        process.executor = awsbatch-or-lsf-or-slurm-etc
        google.zone = "us-central1-a" // AoU uses central time zone (static)
        google.location = "us-central1"
        google.lifeSciences.debug = true 
        google.lifeSciences.network = "network"
        google.lifeSciences.subnetwork = "subnetwork"
        google.lifeSciences.usePrivateAddress = false
        google.lifeSciences.copyImage = "gcr.io/google.com/cloudsdktool/cloud-sdk:alpine"
        google.enableRequesterPaysBuckets = true
        // google.lifeSciences.bootDiskSize = "20.GB" // probably don't need this
    }
}

```