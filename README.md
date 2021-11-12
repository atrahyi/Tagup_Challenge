# Tagup_Challenge
Tagup Data Engineer challenge, submitted by Benjamin Lampert

This is my submission for the technical challenge for the Data Engineering position at Tagup Inc. The challenge involved reading in example data from a database, cleaning it up, and transforming it to an array format which is more useful as input to Tagup's ML pipelines.

# Explanation of code
I first used Python's sqlalchemy library to load in the tables from the exampleco database. I then used pandas to read those tables into a csv. There were 5 tables: a "static_data" table and 4 "feature" tables. I merged the feature tables into a dataframe and used describe() to generate its summary statistics. From the quartile values alone, it was already clear that most machines had failed and that distribution of each of the features has fat tails. However, these summary statistics can be deceptive when trying to identify outliers, especially considering that we know there is heterogeneity in machine behavior (i.e. different types of behavior which draw from different distributions). To check for outliers, I first visaully skimmed through the data

The methodology used to create this table is not perfect. It misses out on some valuesHowever, despite its flaws, I think that this table can be useful for finding the exact point at which a machine becomes faulty and breaks, the former of which is not necessarily clear from the time series. It could also be useful for judging the behavior of new machines which were not included in this database.

# Findings
Data is collected at regular intervals of a little over 8 hours.

I found that "normal" behavior is characterized by a quasi-sinusoidal behavior pattern

It seems likely that this "noise" follows the same or a very similar distribution to the "noise" which characterizes the measurements in the "broken" state, though I have not yet tested this.

|Machine | Number of periods before breaking |
|---|---|
| 0 | 397 |
| 1 | 1738 |
| 2 | 1780 |
| 3 | 111 |
| 4 | 63 |
| 5 | 397 |
| 6 | 624 |
| 7 | 1466 |
| 8 | 894 |
| 9 | 748 |
| 10 | 218 |
| 11 | 269 |
| 12 | 655 |
| 13 | 590 |
| 14 | 407 |
| 15 | 1778 |
| 16 | 605 |
| 17 | 695 |
| 18 | 426 |
| 19 | 51 |

Machines of Model A lasted periods before breaking

I found these values by inspection, but it would not be hard 
