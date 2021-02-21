# patient-safety
Resource for *Population-scale patient safety data reveal inequalities in adverse events before and during COVID-19 pandemic* ([preprint](https://www.medrxiv.org/content/10.1101/2021.01.17.21249988v1)) by Xiang Zhang, Marissa Sumathipala, and Marinka Zitnik.


## Regenerating results following scripts

- 1\_rawdata\_download.ipynb: Download raw FAERS data from 2013 Q1 to 2020 Q3 (31 quarters). We provide bash commands to effeciently download quarterly dataset. See XML_NTS.pdf (provided in [Dataset](#dataset)) to have a better understanding of data structure and description. 

- 2\_parsing\_preprocessing.ipynb: Extract key information from FAERS raw XML data, including administration (version, report\_id, case\_id, country, qualify, serious, serious subtype list, receivedate, receiptdate), demographic (age, gender, weight), adverse events list (each element represents an adverse drug reaction including PT\_code, PT, and outcome), and drug list (each element represents a drug, the key information is drug substance)). Some reports are incomplete, we fill a fixed number for missing values (more details are introduced in script). The files are parsed in quarter and then merged together. We provide the curated and processed data at [Dataset](#dataset).

- 3\_generate\_population\_cohort.ipynb: Load processed dataset and narrow down based on reporting country (only keep reports occurred in US) and reporter's qualification (only keep reports submitted by healthcare professionals). In order to conduct population-scale analysis, we generate cohort for different populations: all patients, male, female, young (age:1-19), adult (20-65), and elderly (>65). 

- 4\_disproportionality\_estimation.ipynb:

- 5\_AE\_trajectories.ipynb:

- 6\_remove\_drug\_interference.ipynb:

- 7\_results\_analysis.ipynb: 

### Run our model in different time period

## Dataset
<span id="dataset"> </span>

We provide [dataset](https://dataverse.harvard.edu/privateurl.xhtml?token=d796b626-23b9-4a60-86d3-5525fda3c108) for reproducing our work:
- Name: Processed FAERS dataset (We depicted the procedure of raw data download, data parse, and preprocessing in 1_.ipynb)


## Miscellaneous

Please send any questions you might have about the code and/or the algorithm to <xiang_zhang@hms.harvard.edu>.

## License

This work is licensed under the MIT License.
