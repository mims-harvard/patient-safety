# patient-safety
<!--
Resource for *Population-scale patient safety data reveal inequalities in adverse events before and during COVID-19 pandemic* ([preprint](https://www.medrxiv.org/content/10.1101/2021.01.17.21249988v1)) by Xiang Zhang, Marissa Sumathipala, and Marinka Zitnik.
-->

## Population-scale patient safety data reveal inequalities in adverse events before and during COVID-19 pandemic


#### Author: [Xiang Zhang](http://xiangzhang.info/), [Marissa Sumathipala](https://www.linkedin.com/in/marissa-sumathipala-558bb5179/), [Marinka Zitnik](https://zitniklab.hms.harvard.edu/)

#### [Project website](https://zitniklab.hms.harvard.edu/projects/patient-safety), [Preprint](https://www.medrxiv.org/content/10.1101/2021.01.17.21249988v1)



## Regenerating results following scripts

- `1\_rawdata\_download.ipynb`: Download raw FAERS data from 2013 Q1 to 2020 Q3 (31 quarters). We provide bash commands to efficiently download the quarterly dataset. See `XML_NTS.pdf` (provided in [Dataset](#dataset)) for a better understanding of data structure and description. 

- `2\_parsing\_preprocessing.ipynb`: Extract key information from FAERS raw XML data, including administration (version, report\_id, case\_id, country, qualify, serious, serious subtype list, receivedate, receiptdate), demographic (age, gender, weight), adverse events list (each element represents an adverse drug reaction including PT\_code, PT, and outcome), and drug list (each element represents a drug, the key information is drug substance)). Some reports are incomplete, we fill a fixed number for missing values (more details are introduced in script). The files are parsed in quarter and then merged. We provide the curated and processed data at [Dataset](#dataset).

- `3\_generate\_population\_cohort.ipynb`: Load processed dataset and narrow down based on reporting country (only keep reports occurred in the US) and reporter's qualification (only keep reports submitted by healthcare professionals). In order to conduct population-scale analysis, we generate cohorts for different populations: all patients, male, female, young (age:1-19), adult (20-65), and elderly (>65). 

- `4\_disproportionality\_estimation.ipynb`: This script corresponds to the *disproportionality estimation* as introduced in the online methods in our paper. For each population, we conduct disproportionality estimation on each adverse event to examine the association between the adverse event and pandemic (March 11–September 30, 2020) in contrast to before the pandemic (March 11–September 30, 2019). We here use disproportionality analysis in a novel way that quantifying the association between adverse events and their submission periods. We calculate the significance values using Fisher’s exact test followed by the Bonferroni correction for multiple hypothesis testing. We keep only the adverse events, which pass both the significance test (adjusted p-value < 0.05) and the ROR criterion.

- `5\_AE\_trajectories.ipynb`: Implement *AE reporting trajectories* as described in our paper. For each adverse event, we calculate its incidence proportion from 2013 to 2019 and use 2-order auto-regressive regression to model its trajectory. We then define a dedicated pandemic-adverse event association index (PAEAI) to measure whether a medication’s incidence during the pandemic conforms to its trajectory. We only feed adverse events with positive PAEAI to the next step of our approach. In this file, we also map adverse events from *Preferred Terms (PTs)* level to *System Organ Classes (SOCs)* level, based on etiology (such as infections and infestations) and manifestation sites (such as gastrointestinal disorders) following the [MedDRA hierarchy](https://www.meddra.org/how-to-use/basics/hierarchy). 

- `6\_remove\_drug\_interference.ipynb`: Correspond to the step of *drug interference* in our approach. In this file, we check two aspects of the association among drugs, adverse events, and the pandemic. First, the adverse event (such as hallucination) should be significantly associated with the therapy of at least one drug (like Pimavanserin). Second, the formed drug-adverse event pair (like Pimavanserin-hallucination) should be significantly associated with the pandemic. *Note: 1) exchange the order of the above two moves will not change the results. 2)this step is kind of computational expensive as containing nest loops.* 
<!--We use patient matching strategy by comparing adverse drug reactions in test group and control group. The test group is selected through propensity scores which are measured based on available information such as age, sex, weight, the qualification of the reporter, severity vector, and the submission date. -->

- `7\_results\_analysis.ipynb`: This file mainly shows the results of our analysis, including the high-level comparison of different populations, enriched adverse events in each population, difference and common SOC of populations, significant drug-adverse event pairs in each population (both overrepresentation and underrepresentation), etc.


### Configuration and run scripts

Our scripts are run in [O2 cluster](https://wiki.rc.hms.harvard.edu/display/O2/) which is a platform for Linux-based high-performance computing at Harvard Medical School. 

- Download our scripts to your local server/computer through:
```
git clone https://github.com/mims-harvard/patient-safety.git
```

- Please put all the scripts in same folder (such as `Code`), and create a new folder named `Data` in the parent folder. Then create a subfolder in `Data` named `pandemic`, and a subfolder of `pandemic` called `results`. These folders are used to save results. You can do that in a terminal as following

```
cd ..
mkdir Data
mkdir Data/pandemic/
mkdir Data/pandemic/results
```

- Before running the codes, please make sure you have configured the required environment. All the necessary packages can be installed using the following command

```
pip install -r requirements.txt
```

- We recommend the user run scripts in a virtual environment. Build a virtual environment named *patientsafety* and run it in Jupyter Notebook:

```
virtualenv patientsafety
source patientsafety/bin/activate
pip install jupyter
jupyter notebook
```

- Enjoy your analysis.

### Convert to python scripts
Our codes (`*.ipynb`) are written in Jupyter Notebook. It is very easy to convert codes to python scripts. Take `3\_generate\_population\_cohort.ipynb` as an example, run the following command in your terminal,

```
jupyter nbconvert --to script 3\_generate\_population\_cohort.ipynb
```
The above command will generate a `3\_generate\_population\_cohort.py` which can be directly run in python IDEs (such as PyCharm) or in terminal.

```
python 3\_generate\_population\_cohort.py
```


### Apply our model to flexible scenarios

The users can easily modify **3\_generate\_population\_cohort.ipynb** to generate their interested cohorts as a function of reporting time, age, sex, weight, drug, adverse events, etc. For example, our work investigates the period from 03-11 to 09-30 from 2013 to 2020. If anyone wants to analyze a different period, just replace the start or end time: such as replace '09-30' by '12-31' to study the period from March 11 to December 31. 



## Dataset 
<span id="dataset"> </span>

We provide [all necessary datasets](https://dataverse.harvard.edu/privateurl.xhtml?token=d796b626-23b9-4a60-86d3-5525fda3c108) for reproducing our work:

- **reports_v4_pd_new.pk**: Processed FAERS dataset (We depicted the procedure of raw data download, data parsing, and preprocessing in `1\_rawdata\_download.ipynb` and `2\_parsing\_preprocessing.ipynb`). Duplicated reports are already removed. The file size is 2.3G. This is a DataFrame where each row denotes an independent report. The rows represent the report version, report ID, case ID, country, qualify, severity, severity subtype 1,  severity subtype 2, severity subtype 3, severity subtype 4, severity subtype 5, severity subtype 6, receivedate, receiptdate, age, gender, weight, adverse events list (each element represents an adverse drug reaction including PT\_code, PT name, and outcome), and drug list (each element represents a drug, the key information is drug substance). Please find more details about the meaning of each column in `2\_parsing\_preprocessing.ipynb` and `XML_NTS.pdf`. 


- **XML_NTS.pdf**: Included in raw data released by FDA. This file describes the data structure in the XML files in detail. Use of the XML file adheres to the original standards documents for E2b and the M2 implementation specifications.

- **MedDRA\_dic\_all.pk**: This is a DataFrame stored in `pickle` format. This file contains all adverse events in MedDRA ontology. Each row denotes one adverse event. The columns denote the name and code of the adverse event in different levels: PT (preferred terms), HLT (high-level term), HLGT (high-level group term), and SOC (system organ class). In particular, the columns are *PT*, *PT\_name*, *HLT*, *HLT\_name*, *HLGT*, *HLGT\_name*, *SOC*, *SOC\_name*, and *SOC\_abbr* (abbreviation of SOC name).

- **MedDRA\_dic.pk**: Same content with **MedDRA\_dic\_all.pk** but is formed as dictionary instead of DataFrame. The *key* of the dictionary is an adverse event name in PT level, the corresponding *value* is a list including nine elements: 


- **drug\_dic\_FAERS.pk**: Dictionary that maps drug substance from text to DrugBank identifier retrieved from [DrugBank Vocabulary](https://go.drugbank.com/releases/latest#open-data). The *key* of the dictionary is textural name of drug substance, the corresponding *value* is a list including two elements: DrugBank ID and a number representing the order in which the drug appeared in the FAERS dataset.


**Note: All data about MedDRA ontology are retrieved from [MedDRA](https://www.meddra.org/) through [Non-Profit/Non-Commercial License](https://www.meddra.org/subscription/subscription-type) and can only be used in non-profit activities!**


## Miscellaneous

Please send any questions you might have about the approach and/or the scripts to <xiang_zhang@hms.harvard.edu>.

## License

This work is licensed under the MIT License.
