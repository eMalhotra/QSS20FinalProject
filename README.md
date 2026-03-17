# The Digital Fingerprint of Gentrification: How 311 Calls Reflect Changes in Race and Income in Three Disparate Brooklyn Neighborhoods

**Group Members:**
Anfisa Kryvtsun, Emelia Malhotra, Vidhi Piparia

---

## Introduction

We wanted to better understand how non-emergency service needs vary across neighborhoods, and specifically, how the process of gentrification may play a role. To explore this, we analyzed 311 call records (NYC Open Data), as well as demographic data (U.S. Census Bureau), for three Brooklyn neighborhoods with distinctly different economic trajectories. Using an observation period of 2012-2022 (inclusive), we tracked essential and non-essential complaints in Brownsville (consistently low-income, hovering around the poverty line), Bushwick (rapidly gentrified before and during this period), and Park Slope (stably affluent for decades). Complaints were split into survival needs and quality of life issues using both manual and automated sorting methods. By comparing these areas, we aim to measure the shifting needs of neighborhoods in demographic transition.

---

## Methods

**Neighborhood Selection:** Analyzed demographic trends in Brooklyn by zip code tabulation area in order to select and compare a consistently low-income neighborhood, a stably high wealth neighborhood, and one that transitioned through gentrification.

**Data Categorization Strategies:**
- *Manual Keyword Based Sorting:* Used specific dictionaries to map raw complaint type strings into 10 categories that could then be divided into "Survival" (housing & buildings, public spaces and safety, sanitation, environmental concerns) and "Quality of Life" (noise, nuisances, streets and traffic, parks and nature, business and consumer, finance or legal disputes, schools).
- *Latent Dirichlet Allocation (LDA):* Employed topic modeling to discover latent themes in complaint description data, providing a statistically driven alternative to the manually categorized records.

**Normalization & Feature Engineering:**
- Created a unique ZIPYEAR variable for use as a key when merging datasets and for clean comparison and plotting over time.
- Calculated complaint resolution time (issue duration) in order to filter out error entries and measure agency responsiveness.
- Normalized complaint counts by population to create realistic rates unaffected by sheer population density differences.

**Statistical Modeling:**
- Performed correlation analysis between demographic shifts (Gini index, racial makeup, median income) and complaint ratios.
- Utilized OLS regression to test the impact of income growth on the volume of quality of life reporting (non essential service requests).

---

## Hypotheses

1. As a neighborhood undergoes gentrification, there is a shift in the types of complaints called into 311. Survival type complaints decrease, while quality of life concerns rise.
2. Wealthier neighborhoods have more quality of life concerns (and less survival concerns) than lower income neighborhoods in the same borough.

---

## Directory for Data Storage and Computation

```
├── data/
│   ├── brooklyn_selected_zips.csv # Consolidated year-over-year demographic data
│   ├── cleaned311.csv             # 311 data, saved after all cleaning and counts
│   ├── merged.csv                 # The census & 311 data, merged on ZIPYEAR
│   ├── LDAdata.csv                # 311 data sorted using an LDA model, merged with census 
│   └── other_data_link            # Contains a drive link containing our large data files
│      ├── raw311_2012-2019.csv    # Raw downloaded 311 data for our 3 zip codes, 2012-2019 
│      ├── raw311_2021-2022.csv    # Raw downloaded 311 data for our 3 zip codes, 2020-2022 
│      ├── combined311.csv         # Cleaned and categorized 311 data by ZIPYEAR, 2012-2022
│      └── preCategories.csv       # Cleaned 311 data prior to categorization, for LDA model use
│   
├── scripts/
│   ├── QSS20_clean_combine.ipynb  # First round of 311 cleaning, merges all years
│   │                                 ← input: raw311_2012-2019.csv & raw311_2020-2022.csv
│   │                                 ← output: combined311.csv
│   ├── census.ipynb               # Scrapes census data, compares NYC zips, saves 3 zips 
│   │                                 ← output: brooklyn_selected_zips.csv
│   ├── merge.ipynb                # Fully cleans 311 data and merges it with census data
│   │                                 ← input: combined311.csv, brooklyn_selected_zips.csv
│   │                                 ← output: cleaned311.csv, preCategories.csv, merged.csv
│   ├── categorization2.ipynb      # Applies LDA categorization and compares to manual work
│   │                                 ← input: preCategories.csv, brooklyn_selected_zips.csv 
│   │                                 ← output: LDAdata.csv
│   └── analysis.ipynb             # Runs analyses on the data and creates plots
│                                     ← input: merged.csv 
│  
├── figures/
│   ├── fig01_longitudinal_trends.png        
│   ├── fig02_complaint_composition.png    
│   ├── fig03_income_vs_complaints.png        
│   ├── fig04_heatmap_qol_survival.png                      
│   ├── fig05_coef_plot.png                                        
│   ├── fig06_underreporting_gap.png                               
│   ├── fig07_bushwick_gentrification.png                          
│   ├── fig08_correlation_matrix.png                               
│   ├── fig09_LDA_longitudinal_trends.png           
│   ├── fig10_LDA_underreporting_gap.png                                  
│   ├── fig11_LDA_income_vs_complaints.png                                
│   ├── fig12_LDA_comparison_correlation.png  
│   ├── tab01_summary_statistics.png                            
│   ├── tab02_panel_results_full.png                            
│   ├── tab03_LDA_QoL_predictors_full.png    
│   ├── tab04_LDA_descriptive_statistics.png                              
│   ├── tab05_LDA_income_impact_regression.png                                                        
│   └── tab06_LDA_neighborhood_averages_by_zip.png                              
│  
└── README.md
```

---

## Data Preprocessing

### Census Data

**Goals:**
- Gather annual demographic data (specifically, variables that are known to reflect gentrification processes, such as race and income) at the 5-digit zip code level for all zip codes in New York City.
- Analyse changes in income over our observation period (2012-2022 inclusive) and select candidate zip codes for further exploration.
- Create and save a dataframe containing the relevant demographic data for each selected zip code.

**Tool:** Visual Studio Code, Python, By-Request Census API Key

**Datasets:**
- Raw Data: https://www.census.gov/data/developers/data-sets/acs-5year.html
- Cleaned Data: `brooklyn_selected_zips.csv`

**Processing steps, as seen in `census.ipynb`:**
- *API Extraction:* Pulled 20+ variables including median household income, poverty rate, and racial composition.
- *Error Handling:* Identified and replaced Census-specific error codes (e.g. -666666666) with NaN values to avoid affecting averages.
- *Data Formatting:* Converted ZIP codes and Year values to standard string/integer formats to create a key for consistent merging.
- *Export:* Saved a cleaned year over year dataframe to `brooklyn_selected_zips.csv`

### 311 Data

**Goals:**
- Clean the massive dataset, extracting only potentially relevant variables for the three selected zip codes.
- Prepare all variable columns for year by year analysis (sort and categorize complaint types and agencies, calculate duration averages).
- Create and save a dataframe that merges the annual 311 variable measurements with the Census data.

**Tool:** Visual Studio Code, Python

**Datasets:**
- Raw Data: `raw311data.csv`
  - In order to access the full raw dataset, please go to https://drive.google.com/drive/folders/1jlRPKzeqzUxn2A3depW_ZSNS9aE4w93p?usp=share_link
- Cleaned Data: `cleaned311.csv`, `preCategories.csv`
  - In order to access `preCategories.csv`, please go to https://drive.google.com/drive/folders/1jlRPKzeqzUxn2A3depW_ZSNS9aE4w93p?usp=share_link
- Merged Data: `merged.csv`, `LDAdata.csv`

**Processing Steps [in `merge.ipynb`, `QSS20_clean_combine.ipynb`, and `categorization2.ipynb`]:**
- *Initial Data Joining:* Merged and reordered 11 annual datasets into one chronologically combined file.
- *Column Cleaning:* Removed a number of irrelevant columns and renamed the remaining ones, transforming the titles from strings to easily referenced variables.
- *Temporal Engineering:* Extracted year values from timestamps and calculated the average duration (resolution time) of complaints in minutes, hours, and days.
- *Aggregating:* Grouped data by zip code and year in order to track year-over-year changes in accordance with census data.
- *Manual Mapping:* Identified all unique complaint descriptions and created conditional logic to sort complaints by type and agency.
- *Agency Categorization:* Created a dictionary based on existing standards to distribute agencies (tasked with responding to given complaints) into 6 unique categories: education and schools (doe); public safety (safety); infrastructure and environment (infr_env); social services, health, and housing (ss_hous); internal units such as finance, tax, and property (fin_prop); and business and tech (biz_tech).
- *Complaint Type Indicators:* Used keyword matching to sort complaint instances into one of 10 unique categories: housing and buildings (housing_building); noise, nuisances, and quality of life (noise_qol); sanitation and environmental (sanitation_env); streets, sidewalks, and traffic (street_traffic); public safety and policing (public_safety); parks and trees (parks_nature); finance and property (finance_legal); business and consumer affairs (business_consumer); technology and infrastructure (tech_infrastructure); and schools (education).
- *NLP Refinement:* Processing in `categorization2.ipynb`:
  - Note: `categorization2.ipynb` relies on steps 0-2bii in `merge.ipynb`, and uses the dataset `preCategories.csv` saved in step 2bii of `merge.ipynb`.
  - *Text Preparation:* Used spacy to tokenize and lemmatize complaint descriptions.
  - *Topic Modeling:* Ran an LDA (Latent Dirichlet Allocation) model via gensim to find latent complaint themes.
  - *Comparison:* Statistically compared findings using the LDA sorted data against the manually sorted data to ensure findings remained consistent across varying categorization methods.

---

## Analyses

**Analysis 1 — Census Data Collection and Demographic Trends**
- *Variables:* ZIP code, year, median household income, racial composition, population.
- *Findings:* Using the Census API, we scraped demographic and socioeconomic data from the ACS 5-year estimates for NYC ZIP Code Tabulation Areas. These data allow us to observe changes in income and demographics over time across neighborhoods.

**Analysis 2 — Neighborhood Selection Based on Income and Demographic Patterns**
- *Variables:* ZIP code, median household income, income growth.
- *Findings:* Exploratory analysis of census data identified neighborhoods with distinct economic trajectories. Brownsville emerged as a consistently low-income neighborhood, Park Slope as a stable high-income neighborhood, and Bushwick as a neighborhood experiencing significant income growth and demographic change.

**Analysis 3 — Cleaning and Aggregating NYC 311 Complaint Data**
- *Variables:* complaint type, ZIP code, creation date.
- *Findings:* The NYC 311 dataset contains millions of records, so we removed unnecessary columns and filtered the data to the three selected ZIP codes. Complaint timestamps were converted to a year variable, allowing complaints to be aggregated by ZIP code and year.

**Analysis 4 — Construction of ZIP-Year Panel Dataset**
- *Variables:* ZIP code, year, ZIPYEAR identifier.
- *Findings:* A composite ZIPYEAR variable was created to merge the aggregated 311 complaint data with the Census demographic dataset. This produced a panel dataset containing complaint counts and neighborhood characteristics for each ZIP code and year.

**Analysis 5 — Topic Modeling and Complaint Categorization**
- *Variables:* complaint type text.
- *Findings:* Because the 311 dataset includes hundreds of complaint types, we used topic modeling with the Gensim LDA model to identify clusters of related complaint types. The resulting topics corresponded to broad issue categories such as housing problems, sanitation concerns, park or safety issues, and noise complaints.

**Analysis 6 — Categorized Complaint Counts by ZIP-Year**
- *Variables:* complaint category, ZIP code, year.
- *Findings:* Complaints were assigned to thematic categories and aggregated using pandas.crosstab. These categorized complaint counts were merged with demographic variables to create a dataset linking complaint behavior to neighborhood socioeconomic characteristics.

**Analysis 7 — Longitudinal Complaint Trend Analysis**
- *Variables:* complaint category counts, ZIP code, year.
- *Findings:* Time-series visualizations show clear differences in complaint composition across neighborhoods. Brownsville consistently shows higher counts of survival-related complaints such as housing maintenance and infrastructure issues, while Park Slope exhibits higher levels of quality-of-life complaints.

**Analysis 8 — Quality-of-Life vs Survival Complaint Ratio**
- *Variables:* QoL complaint counts, survival complaint counts.
- *Findings:* We calculated the ratio of quality-of-life complaints to survival complaints to capture changes in neighborhood priorities. Bushwick's ratio increases over time, reflecting a shift toward lifestyle complaints consistent with the neighborhood's ongoing gentrification.

**Analysis 9 — Regression Analysis of Demographic Predictors**
- *Variables:* complaint rates, median household income, percent White non-Hispanic population, inequality indicators.
- *Findings:* Regression models estimated using statsmodels suggest that higher income levels and demographic shifts are associated with increased quality-of-life complaint rates. Survival complaint rates appear more strongly associated with lower-income neighborhoods.

---

## Results

In the interest of simplicity, all eighteen graphs and tables can be found in the `/figures` branch of our repository.

---

## Discussion

### Challenges and Limitations

**a. Small sample size**

Our analysis focuses on only three neighborhoods, which limits the number of observations available for regression analysis. Because of this small sample size, we do not conduct formal statistical significance testing and instead focus on descriptive trends.

**b. Complexity of complaint classification**

The NYC 311 dataset contains hundreds of complaint types that vary across agencies. Although topic modeling and manual validation helped group similar complaints, some classification uncertainty remains.

**c. Reporting bias in 311 data**

311 complaints represent reported issues rather than actual conditions. Residents in lower-income neighborhoods may be less likely to report problems due to lower awareness of city services or lower expectations of response.

**d. ACS rolling averages**

Demographic data from the ACS represents 5-year rolling averages, meaning demographic shifts appear gradual even if changes occurred rapidly.

### Future Work

**x. Expanding geographic coverage**

Future work could include additional ZIP codes across New York City to increase the sample size and improve statistical power.

**y. Improved natural language processing**

More advanced NLP methods could be used to improve complaint categorization and capture subtle differences between complaint types.

**z. Incorporating additional urban data**

Integrating datasets such as housing price indices, rent burden statistics, or eviction filings could help better understand the mechanisms linking gentrification to changes in complaint behavior.

**w. Causal identification strategies**

Future research could apply causal inference methods such as difference-in-differences or event study designs to better isolate the impact of gentrification on complaint patterns.

---

## Related Work

- **Legewie & Schaeffer (2016)** — Argues that 311 complaints function as instruments of boundary-making rather than neutral infrastructure reports, demonstrating that racial and socioeconomic demographic shifts in NYC neighborhoods produce spikes in noise and quality-of-life complaints that cannot be explained by underlying conditions alone.

- **Legewie (2024)** — A longitudinal update refining the 2016 findings, showing that 311-recorded conflict peaks not in the most diverse neighborhoods but in those experiencing the fastest demographic transition, directly supporting our framing of Bushwick (11237) as a site of visible social friction.

- **Beck (2020)** — Identifies "development-directed policing," a pattern in which surges in 311 calls to police correlate with rising property values and incoming middle-class residents who deploy the complaint system to clear public spaces of behaviors they perceive as disorderly.

- **Stolper (2024)** — Research for the Community Service Society finding that quality-of-life 311 calls in gentrifying neighborhoods of color are three times more likely to result in police summons or arrests than in stably affluent areas, establishing 311 as a gateway mechanism for police intervention that disproportionately affects long-term residents.

- **Kontokosta & Hong (2021)** — NYU Marron Institute research demonstrating that residents in high-poverty areas report at lower rates despite higher levels of infrastructure decay, meaning 311 data systematically under-represents survival-category issues in neighborhoods like Brownsville (11212) and our counts there should be read as conservative floors.

- **Zukin et al. (2009)** — Sociological account of how incoming residents leverage social capital to reshape public space through "new retail capital," providing the theoretical grounding for why lifestyle and social-ordering complaint categories rise in transitioning Brooklyn neighborhoods.

- **NYU Furman Center (2024)** — Longitudinal tracking of rent burden and housing code violations confirming that rent-burdened, high-poverty zip codes concentrate 311 activity in essential survival categories (heat, structural maintenance), while affluent and transitioning areas skew toward social-ordering complaints, the empirical pattern our QoL-to-Survival ratio is designed to capture.

---

## References

Beck, Brenden. 2020. "As Neighborhoods Gentrify, Police Presence Increases." *Housing Matters*, Urban Institute.

Kontokosta, Constantine E., and Boyeong Hong. 2021. "Estimating Reporting Bias in 311 Complaint Data." Technical Report, NYU Marron Institute of Urban Management.

Legewie, Joscha. 2024. "Digital Traces of Urban Conflict: A 10-Year Analysis of 311." *American Sociological Review*.

Legewie, Joscha, and Merlin Schaeffer. 2016. "Contested Boundaries: Explaining Where Ethnoracial Diversity Provokes Neighborhood Conflict." *American Journal of Sociology* 122(1): 125–161.

NYU Furman Center. 2024. "State of New York City's Housing and Neighborhoods in 2023."

Stolper, Harold. 2024. "New Neighbors and the Over-Policing of Communities of Color." Technical Report, Community Service Society of New York.

Zukin, Sharon, Valerie Trujillo, Peter Frase, Philip Kasinitz, and Xiangming Chen. 2009. "New Retail Capital and Neighborhood Change: Boutiques and Gentrification in New York City." *City & Community* 8(1): 47–64.
