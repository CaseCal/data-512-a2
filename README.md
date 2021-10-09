# Bias in Wikipedia Article Quality

## Goal
This project examines the variation in article quality for English Wikipedia articles about politicians. Does the quality of articles vary depending on the nationality and cuulture of the subject?

## Setup

### Environment
All scripts are run in Jupyter Notebook using python 3.7.9. The full list of dependencies can be found in the requirements.txt file

### How to Run
All code used is contained in the hcds-a2-bias.ipynb notebook. Running all cells will collect and process all data. See the description in the notebook to skip the data acquisition or processing stages.

### Stages

#### 1. Data Acquisition
Page data from 2017 is downloaded as a csv from the Figshare repository. Population data is downloaded as a csv from the Population Reference Bureau. Both files are contained in the *data/raw* folder of this repository, and are simply loaded in this step. See [Data](#data) section below for details on format and licensing.

#### 2. Data Processing

##### Cleaning and Combining
The page data is filtered to remove template pages, by removing all page names that start with "Template:". The population data is filtered on Type = Country to exclude the regional and worldwide data. The data is then merged, matching the "Name" field in the population data with the "country" field in the page data.

Entries that could not be matched are removed and saved in the *data/unmatched* folder. Overall, 26 countries had no matching articles, and 1859 pages had no matching country. Unmatched pages appeared to be primarily those where teh country was inconsistently encoded as an adjective, such as "South Korean", "Salvadoran", and "Icelandic". This analysis could be improved by attempting to match more of these articles.


## Data

### Page Data

#### Source
Page Data is collected from [this](https://figshare.com/articles/dataset/Untitled_Item/5513449) Figshare repository on 10/09/2021. Is is shared under the CC-BY-SA 4.0 License

#### Description
This data can be found in the *page_data.csv* file in the *data* folder. It contains three columns:
1. "country": Santized country name.
1. "page": Unsanitized page name.
1. "last_edit": Edit ID of last page edit.

#### Notes
Although the repository was accessed in October 2021, it was posted in October 2017.

### Population Data

#### Source
Population data is collected on 10/09/2021 from the [World Population Data Sheet](https://www.prb.org/international/indicator/population/table/) hosted by the [Population Reference Bureau](https://www.prb.org/). This file contains national population estimates for 2020, "based on a recent census, official national data, or analyses conducted by national statistical offices, regional organizations, PRB, UN Population Division, or International Programs of the U.S. Census Bureau."

#### Description
This data can be found in the *WPDS_2020_data* file in the *data* folder. It contains three columns:
1. "FIPS": The [Federal Information Processing Standards](https://www.nist.gov/standardsgov/compliance-faqs-federal-information-processing-standards-fips) country code.
1. "Name": Country name.
1. "Type": Level of aggregration, one of "World", "Sub-Region", or "Country".
1. "TimeFrame": Year of estimate.
1. "Data (M)": Estimated population in millions.
1. "Population": Estimated population.

#### Notes
The date that the estimates were made and the specific source for each population count are unavailable. No license was included with the data.


