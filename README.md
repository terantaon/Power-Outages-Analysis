# Power Outages Analysis
A project for DSC 80 at UCSD!!!

Author: Trent Gong

## Introduction
In this project, I will study a data set of power outages from Purdue University’s Laboratory for Advancing Sustainable Critical Infrastructure, at [this website](https://engineering.purdue.edu/LASCI/research-data/outages). The data set convers major outages witnessed by different states in the continental U.S. during January 2000–July 2016, as well as the data on geographical location of the outages, date and time of the outages, regional climatic information, land-use characteristics, electricity consumption patterns and economic characteristics of the states affected by the outages.

The Department of Energy defined the major outages as those that impacted atleast 50,000 customers or caused an unplanned firm load loss of at least 300 MW.

First, I will explore various features in the data set. Then I will analyze their missingness mechanisms, and how they are correlated or distributed.

Last but not least, I will pay special attention on the cause of power outages. More detailly, I will build a model that predicts the cause of a power outage using existing information. This model is expected to help energy companies better prevent power outages and improve their efficiency.

The orginal data set contains 1534 rows and 55 columns, but only these features are related to our study:

| Column                | Description                                                    |
|:--------------------- |:-------------------------------------------------------------- |
|YEAR                   |Year when the outage event occurred                             |
|MONTH                  |Month when the outage event occurred                            |
|POSTAL.CODE            |Postal code of the states                                       |
|NERC.REGION            |NERC regions involved in the outage event                       |
|CLIMATE.REGION         |U.S. Climate regions                                            |
|ANOMALY.LEVEL          |The oceanic El Niño/La Niña (ONI) index                         |
|OUTAGE.START.DATE      |The day of the year when the outage event started               |
|OUTAGE.START.TIME      |The time of the day when the outage event started               |
|OUTAGE.RESTORATION.DATE|The day of the year when power was restored to all the customers|
|OUTAGE.RESTORATION.TIME|The time of the day when power was restored to all the customers|
|CAUSE.CATEGORY         |Categories of all the events causing the major power outages    |
|OUTAGE.DURATION        |Duration of outage events                                       |
|DEMAND.LOSS.MW         |Amount of peak demand lost during an outage event               |
|CUSTOMERS.AFFECTED     |Number of customers affected by the power outage event          |
|TOTAL.PRICE            |Average monthly electricity price in the state                  |
|TOTAL.SALES            |Total electricity consumption in the state                      |
|TOTAL.CUSTOMERS        |Annual number of total customers served in the state            |
|POPULATION             |Population in the state in a year                               |
|POPPCT_URBAN           |Percentage of the urban population in state                     |

## Data Cleaning and Exploratory Data Analysis
### Data Cleaning
1. In the original excel file, the column names are on row 5, the data set starts at row 7, and the first column is useless. Thus after reading the file as a dataframe `df`, I use row 5 as the header and drop the first row.
2. Only the features above are related to my study, so I only keep these columns in `df`.
3. I combine the `OUTAGE.START.DATE` and `OUTAGE.START.TIME` columns into one column `OUTAGE.START`. I also combine `OUTAGE.RESTORATION.DATE` and `OUTAGE.RESTORATION.TIME` into one column `OUTAGE.RESTORATION`. Then the original columns are dropped.
4. Notice that some of the values in `OUTAGE.DURATION`, `CUSTOMERS.AFFECTED` and `DEMAND.LOSS.MW` columns are 0. This does not make sense because a major power outage is unlikely to last 0 minutes, affect 0 customers and cause 0 MW of energy lost. Therefore I assume these values are missing and I replace them with `np.nan`. Also notice that some of the time in `OUTAGE.RESTORATION` is equal to the time on the same row in `OUTAGE.START`. Since major power outage is unlikely to restore at once, I also replace these values with `np.nan`.
5. There are 9 rows where `MONTH`, `ANOMALY.LEVEL`, `OUTAGE.DURATION`, `TOTAL.PRICE`, `TOTAL.SALES`, `OUTAGE.START`, `OUTAGE.RESTORATION` are all missing. I assume these values are MCAR, perhaps due to unexpected errors during data collection or storage. These are only 0.5% of my data which should not greatly affect my study, so for convenience I'll just drop these rows.
6. I add two features to `df`: `HOUR` derived from `OUTAGE.START` that represents the hour when outages start, and `DAY` derived from `HOUR` that represents whether outages start at day or night.

Here are the first 5 rows of some columns of my cleaned `df`:

|   YEAR |   MONTH | CAUSE.CATEGORY     |   CUSTOMERS.AFFECTED | OUTAGE.START        |   HOUR | DAY   |
|-------:|--------:|:-------------------|---------------------:|:--------------------|-------:|:------|
|   2011 |       7 | severe weather     |                70000 | 2011-07-01 17:00:00 |     17 | True  |
|   2014 |       5 | intentional attack |                  nan | 2014-05-11 18:38:00 |     18 | False |
|   2010 |      10 | severe weather     |                70000 | 2010-10-26 20:00:00 |     20 | False |
|   2012 |       6 | severe weather     |                68200 | 2012-06-19 04:30:00 |      4 | False |
|   2015 |       7 | severe weather     |               250000 | 2015-07-18 02:00:00 |      2 | False |


### Univariate Analysis
First, I want to see how many outages occur in each month.
<iframe
  src="assets/fig1.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>
As we can see, the number of power outages occur in winter and summer is relatively higher. This fits with our understanding that energy consumption increases in summer and winter due the demand for air conditioning and heating.

### Bivariate Analysis
I analyze how the number of customers affected is distributed over cause category.
<iframe
  src="assets/fig3.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>
The graph suggests that power outage caused by severe weather or system operability disruption tend to affect much more customers.

### Interesting Aggregates
I groupee by climate region and cause category to see if there are any difference in the distributions of causes over climate region(some columns are hidden).

| CLIMATE.REGION     |   equipment failure |   intentional attack |   public appeal |   severe weather |   system operability disruption |
|:-------------------|--------------------:|---------------------:|----------------:|-----------------:|--------------------------------:|
| Central            |                   7 |                   38 |               2 |              134 |                              11 |
| East North Central |                   3 |                   20 |               2 |              104 |                               3 |
| Northeast          |                   5 |                  135 |               4 |              176 |                              14 |
| Northwest          |                   2 |                   89 |               2 |               29 |                               4 |
| South              |                   9 |                   28 |              42 |              112 |                              27 |
| Southeast          |                   4 |                    9 |               5 |              116 |                              16 |
| Southwest          |                   5 |                   64 |               1 |               10 |                               9 |
| West               |                  21 |                   31 |               9 |               70 |                              41 |
| West North Central |                   1 |                    4 |               2 |                4 |                             nan |

## Assessment of Missingness

### NMAR Analysis
One of the column that may be NMAR is DEMAND.LOSS.MW. I googled that the local utility companies are responsible for collecting data of the loss of energy during a major power outage. These companies may have specific rules for data collection, for instance, they may not report data for relatively small sized outages. If they did not report the energy loss in a power outage, this value would be missing. Therefore, an additional data I might want is the name of the local utility companies. Then I can perform a permutation test to see if DEMAND.LOSS.MW is MAR, depending on my new data.

### Missingness Dependency
I tested if the missingness of OUTAGE.RESTORATION depend on MONTH and CLIMATE.REGION.

1.
**Null Hypothesis**: The distribution of Month is the same when Outage Restoration is missing vs not missing.
**Alternate Hypothesis**: The distribution of Month is different when Outage Restoration is missing vs not missing.
**Test Statistic**: TVD
<iframe
  src="assets/fig5.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>
I got an observed statistic of 0.143. After running the permutation test, I got a p-value of 0.194. Therefore, I fail to reject the null hypothesis in favor of the alternate hypothesis. This result suggests that the distribution of Month is different when Outage Restoration is missing vs not missing, which means that the missingness of Outage Restoration is not dependent on Month.

## Hypothesis Testing
H0: The mean duration of outages in California is equal to the mean duration of outages nationwide.  
Ha: The mean duration of outages in California is less than the mean duration of outages nationwide.  
Test statistic: The difference in means.

## Framing a Prediction Problem

## Baseline Model

## Final Model

## Fairness Analysis