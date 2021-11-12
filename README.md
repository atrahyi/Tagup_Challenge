### Tagup_Challenge
Tagup Data Engineer challenge, submitted by Benjamin Lampert

This is my submission for the technical challenge for the Data Engineering position at Tagup Inc. The challenge involved reading in example data from a database, cleaning it up, and transforming it to an array format which is more useful as input to Tagup's ML pipelines.

## Explanation of code

#Identifying outliers

I first used Python's sqlalchemy library to load in the tables from the exampleco database. I then used pandas to read those tables into a csv. There were 5 tables: a "static_data" table and 4 "feature" tables. I merged the feature tables into a dataframe and used describe() to generate its summary statistics. From the quartile values alone, it was already clear that most machines had failed and that distribution of each of the features has fat tails. However, these summary statistics can be deceptive when trying to identify outliers, especially considering that we know there is heterogeneity in machine behavior (i.e. different types of behavior which draw from different distributions). To check for outliers, I first visually skimmed through the data both in table form and in density plots. For each of the features, although nearly all of the observations were below 100 in absolute value, I found clusters of observations around -250 and +250 which I assume must be erroneous observations. About 5% of observations for each feature was an outlier of this kind, though there were no obvious patterns to their occurrence. I replaced any observations with absolute value above 200 with NaNs and found that the min and max of the remaining observations (for any feature) come out to be -116 and 119 respectively, making it unlikely that I discarded any reliable observations. I examined the density plots of the new data and, although there were long tails, I was hesitant to discard any more observations considering that behavior is supposed to be chaotic during “faulty” states.

#Choosing schematic

The schematic I chose to represent the feature data is a 3D array where the dimensions represent timestamp, machine, and feature. I found this to be the most compact and intuitive way to represent the data, and slicing in any dimension either gives you a time-series for a certain machine, a time-series for a certain feature, or a cross-sectional analysis of all machine-feature pairs, all of which seem useful for analysis.

With this in mind, I converted the dataframe into an appropriate 3D array. To do this, I first widened the data so that each row represented a timestamp and the columns were machine-feature pairs. I then reordered the columns so that all of the features for a given machine were together, although I could have just as easily reshaped to include feature as the second dimension and machine as the third. I converted the wide dataframe to an array with shape (3000, 80) and then reshaped it to add a third dimension which separated machine-feature pairs into machine x feature crosses. I then created a DataArray called “inputs” from this 3D array and labelled the dimensions “timestamp”, “machine”, and “feature”. I included coordinate labels as well which mimic the row labels on the original dataframe.

#Visualizations

To demonstrate the efficacy of my approach, I generated visualizations for each of the cross-sections of “inputs” (although I did not use “inputs” directly to generate them as I have never worked with xarray before; given more time I would have done that). First, I looked at machine 0’s time series data for each of the features to get a sense of what normal, faulty, and broken behavior looked like and to see what happened when a machine transitioned states. Then, I generated the time series data for the first feature for each machine to compare normal, faulty, and broken performances. Finally, I looked at a few cross-sections of the data: one at t = 0 (i.e. the first time period) to see how many machines were working at the beginning, one at t = 701 (i.e. the 702nd time period) - a period when there were normal, faulty, and broken machines - and one at t = 2800, where all of the machines were broken. Noticing similarities in machines’ “normal” behavior (see “Findings” below), I created a table characterizing the “normal” behavior of machines by rounding values to the nearest 10th, dropping “broken” values near 0, and finding the mode of the remaining values to remove “faulty” behavior. To ensure that “faulty” behavior did not show up on this table, I added the condition that the mode must occur at least twice in the dataset. The methodology used to create this table is not perfect. For instance, if only one machine is operating normally, it will not capture those data points. However, despite its flaws, I think that this table can be useful for finding the exact time at which a machine becomes faulty and breaks, the former of which is not necessarily clear from the time series. It could also be useful for judging the behavior of new machines which were not included in this database.

##Findings

#Data

The data contains 3000 observations of 4 features for a fleet of 20 machines, 14 of which are of Model “Model A” and 6 of which are of model “Model B”. Model A machines are located in Room 123 and Model B machines are in Room 456. Observations are equally spaced out in time at intervals of just over 8 hours.

# Characterization of behavior

The machines have 3 states of behavior: “normal”, “faulty”, and “broken”.

I found that "normal" behavior is characterized by a quasi-sinusoidal behavior pattern for each of the features. The amplitudes are about 40 for feats 0 & 2, 10 for feat 1, and 25 for feat 3. As the machine goes into "faulty" state, the behavior becomes more erratic and amplitude increases drastically for all features except feature 2, where the range tends to stay about the same but the behavior becomes less regular. Two of the machines in the sample - machines 8 and 10 - never enter “faulty” mode (at least visually; to rigorously conclude this I would need to confirm this with the “normal-behavior” table) and instead go straight from “normal” to “broken”. What’s more interesting is that all of the machines in “normal” mode exhibit almost the exact same behavior, as can be seen in cell 19 (note that machines 4 and 19 start in “faulty” mode and break soon thereafter. The only differences between them is noise on the order of 10^-2. It seems likely that this "noise" follows the same or a very similar distribution to the "noise" which characterizes the measurements in the "broken" state, though I have yet to test this.

I did, however, examine the distribution of “broken” observations by zooming in on values near 0. While this methodology is imperfect since “normal” behavior can show up (given more time I’d find out when each machine breaks and then use observations after that ), the biases induced by it appear to be minimal since the density plots for the features looks almost identical in cell 14 despite apparent differences near 0 in cell 13 (see the code comments for a more in-depth explanation). I found that the noise in broken observations was a similar, normal-looking distribution for each of the features with mean 0 and a standard deviation of about 0.01. I have not yet looked into characterizing “faulty” behavior beyond visual descriptions.

#Machine Performance

Every machine broke by the end of September 2020. Below is a table of the number of time periods each machine lasted before braking (i.e. were either in normal or faulty mode)


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


I found these values by inspection, but given more time I would use the “normal-behavior” table to identify periods of normal, faulty, and broken behavior for each machine. The reason I did not do that right now is due to the aforementioned flaws in the methodology used to construct the table.

Machines of Model A lasted an average of 765.4 periods before breaking, while machines of Model B lasted 660.3 periods on average.

