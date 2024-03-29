
<!--
# patient-safety
Resource for *Population-scale identification of differential adverse events before and during a pandemic* ([preprint](https://www.medrxiv.org/content/10.1101/2021.01.17.21249988v1)) by Xiang Zhang, Marissa Sumathipala, and Marinka Zitnik.
-->

# Population-scale identification of differential adverse events before and during a pandemic


#### Authors: [Xiang Zhang](http://xiangzhang.info/), [Marissa Sumathipala](https://www.linkedin.com/in/marissa-sumathipala-558bb5179/), [Marinka Zitnik](https://zitniklab.hms.harvard.edu/)

#### [Paper](https://www.nature.com/articles/s43588-021-00138-4) on Nature Computational Science
#### [Project website](https://zitniklab.hms.harvard.edu/projects/patient-safety)
<!-- [Preprint](https://www.medrxiv.org/content/10.1101/2021.01.17.21249988v1) -->

## Approach overview

We investigate 10,443,476 adverse event reports (involving 19,193 adverse events and 3,624 drugs) from the U.S. Food and Drug Administration (FDA) Adverse Event Reporting System (FAERS) dataset, collected from January 2013 to September 2020. To support safety surveillance of therapeutic products, the FAERS stores anonymized, manually reviewed adverse event reports received by the FDA.

We use the dataset to detect adverse events that are significantly associated with the pandemic, pinpoint clinically relevant drugs strongly connected with adverse drug events, and identify disparities in the distribution of adverse events across sex and age. To this end, we develop an approach that identifies clinically meaningful adverse events that meet three criteria: i) the reporting frequency of the adverse event changed significantly between 2019 and 2020, ii) the change cannot be explained by its trend in previous years (2013 to 2019), and iii) the adverse drug reaction is strongly associated with at least one drug and cannot be explained by drug interference.

<p align="center">
<img src="https://github.com/mims-harvard/patient-safety/blob/main/image/workflow.JPG" width="1200" align="center">
</p>


## Regenerating results following scripts

- **Step 1**  `1_rawdata_download.ipynb`: Download raw FAERS data from 2013 Q1 to 2020 Q3 (31 quarters). We provide bash commands to efficiently download the quarterly dataset. See `XML_NTS.pdf` (provided in [Dataset](#dataset)) for a better understanding of data structure and description. 

- **Step 2**  `2_parsing_preprocessing.ipynb`: Extract key information from FAERS raw XML data, including administration (version, report\_id, case\_id, country, qualify, serious, serious subtype list, receivedate, receiptdate), demographic (age, gender, weight), adverse events list (each element represents an adverse drug reaction including PT\_code, PT, and outcome), and drug list (each element represents a drug, the key information is drug substance)). Some reports are incomplete, we fill a fixed number for missing values (more details are introduced in script). The files are parsed in quarter and then merged. We provide the curated and processed data at [Dataset](#dataset).

- **Step 3**  `3_generate_population_cohort.ipynb`: Load processed dataset and narrow down based on reporting country (only keep reports occurred in the US) and reporter's qualification (only keep reports submitted by healthcare professionals). In order to conduct population-scale analysis, we generate cohorts for different populations: all patients, male, female, young (age:1-19), adult (20-65), and elderly (>65). 

- **Step 4**  `4_disproportionality_estimation.ipynb`: This script corresponds to the *disproportionality estimation* as introduced in the online methods in our paper. For each population, we conduct disproportionality estimation on each adverse event to examine the association between the adverse event and pandemic (March 11–September 30, 2020) in contrast to before the pandemic (March 11–September 30, 2019). We here use disproportionality analysis in a novel way that quantifying the association between adverse events and their submission periods. We calculate the significance values using Fisher’s exact test followed by the Bonferroni correction for multiple hypothesis testing. We keep only the adverse events, which pass both the significance test (adjusted p-value < 0.05) and the ROR criterion.

- **Step 5**  `5_AE_trajectories.ipynb`: Implement *AE reporting trajectories* as described in our paper. For each adverse event, we calculate its incidence proportion from 2013 to 2019 and use 2-order auto-regressive regression to model its trajectory. We then define a dedicated pandemic-adverse event association index (PAEAI) to measure whether a medication’s incidence during the pandemic conforms to its trajectory. We only feed adverse events with positive PAEAI to the next step of our approach. In this file, we also map adverse events from *Preferred Terms (PTs)* level to *System Organ Classes (SOCs)* level, based on etiology (such as infections and infestations) and manifestation sites (such as gastrointestinal disorders) following the [MedDRA hierarchy](https://www.meddra.org/how-to-use/basics/hierarchy). 

- **Step 6**  `6_remove_drug_interference.ipynb`: Correspond to the step of *drug interference* in our approach. In this file, we check two aspects of the association among drugs, adverse events, and the pandemic. First, the adverse event (such as hallucination) should be significantly associated with the therapy of at least one drug (like Pimavanserin). Second, the formed drug-adverse event pair (like Pimavanserin-hallucination) should be significantly associated with the pandemic. **Note: 1) exchange the order of the above two moves will not change the results. 2)this step is kind of computational expensive as containing nest loops.**
<!--We use patient matching strategy by comparing adverse drug reactions in test group and control group. The test group is selected through propensity scores which are measured based on available information such as age, sex, weight, the qualification of the reporter, severity vector, and the submission date. -->

- **Step 7**  `7_results_analysis.ipynb`: This file mainly shows the results of our analysis, including the high-level comparison of different populations, enriched adverse events in each population, difference and common SOC of populations, significant drug-adverse event pairs in each population (both overrepresentation and underrepresentation), etc.


### Configuration and run scripts

Our scripts are run in [O2 cluster](https://wiki.rc.hms.harvard.edu/display/O2/) which is a platform for Linux-based high-performance computing at Harvard Medical School. The source codes are tested on Linux with Python 3.7, however, our scripts have **no** requirement of operating systems.

1. We recommend the user run scripts in a virtual environment. Build a virtual environment named *patientsafety* and install [Jupyter Notebook](https://jupyter.org/install) if you don't have it:

```
virtualenv patientsafety
source patientsafety/bin/activate
pip install jupyter
```
The installation will only takes a few minutes. 


2. Download our scripts to your local server/computer. Then creat a folder (such as `Code`) and copy all scripts in:
```
git clone https://github.com/mims-harvard/patient-safety.git
cd patient-safety
mkdir Code
mv *.ipynb Code/
```

3. Create a new folder named `Data` at the same level as `Code`. Then create subfolders: `curated` for all necessary data to run our scripts including patient safety dataset and supporting files (such as drug and adverse event ontology); `pandemic` to save intermediate results; a subfolder of `pandemic` called `results` to save final analyze results; `parsed` for datasets parsed from raw data through `2_parsing_preprocessing.ipynb`. You can do that in a terminal as following

```
mkdir Data
mkdir Data/curated
mkdir Data/pandemic/
mkdir Data/pandemic/results
mkdir Data/parsed
```


4. Next we need<span id="usepk"> </span>to prepare the curated dataset and mapping dictionary (for drugs and adverse events). The necessary dataset to reproduce our work are processed patient safety reports (`patient_safety.pk`), drug mapping files (`AE_mapping.pk `, `AE_dic.pk`), and adverse event mapping (`drug_mapping.pk `). Download them from [here](https://doi.org/10.7910/DVN/G9SHDA) or [here](#downloadlinks), and move data to folder `Data/curated/`. If your data path is different, please remember to revise file path accordingly. *Note: we also provide a corresponding .csv version for all the necessary data. Find them in the same link.*

5. Before running the codes, please make sure you have configured the required environment. The dependencies and versions are listed in `requirements.txt`. All the necessary packages can be installed using the following command

```
pip install -r requirements.txt
```


6. Open Jupyter Notebook and enjoy your analysis:
```
cd Code
jupyter notebook
```
About running time, `2_parsing_preprocessing.ipynb`, `3_generate_population_cohort.ipynb`, and `6_remove_drug_interference.ipynb` will take a rather long time (like a few hours) due to large-scale dataset and nest loops; the other scripts are much faster (such as ten minutes).



### Convert to python scripts
Our codes (`*.ipynb`) are written in Jupyter Notebook. It is very easy to convert codes to python scripts. Take `3_generate_population_cohort.ipynb` as an example, run the following command in your terminal,

```
jupyter nbconvert --to script 3_generate_population_cohort.ipynb
```
The above command will generate a `3_generate_population_cohort.py` which can be directly run in python IDEs (such as PyCharm) or in terminal.

```
python 3_generate_population_cohort.py
```


### Apply our model to flexible scenarios

The users can easily modify **3\_generate\_population\_cohort.ipynb** to generate their interested cohorts as a function of reporting time, age, sex, weight, drug, adverse events, etc. For example, our work investigates the period from `03-11` to `09-30` from 2013 to 2020. If anyone wants to analyze a different period, just replace the start or end time: such as replace `09-30` by `12-31` to study the period from March 11 to December 31. 



## Dataset description
<span id="dataset"> </span>

We provide [all necessary dataset](https://doi.org/10.7910/DVN/G9SHDA) for reproducing our work, or direct download files [here](#downloadlinks).


### Data for running
In our scripts, data in pickle format are directly used to generate results. However, for preview and broader use, we also supply all required dataset in non-language-specific format ([CSV version](#csv)). See instructions for use these `.pk` files [here](#usepk).

#### Patient safety dataset

- **patient_safety.pk**: Processed FAERS dataset (We depicted the procedure of raw data download, data parsing, and preprocessing in `1_rawdata_download.ipynb` and `2_parsing_preprocessing.ipynb`). Duplicated reports are already removed. This is a DataFrame where each row denotes an independent report. The rows represent the report version, report ID, case ID, country, qualify, severity, severity subtype 1,  severity subtype 2, severity subtype 3, severity subtype 4, severity subtype 5, severity subtype 6, receivedate, receiptdate, age, gender, weight, adverse events list (each element represents an adverse drug reaction including PT\_code, PT name, and outcome), and drug list (each element represents a drug, the key information is drug substance). Please find more details about the meaning of each column in `2_parsing_preprocessing.ipynb` and `XML_NTS.pdf`. 


- **XML_NTS.pdf**: Included in raw data released by FDA. This file describes the data structure in the XML files in detail. Use of the XML file adheres to the original standards documents for E2b and the M2 implementation specifications.

#### Drug and adverse event ontology (mapping tables)

- **AE\_mapping.pk**: This is a DataFrame stored in `pickle` format. This file contains all adverse events in MedDRA ontology. Each row denotes one adverse event. The columns denote the name and code of the adverse event in different levels: PT (preferred terms), HLT (high-level term), HLGT (high-level group term), and SOC (system organ class). In particular, the columns are *PT*, *PT\_name*, *HLT*, *HLT\_name*, *HLGT*, *HLGT\_name*, *SOC*, *SOC\_name*, and *SOC\_abbr* (abbreviation of SOC name).

- **AE\_dic.pk**: Extracted adverse event name (*PT\_name*) from **AE\_mapping.pk** and form as a dictionary, which enables fast indexing the adverse event. 


- **drug\_mapping.pk**: Dictionary that maps drug substance from text to DrugBank identifier retrieved from [DrugBank Vocabulary](https://go.drugbank.com/releases/latest#open-data). The *key* of the dictionary is textural name of drug substance, the corresponding *value* is a list including two elements: DrugBank ID and a number representing the order in which the drug appeared in the FAERS dataset.

<!-- - **drug\_dic.pk**: Extracted drug names from **drug\_mapping.pk** and form as a dictionary, which enables fast indexing the drug. -->

- **drug\_ATC\_mapping.csv**: Anatomical Therapeutic Chemical ([ATC](https://www.whocc.no/atc_ddd_index/)) ontology. The ATC categorization is an internationally accepted classification system maintained by the WHO that classifies active ingredients of drugs according to the organ or system on which they act and their therapeutic, pharmacological, and chemical properties. This file contains the broad class of ATC codes, each subclass level in a separate column, the DrugBank ID, FAERS code ID, and string name. 


**Note: All data about MedDRA ontology are retrieved from [MedDRA](https://www.meddra.org/) through [Non-Profit/Non-Commercial License](https://www.meddra.org/subscription/subscription-type) and can only be used in non-profit activities!**

### Data in CSV format
<span id="csv"> </span>

- **patient_safety.csv**, **AE\_mapping.csv**, and ***drug\_mapping.csv** are the same files in CSV format of **patient_safety.pk**, **AE\_mapping.pk**, and **drug\_mapping.pk**. These CSV files are not used in our scripts but offer opportunities for researchers to study population-scale patient safety in various platforms and programming languages.

<!-- *AE\_dic.csv**,  **drug\_dic.csv**-->

### Analysis results

We share the intermediate and final results of our analysis. 

#### Final results

The following results are organized in the same way: `beta` represents the association between adverse events and the pandemic period (p-value is measured by Fisher’s exact test, adjusted by Bonferroni correction). The adverse events are sorted in descending order of PAEAI. The defined PAEAI presents the extent (higher PAEAI suggests larger deviation) that adverse event deviates from its historical trend (Methods). All identified adverse events have positive PAEAI which indicates the reporting pattern during the pandemic doesn’t follow its trajectory. The ‘E’ and ‘P’ in the ‘E\P’ column denotes this adverse event is enriched or purified, respectively. 

- **Identified_adverse_events_in_all_patients.csv**: Summary of adverse events identified by our approach in all patients. 

- **Identified_adverse_events_in_male.csv**: Summary of adverse events identified by our approach in male patients.

- **Identified_adverse_events_in_female.csv**: Summary of adverse events identified by our approach in female patients.

- **Identified_adverse_events_in_young.csv**: Summary of adverse events identified by our approach in young patients.

- **Identified_adverse_events_in_adults.csv**: Summary of adverse events identified by our approach in adult patients.

- **Identified_adverse_events_in_elderly.csv**: Summary of adverse events identified by our approach in elderly patients.

#### Intermediate results
We present intermediate results during our analysis. The top 100 results (adverse events or drug-AE pairs) the lowest p-value are already reported in [supplementary](https://www.medrxiv.org/content/10.1101/2021.01.17.21249988v1.full.pdf+html) Tables 7-9. Here show the full table of them.

- **Associations_between_adverse_events_and_the_pandemic.csv**: Evaluate the association between adverse events and the pandemic period. The `Occur 2019` and `Nonoccur 2019` denote the submitted reports that contain and do not contain the specific adverse event in 2019 (March 11 -- September 30), respectively. Similarly, `Occur 2020` and `Nonoccur 2020` denote the statistics in the same period of 2020. The p-value is measured by Fisher's exact test followed by Bonferroni correction. Higher `beta` and smaller p-value indicate the adverse event has stronger association with the pandemic.

- **Associations_between_adverse_events_and_the_pandemic_Top100.csv**: The top 100 adverse events in **Associations_between_adverse_events_and_the_pandemic.csv** with the lowest p-values. Sorted in descending order of `beta`. Correspond to SI Table 7. 

- **Associations_between_adverse_events_and_drugs.csv**: Evaluate the association between adverse events and drugs during the pandemic period. Contains the number of patients who: exposed to the drug and have the adverse event (denoted by `A`); exposed to the drug but not have the adverse event (denoted by `B`); not exposed to the drug but have the adverse event (denoted by `C`); not exposed to the drug and not have the adverse event (denoted by `D`). All the numbers (A, B, C, and D) are measured during the pandemic period (March 11 -- September 30, 2020). We conduct patient matching and split patients into test group and control group. The size of control group is ten times of that of test group, which means that 10 *(A+B) = C+D (Methods). Higher `gamma` and smaller p-value indicate the adverse event has stronger association with the drug. 

- **Associations_between_adverse_events_and_drugs_Top100.csv**: The top 100 drug-adverse event pairs of **Associations_between_adverse_events_and_drugs.csv** with the lowest p-values. Sorted in descending order of `gamma`. Correspond to SI Table 8. 

- **Associations_between_drug_adverse_event_pairs_and_the_pandemic.csv**: Evaluate the association between drug-adverse event pairs and the pandemic. The `Occur 2019` and `Nonoccur 2019` denote the number of submitted reports that contain and do not contain the specific drug-adverse event pair in 2019 (March 11 -- September 30), respectively. Similarly, `Occur 2020` and `Nonoccur 2020` denote the statistics in the same period of 2020. The p-value is measured by Fisher's exact test followed by Bonferroni correction. Higher `delta` and smaller p-value indicate the pair has stronger connection with the pandemic. 

- **Associations_between_drug_adverse_event_pairs_and_the_pandemic_Top100.csv**: The top 100 drug-adverse event pairs of **Associations_between_drug_adverse_event_pairs_and_the_pandemic.csv** with the lowest p-values. Sorted in descending order of `delta`. Correspond to SI Table 9. 

## Dataset download 
<span id="downloadlinks"> </span> 

All files in dataset can be downloaded [here](https://doi.org/10.7910/DVN/G9SHDA). Download portals for separate files:

### Data in pickle format
- [patient_safety.pk](https://dataverse.harvard.edu/api/access/datafile/4416415)

- [XML_NTS.pdf](https://dataverse.harvard.edu/api/access/datafile/4415941)

- [AE_mapping.pk](https://dataverse.harvard.edu/api/access/datafile/4416414)

- [AE_dic.pk](https://dataverse.harvard.edu/api/access/datafile/4416416)

- [drug_mapping.pk](https://dataverse.harvard.edu/api/access/datafile/4416417)

- [drug_ATC_mapping.csv](https://dataverse.harvard.edu/api/access/datafile/4416430)


### Data in CSV format
- [patient_safety.csv](https://dataverse.harvard.edu/api/access/datafile/4416428)

- [XML_NTS.pdf](https://dataverse.harvard.edu/api/access/datafile/4415941)

- [AE_mapping.csv](https://dataverse.harvard.edu/api/access/datafile/4416429)

- [drug_mapping.csv](https://dataverse.harvard.edu/api/access/datafile/4416431)

- [drug_ATC_mapping.csv](https://dataverse.harvard.edu/api/access/datafile/4416430)


### Results
- [Identified_adverse_events_in_all_patients.csv](https://dataverse.harvard.edu/api/access/datafile/4416446)

- [Identified_adverse_events_in_male.csv](https://dataverse.harvard.edu/api/access/datafile/4416447)

- [Identified_adverse_events_in_female.csv](https://dataverse.harvard.edu/api/access/datafile/4416443)

- [Identified_adverse_events_in_young.csv](https://dataverse.harvard.edu/api/access/datafile/4416444)

- [Identified_adverse_events_in_adults.csv](https://dataverse.harvard.edu/api/access/datafile/4416448)

- [Identified_adverse_events_in_elderly.csv](https://dataverse.harvard.edu/api/access/datafile/4416445)

- [Associations_between_adverse_events_and_the_pandemic.csv](https://dataverse.harvard.edu/api/access/datafile/4416450)

- [Associations_between_adverse_events_and_the_pandemic_Top100.csv](https://dataverse.harvard.edu/api/access/datafile/4416451)

- [Associations_between_adverse_events_and_drugs.csv](https://dataverse.harvard.edu/api/access/datafile/4416454)

- [Associations_between_adverse_events_and_drugs_Top100.csv](https://dataverse.harvard.edu/api/access/datafile/4416452)

- [Associations_between_drug_adverse_event_pairs_and_the_pandemic.csv](https://dataverse.harvard.edu/api/access/datafile/4416455)

- [Associations_between_drug_adverse_event_pairs_and_the_pandemic_Top100.csv](https://dataverse.harvard.edu/api/access/datafile/4416449)


## Miscellaneous

Please send any questions you might have about the approach and/or the scripts to <xiang_zhang@hms.harvard.edu>.

## License

This work is licensed under the MIT License.
