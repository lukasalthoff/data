# Representative US Census Links (1850-1950)
This repository contains crosswalks to link census records for men and women in the United States from 1850 to 1950, constructed by combining historical census records with Social Security Number (SSN) application data. The dataset enables tracking individuals across multiple censuses despite name changes (e.g., due to marriage) and is particularly valuable for including women in the study of intergenerational mobility.

### Citation
If you use this data, please cite:
Althoff, Lukas, Brookes Gray, Harriet, & Reichardt, Hugo (2024). The Missing Link(s): 
Women and Intergenerational Mobility. [Working Paper](https://lukasalthoff.github.io/pdf/igm_mothers.pdf)

## Data features
- Links 42 million unique individuals across non-adjacent census decades (186 million total linked census records)
- Covers approximately 16-24% of the US population during 1910-1940 and 2-14% during 1850-1900
- Tracks women before and after marriage using SSN application data containing both maiden and married names
- More representative than existing historical linked census datasets, particularly for women and racial minorities

## Data construction
The dataset was constructed through a multi-stage process:
1. **SSN Application Linking**

   Matched SSN applications to census records using detailed criteria including
   - Full names (applicant's first and middle names and parents' full names)
   - Race
   - Sex
   - Year of birth
   - State of birth
2. **Parent Identification**

   Linked applicants' parents to census records using
   - SSN application parent information
   - Census household relationships
   - Parent birthplaces
   - Spouse names
3. **Cross-Census Linking**

   Connected census records across decades using the established SSN and synthetic identifiers. The final dataset consists of crosswalks between “histids” (unique identifiers of individuals specific to each census decade) across adjacent and non-adjacent census decades.

## Getting started
1. **Download Census Crosswalks**
   - Download the crosswalk files from this repository 
   - Files are organized by census year pairs (e.g., `crosswalk_1850_1860.csv`)
   - Each file contains linking identifiers and weights
2. **Obtain Census Data from IPUMS**
   - Go to IPUMS USA website
   - Select census years
   - Add variables of interest (histid identifiers will be included automatically)
3. **Merge Crosswalks with Census Data**

### Code Examples
<details>
<summary>Python</summary>

```python
census1900 = pd.read_csv('census1900.csv')
census1930 = pd.read_csv('census1930.csv')
crosswalk = pd.read_csv('1900_1930.csv')

# Add year suffix to 1900 census variables
census1900.columns = [f"{col}_1900" if col != 'histid' else col for col in census1900.columns]

# Rename histid to match crosswalk
census1900 = census1900.rename(columns={'histid': 'histid1900'})

# Merge 1900 census with crosswalk
merged_data = crosswalk.merge(
    census1900,
    on='histid1900',
    how='left'
)

# Add year suffix to 1930 census variables
census1930.columns = [f"{col}_1930" if col != 'histid' else col for col in census1930.columns]

# Rename histid to match crosswalk
census1930 = census1930.rename(columns={'histid': 'histid1930'})

# Complete the merge with 1930 census
final_data = merged_data.merge(
    census1930,
    on='histid1930',
    how='left'
)

# Example: Study occupational transitions
occupational_mobility = final_data.groupby(['occupation_1900', 'occupation_1930']).size()
```
</details>

<details>
<summary>R</summary>

```r
# Load required libraries
library(dplyr)
library(tidyr)

# Read the data
census1900 <- read.csv('census1900.csv')
census1930 <- read.csv('census1930.csv')
crosswalk <- read.csv('1900_1930.csv')

# Add year suffix to 1900 census variables
names(census1900) <- ifelse(names(census1900) != "histid",
                           paste0(names(census1900), "_1900"),
                           names(census1900))

# Rename histid to match crosswalk
names(census1900)[names(census1900) == "histid"] <- "histid1900"

# Merge 1900 census with crosswalk
merged_data <- left_join(crosswalk, census1900, by = "histid1900")

# Add year suffix to 1930 census variables
names(census1930) <- ifelse(names(census1930) != "histid",
                           paste0(names(census1930), "_1930"),
                           names(census1930))

# Rename histid to match crosswalk
names(census1930)[names(census1930) == "histid"] <- "histid1930"

# Complete the merge with 1930 census
final_data <- left_join(merged_data, census1930, by = "histid1930")

# Example: Study occupational transitions
occupational_mobility <- final_data %>%
  group_by(occupation_1900, occupation_1930) %>%
  summarise(count = n())
```
</details>

<details>
<summary>Stata</summary>

```stata
* Read the data
use "census1900.csv", clear

* Rename all variables except histid to add _1900 suffix
foreach var of varlist * {
    if "`var'" != "histid" {
        rename `var' `var'_1900
    }
}

* Rename histid to match crosswalk
rename histid histid1900

* Save temporary file
tempfile census1900_temp
save `census1900_temp'

* Read and prepare crosswalk
use "1900_1930.csv", clear

* Merge with 1900 census
merge 1:1 histid1900 using `census1900_temp', keep(master match) nogen

* Save temporary merged file
tempfile merged_temp
save `merged_temp'

* Read and prepare 1930 census
use "census1930.csv", clear

* Rename all variables except histid to add _1930 suffix
foreach var of varlist * {
    if "`var'" != "histid" {
        rename `var' `var'_1930
    }
}

* Rename histid to match crosswalk
rename histid histid1930

* Merge with previous data
merge 1:1 histid1930 using `merged_temp', keep(master match) nogen

* Example: Study occupational transitions
tab occupation_1900 occupation_1930
```
</details>


## Version history
### Version 2.0 (February 2025)
Updated links including
- Links to the 1950 census
- Improvements to the linking algorithm

### Version 1.0 (November 2023)
Preliminary set of links shared with individual research teams for validation purposes.
