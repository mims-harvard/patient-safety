# patient-safety
Resource for *Population-scale patient safety data reveal inequalities in adverse events before and during COVID-19 pandemic* ([preprint](https://www.medrxiv.org/content/10.1101/2021.01.17.21249988v1)) by Xiang Zhang, Marissa Sumathipala, and Marinka Zitnik.


## Regenerating results following scripts

- `1\_rawdata\_download.ipynb`: Download raw FAERS data from 2013 Q1 to 2020 Q3 (31 quarters). We provide bash commands to effeciently download quarterly dataset. See XML_NTS.pdf (provided in [Dataset](#dataset)) to have a better understanding of data structure and description. 

- `2\_parsing\_preprocessing.ipynb`: Extract key information from FAERS raw XML data, including administration (version, report\_id, case\_id, country, qualify, serious, serious subtype list, receivedate, receiptdate), demographic (age, gender, weight), adverse events list (each element represents an adverse drug reaction including PT\_code, PT, and outcome), and drug list (each element represents a drug, the key information is drug substance)). Some reports are incomplete, we fill a fixed number for missing values (more details are introduced in script). The files are parsed in quarter and then merged together. We provide the curated and processed data at [Dataset](#dataset).

- `3\_generate\_population\_cohort.ipynb`: Load processed dataset and narrow down based on reporting country (only keep reports occurred in US) and reporter's qualification (only keep reports submitted by healthcare professionals). In order to conduct population-scale analysis, we generate cohort for different populations: all patients, male, female, young (age:1-19), adult (20-65), and elderly (>65). 

- `4\_disproportionality\_estimation.ipynb`: This script corresponds to the *disproportionality estimation* as introduced in the online methods in our paper. For each population, we conduct disproportionality estimation on each adverse event to examine the association between the adverse event and pandemic (March 11–September 30, 2020) in contrast to before the pandemic (March 11–September 30, 2019). We here use disproportionality analysis in a novel way that quantifying the association between adverse events and their submission periods. We calculate the significance values using the Fisher’s exact test followed by the Bonferroni correction for multiple hypothesis testing. We keep only the adverse events, which pass both the significance test (adjusted p-value < 0.05) and the ROR criterion.

- `5\_AE\_trajectories.ipynb`: Implement *AE reporting trajectories* as described in paper. For each adverse event, we calculate its incidence proportion from 2013 to 2019 and use 2-order auto-regressive regression to model its trajectory. We then define pandemic-adverse event association index (PAEAI) to measure whether a medication’s incidence during the pandemic conforms to its trajectory. We only feed adverse events with positive PAEAI to the next step of our approach. In this file, we also map adverse events from *Preferred Terms (PTs)* level to *System Organ Classes (SOCs)* level, based on etiology (such as infections and infestations) and manifestation sites (such as gastrointestinal disorders) following the [MedDRA hierarchy](https://www.meddra.org/how-to-use/basics/hierarchy). 

- `6\_remove\_drug\_interference.ipynb`: 

- `7\_results\_analysis.ipynb`: 

### Run our model in flexible scenario

The users can easily modify **3\_generate\_population\_cohort.ipynb** to generate their interested cohorts as a function of reporting time, age, sex, weight, drug, adverse events, etc. For example, our work investigates the period from 03-11 to 09-30 from 2013 to 2020. If anyone want to analyze different time period, just replace the start or end time: such as replace '09-30' by '12-31' to study the period from March 11 to December 31. 

## Dataset 
<span id="dataset"> </span>
### FAERS
We provide [dataset](https://dataverse.harvard.edu/privateurl.xhtml?token=d796b626-23b9-4a60-86d3-5525fda3c108) for reproducing our work:
- **Name**: Processed FAERS dataset (We depicted the procedure of raw data download, data parse, and preprocessing in 1_.ipynb)

### MedDRA

link to meddra, non-commencial

### ATC

## Miscellaneous

Please send any questions you might have about the approach and/or the scipts to <xiang_zhang@hms.harvard.edu>.

## License

This work is licensed under the MIT License.
