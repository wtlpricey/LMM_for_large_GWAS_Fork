# LMM_for_large_GWAS
COMP383/483 project

## General Dependencies (Install If Needed)
To run the included scripts, users need to have the following programs (and packages) installed:

- R & Rscript (dplyr, ggplot2, qqman, data.table) [ANY MISSING?]  

## LMM Tools and PLINK2 Installation

The following tools (and any necessary dependencies) were installed according to each tool's Wiki page (in this repo).
* BOLT-LMM
* SAIGE
* SUGEN (so far not entirely functional)
* REGENIE
* PLINK2

## Preprocessing & File Preparation

Original simulated data files generated by Dr. Wheeler; n=5000 plink files with a continuous and a categorical/case-control phenotype file. True beta values for both phenotypes also provided. Located in:
```
#plink files
/home/data/simualted_gwas/simu_genos*
#pheno files
/home/data/simulated_gwas/simu*.phen
#true beta values
/home/data/simulated_gwas/simu*.par
```

Subsetted using subsampling.R and subsampling2.sh. path to subsetted plink files: 
```
/home/igregga/LMM_files/*simu-genos* 
#test if you can read them
head /home/igregga/LMM_files/*simu-genos.bim
```
VCF file conversion using plink_to_vcf.sh. path:
```
/home/igregga/LMM_files/vcfs/
```
Header added to pheno files using cat on the command line. Case/control coding converted from 1/2/-9 to 0/1/NA using convert_casecontrol_codes.R. path:
```
/home/igregga/LMM_files/phenos/
```
## Running Test Scripts with Test Data

The directory `test` includes generalized scripts and small data files to test functionality. All scripts should be run from within the `test` directory.

### PLINK2 (baseline)

### BOLT-LMM

BOLT requires that categorical phenotypes are coded control=0 and case=1, but otherwise script can be used for categorical or continuous phenotypes with no adjustment needed.

Command to run with provided test data using nohup:
```
nohup bash run_BOLT_test.sh -i 100simu-genos -p simu_categorical-01na.phen -c TRAIT -o 100simu-genos_cc_bolt -t 2 > nohup_BOLT_test.out &
```
A more generalized command (update with user-specific parameters):
```
nohup bash run_BOLT_test.sh -i <bed-bim-fam_input_prefix> -p <phenotype_file> -c <pheno_column_name> -o <output_prefix> -t <number_threads> > nohup_BOLT_test.out &
```
### REGENIE

### SAIGE

## Benchmark Runs

*Note: This section details the scripts and data used to benchmark the tools in this project. May include hardcoded paths, as many files are too big to store on github.*

For benchmark runs, each tool was set to use 2 threads where possible. Time was measured using the `time` command and memory usage by `/usr/bin/time --verbose`, which logs the maximum resident set size. The general command format:
```
time /usr/bin/time --verbose <tool command>
```
This was incorporated into each of the tool-specific shell scripts detailed below.

### BOLT-LMM

The script to run BOLT, `run_BOLT.sh`, first completes a GWAS for the continous trait (looping through each of the subsets) and then completes a GWAS for the categorical trait. 

Command to run:
```
nohup bash run_BOLT.sh > nohup_BOLT.out &
```
Location of results files:
```
# final BOLT output for continous trait:
/home/tfischer1/LMM_for_large_GWAS/BOLT_results/1250simu-genos_qt_stats.tab 
/home/tfischer1/LMM_for_large_GWAS/BOLT_results/2500simu-genos_qt_stats.tab
/home/tfischer1/LMM_for_large_GWAS/BOLT_results/5000simu-genos_qt_stats.tab

# final output for categorical:
/home/tfischer1/LMM_for_large_GWAS/BOLT_results/1250simu-genos_cc_stats.tab
/home/tfischer1/LMM_for_large_GWAS/BOLT_results/2500simu-genos_cc_stats.tab
/home/tfischer1/LMM_for_large_GWAS/BOLT_results/5000simu-genos_cc_stats.tab

# log file:
/home/tfischer1/LMM_for_large_GWAS/nohup_BOLT.out 
```

### SAIGE

SAIGE requires two steps for each GWAS run. The first step fits a null model. We used a full GRM (Genetic Relationship Matrix). The second step performs a single-variant association test.

The benchmark script for SAIGE, `run_SAIGE.sh`, first completes the two-step GWAS for the continous trait (looping through the subset sizes) and then does the same for the categorical trait. Note that time and memory are measured separately for step 1 and step 2 of each run.

Command to run:
```
nohup bash run_SAIGE.sh > nohup_SAIGE.out &
```
Location of results files:
```
# final SAIGE output for continous trait:
/home/tfischer1/LMM_for_large_GWAS/SAIGE_results/1250_qt_fullGRM_with_vr.txt
/home/tfischer1/LMM_for_large_GWAS/SAIGE_results/2500_qt_fullGRM_with_vr.txt
/home/tfischer1/LMM_for_large_GWAS/SAIGE_results/5000_qt_fullGRM_with_vr.txt

# for categorical trait:
/home/tfischer1/LMM_for_large_GWAS/SAIGE_results/1250_cc_fullGRM_with_vr.txt
/home/tfischer1/LMM_for_large_GWAS/SAIGE_results/2500_cc_fullGRM_with_vr.txt
/home/tfischer1/LMM_for_large_GWAS/SAIGE_results/5000_cc_fullGRM_with_vr.txt

# log file:
/home/tfischer1/LMM_for_large_GWAS/nohup_SAIGE.out  
```

### SUGEN

SUGEN has a weird error. See wiki page.

### REGENIE

REGENIE like SAIGE also takes two steps, both wrapped in run_REGENIE.sh and run_REGENIE-binary.sh (one for continuous pheno, one categorical/binary pheno). Run with:
```
nohup /home/igregga/LMM_for_large_GWAS/run_REGENIE.sh > /home/igregga/regenie-out/regeniecont.log &
nohup /home/igregga/LMM_for_large_GWAS/run_REGENIE-binary.sh > /home/igregga/regenie-out/regeniecat.log &
```
Tool generates multiple log files for each run, and prints much of this info to command line (so it's a very long nohup log). The actual results are:
```
/home/igregga/regenie-out/*.regenie
```

### PLINK2 (baseline)
The script to run PLINK2, `run_PLINK2.sh`, completes a GWAS for the continuous trait for each subset and then the categorical. 

The command to run the plink2 script is:
```
nohup bash run_PLINK2.sh > PLINK2.out &
```
Location of results files, including the log file, can all be found in the same directory: `/home/wprice2/gwas_results`.

## Analysis

### Scalability
Runtime was recorded using the `--time` command which outputs CPU realtime, CPU user time, and CPU system time. To achieve the most holistic metric, we added user time and system time in our final analysis. 

Memory was recorded using the `--verbose` command which recorded the max CPU the job may occupy (aka max resident) in kilobytes. These values were converted to gigabytes in the final analysis. 

### Accuracy

The effect sizes of new tools were correlated with the effect sizes of PLINK2 results with p<1e-05, under the assumption that these should capture causal SNPs. Seven SNPs were significant at the genome-wide threshold 5e-08 for the continuous trait with PLINK2, and these were highlighted in continuous GWAS results of the new tools.

## Usability Ratings

### PLINK2.0
**Installation & Accessibility**: 5/5

**Usability**: 5/5

**Overall**: 5/5

**Notes**: There is a reason why plink is the industry standard!

### BOLT-LMM
**Installation & Accessibility**: 5/5

**Usability**: 4/5

**Overall**: 4/5

**Notes**: Runs in one step and relatively easy to use. Requires reference LD scores.

### SAIGE
**Installation & Accessibility**: 4/5

**Usability**: 3/5

**Overall**: 3/5

**Notes**:Takes two steps per run, with different settings for categorical and continuous in each step. One the one hand, the plethera of flag options means it's very customizable, but it's a lot to sort through. If fitting a full GRM model, takes a LONG time to run!

### REGENIE
**Installation & Accessibility**: 5/5, if using provided conda env. 2/5 if not

**Usability**: 4/5

**Overall**: 4/5

**Notes**: Dependency requirements are complicated, but conda works smoothly and easily. Needs two steps. Documentation is pretty thorough, with explanation of the algorithm/process etc.

### SUGEN
**Installation & Accessibility**: 3/5

**Usability**: 0/5

**Overall**: 1/5

**Notes**: It has potential: installed easily enough, but documentation has a lot of room for improvement. No help flag. Would only need one step to run, but ran into an error message about formatting (?) and couldn't figure out what is needed to fix it. Could not run. Opened a github issue but haven't gotten any response.

