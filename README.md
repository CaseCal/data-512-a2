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
The page data is filtered to remove template pages, by removing all page names that start with "Template:". A new column is added to the population data containing the region, based on the most recent row in the dataset with Type of Sub-Region. Note that the row for Channel Islands has to be modified to be a Country instead of Sub-Region, otherwise all of Northern Europe becomes grouped under it. The population data is then filtered on Type = Country to exclude the regional and worldwide rows. The data is then merged, matching the "Name" field in the population data with the "country" field in the page data.

Entries that could not be matched are removed and saved in the *data/unmatched* folder. Overall, 27 countries had no matching articles, and 1859 pages had no matching country. Unmatched pages appeared to be primarily those where teh country was inconsistently encoded as an adjective, such as "South Korean", "Salvadoran", and "Icelandic". This analysis could be improved by attempting to match more of these articles.

##### ORES Quality Predictions
The [ORES API](https://ores.wikimedia.org/v3/#) is used to collect the quality ratings for each article. We use  the "GET /v3/scores/{context}" endpoint, providing "enwiki" as context, "articlequality' as model and providing rev_ids in batches of 50.

Out of 44680 articles, 274 did not return a prediction. These are saved in the *data/unmatched/page_data-no_prediction.csv* file, and removed from further analysis.

On a computer with a 7-core CPU, 24GB RAM and 10Mbps ethernet connection, the API requests took approximately two minutes to complete. 

### 3. Analysis
Analysis is done using two metrics. Analysis result tables can be found in the notebook.

1. **Coverage** The number of articles divided by the popuulation, shown in articles per million people. Does not take into consideration the quality of the articles.
1. **Relative Quality** The proportion of articles that are of GA (Good Article) or FA (Featured Article) quality.


#### Country
Data is grouped by country, and sorted. The top 10 countries by coverage are generally small populations, and are all in Oceania or Europe. The bottom countries by coverage are generally large populations, such as India and China, and are all in Asia. Overall, coverage is impacted by size, with smaller countries having better coverage, but also seems to show a regional bias towards Europe and against Asia.

Top countries with relative quality are more mixed, with North Korea at the top with 22% (8 of 36) articles of high quality, although this is liekly an outlier. Asia, Europe, and Africa are all respresented in the top 10. At the bottom, 37 countries tied with no high-quality articles. Sorting secondarily by article count provides the top 10 displayed, which are primarily North and Eastern Europe and Africa.

#### Region
Data is also aggregated by region. In this case, population is taken from the region's population data point in the original data source, and not as a sum of the popualtions in the final data file, in order to account for countries that had no articles.

Regional bias is more apparent here, with Oceania and Europe topping the list and Asia at the bottom.


## Data

### Page Data

#### Source
Page Data is collected from [this](https://figshare.com/articles/dataset/Untitled_Item/5513449) Figshare repository on 10/09/2021. Is is shared under the CC-BY-SA 4.0 License

#### Description
This data can be found in the *page_data.csv* file in the *data/raw* folder. It contains three columns:
1. "country": Santized country name.
1. "page": Unsanitized page name.
1. "last_edit": Edit ID of last page edit.

#### Notes
Although the repository was accessed in October 2021, it was posted in October 2017.

### Population Data

#### Source
Population data is collected on 10/09/2021 from the [World Population Data Sheet](https://www.prb.org/international/indicator/population/table/) hosted by the [Population Reference Bureau](https://www.prb.org/). This file contains national population estimates for 2020, "based on a recent census, official national data, or analyses conducted by national statistical offices, regional organizations, PRB, UN Population Division, or International Programs of the U.S. Census Bureau."

#### Description
This data can be found in the *WPDS_2020_data* file in the *data/raw* folder. It contains three columns:
1. "FIPS": The [Federal Information Processing Standards](https://www.nist.gov/standardsgov/compliance-faqs-federal-information-processing-standards-fips) country code.
1. "Name": Country name.
1. "Type": Level of aggregration, one of "World", "Sub-Region", or "Country".
1. "TimeFrame": Year of estimate.
1. "Data (M)": Estimated population in millions.
1. "Population": Estimated population.

#### Notes
The date that the estimates were made and the specific source for each population count are unavailable. No license was included with the data.

### Unmatched data
All data that was excluded from analysis due to missing matches is saved in the *data/unmatched* folder.

1. **page_data-no_match.csv** Contains pages that could not be matched to a country. Columns are the same as [Page Data](#page-data)
1. **wp_wpds_countries-no_match.csv** Contains countries that had no associated pages. Columns are the same as [Population Data](#population-data)
1. **wp_wpds_politicians-no_prediction.csv** Contains pages that did not return a quality predictions from the ORES API. Columns are the same as [Processed Data](#processed-data)

### Processed data
The remaining data after processing and removing missing values.

#### Description
1. "population": The country population from the [Population Data](#population-data).
1. "page": Name of the Wikkipedia article.
1. "country": Country that the politician in the referenced article belongs to.
1. "revision_id": The article revision id used to get the quality estimate.
1. "article_quality_est": Estimated article quality from ORES API. Values are in ascending order of quality 
 1. "stub"
 1. "start"
 1. "C"
 1. "B"
 1. "GA" = Good Article
 1. "FA" = Featured Article


## Considerations

### Potential Problems
1. As was noted, Channel Islands was incorrectly noted as a Sub-Region, and had to be manually corrected. It's possible that other such errors exist.
1. Some pages had inconsistent country names as noted in the [Cleaning and Combining](#cleaning-and-combining) stage, and were excluded from the final calculations.
1. Page data is from 2017, but ppopulation data is as-of 2019.
1. Page data may not be reliable. For example, pages called "Political Positions of Jeb Bush" and "Bush Family" are included, but pages for the actual politions George H.W Bush, George Bush, and Jeb Bush are not. It may be that people with multiple categories/careers are not being captured as "ppoliticians", and this is actually making the most robust articles be missed.

### Potential Next steps
1. Quality and Coverage could be compared or merged into a weighted calculation.
1. Some cleanup of page data could still be useful, as category and list pages are present.



