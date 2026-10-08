Supplementary Code and Data
===============================================================================
Integrated in silico network toxicology, machine learning and molecular
simulations to predict candidate colorectal cancer-related targets of three
novel non-phthalate plasticizers

Manuscript submitted to Discover Oncology.

This repository contains the R source code used to explore the potential
carcinogenic mechanisms of three alternative non-phthalate plasticizers
(DOTP, DINCH and TMCH) in colorectal cancer (CRC).

------------------------------------------------------------------------------
1. Project structure
------------------------------------------------------------------------------
Code_Data/
├─ Batch-correction-and-differential-expression-analysis/
│   ├─ Batch-correction-and-differential-expression-analysis.R
│   └─ Processed_Data/
├─ Machine-learning-and-SHAP-analysis/
│   ├─ ML_SHAP_analysis.R
│   └─ Processed_Data/
├─ WGCNA/
│   └─ WGCNA.R
└─ README.txt

Each module writes its results to a `results/` directory created at runtime
(`results/data`, `results/figures`, `results/params`, `results/logs`).

------------------------------------------------------------------------------
2. Environment
------------------------------------------------------------------------------
R version 4.5.1

Main R packages
  batch correction / DEG      limma, sva, pheatmap, ggplot2
  network analysis            WGCNA
  machine learning            glmnet, e1071, randomForestSRC, gbm, MASS,
                              xgboost, klaR, plsRglm, mboost, caret
  model evaluation            pROC, PRROC
  model interpretation        kernelshap, shapviz
  visualisation               ComplexHeatmap, circlize, ggplot2, cowplot
  utilities                   readxl, foreach, doParallel, tidyr

A multiple-core machine is recommended. All scripts set an explicit random
seed (123) at every stochastic step, so results are reproducible.

------------------------------------------------------------------------------
3. Analysis workflow
------------------------------------------------------------------------------
Step 1  Batch correction and differential expression analysis
        ComBat integration of GSE21510 and GSE9348 into the training cohort,
        followed by differential expression and WGCNA-based identification of
        CRC-related genes.

Step 2  Weighted gene co-expression network analysis (WGCNA)
        Module detection and module-trait correlation to define the
        CRC-associated gene set.

Step 3  Machine learning and SHAP interpretation
        Screening of 118 machine-learning pipelines across six cohorts,
        followed by SHAP-based prioritisation of the core targets.

Run the scripts in the order above. Each script states the required input
files and the parameters and thresholds used in the manuscript.

------------------------------------------------------------------------------
4. Cohorts used for machine learning (six in total)
------------------------------------------------------------------------------
Cohort        Samples   Normal / CRC   Role
training        230        37 / 193     GSE21510 + GSE9348, merged and
                                         ComBat batch-corrected
GSE24514         49        15 / 34      external validation
GSE32323         34        17 / 17      external validation
TCGA-CRC        431        51 / 380     external validation
GSE44076        148        50 / 98      external validation
GSE113513        28        14 / 14      external validation (paired design)

All cohorts were restricted to the same 79-gene panel shared between the
plasticizer-associated and CRC-associated gene sets.

TCGA-CRC is the true colorectal cohort: the colon adenocarcinoma (TCGA-COAD)
and rectum adenocarcinoma (TCGA-READ) cohorts combined, downloaded as the
merged COADREAD matrix from the UCSC Xena hub (HiSeqV2, log2(norm_count+1)):

  https://tcga-xena-hub.s3.us-east-1.amazonaws.com/download/
      TCGA.COADREAD.sampleMap%2FHiSeqV2.gz

Sample type is taken from characters 14-15 of the TCGA barcode
(01 = primary solid tumour, 11 = solid tissue normal); recurrent (02) and
metastatic (06) samples are excluded so that the tumour-versus-normal contrast
matches the other cohorts.

------------------------------------------------------------------------------
5. Model grid
------------------------------------------------------------------------------
    22 single-algorithm models
  + 96 two-stage pipelines
    (8 feature-selection strategies x 12 classifiers)
  = 118 pipelines, each evaluated in all six cohorts (708 metric rows)

Feature selection is performed inside each cross-validation fold on the
training partition only, so held-out samples never contribute to the definition
of the gene subset.

Model hyperparameters were pre-specified rather than optimised by grid search,
so that no information from the held-out cohorts could influence model
construction. Where a package provides internal cross-validation (regularised
regression lambda; gradient boosting and XGBoost iteration counts; plsRglm
components), that internal cross-validation is run within the training
partition only.

Nine metrics are recorded for every pipeline in every cohort: AUC with its
DeLong 95% confidence interval, PR-AUC (trapezoidal integration), F1-score,
accuracy, sensitivity, specificity, precision, Matthews correlation
coefficient, and Brier score (mean squared error of the predicted
probabilities). Threshold-dependent metrics use the Youden-optimal threshold.

Pipelines are ranked by the mean AUC across the six cohorts. Because
tumour-versus-normal discrimination is near ceiling in these cohorts, the
reported model was selected from the leading group on the basis of parsimony
and stability of the retained gene set rather than on nominal AUC. Null-control
analyses are included to quantify how much of this discrimination is intrinsic
to the classification task.

------------------------------------------------------------------------------
6. Data availability
------------------------------------------------------------------------------
Raw transcriptomic data are publicly available from the NCBI Gene Expression
Omnibus (GEO):

  GSE21510, GSE9348    training cohort
  GSE24514, GSE32323   external validation
  GSE44076             external validation
  GSE113513            external validation
  GSE166555, EMTAB8107 single-cell validation

and from The Cancer Genome Atlas (TCGA) colorectal cohort via the UCSC Xena
hub (see section 4).

Processed intermediate data used by each module are provided in the
`Processed_Data` subfolders.

------------------------------------------------------------------------------
7. Usage
------------------------------------------------------------------------------
1. Clone or download this repository.
2. Set the working directory to the module folder, or edit DATA_DIR in the
   script so that it points to the corresponding `Processed_Data` folder.
3. Run the scripts in the workflow order given in section 3.

All scripts are annotated with the parameters and thresholds used in the
manuscript. Session information for the reported run is written to
`results/logs/sessionInfo.txt` automatically.

------------------------------------------------------------------------------
8. License
------------------------------------------------------------------------------
MIT License
