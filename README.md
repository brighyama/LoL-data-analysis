# Competitiveness of 2022 League of Legends World Championship Matches

by Brighten Hayama (bhayama@ucsd.edu)

## Introduction 

The League of Legends 2022 Competitive Match dataset [found here](https://oracleselixir.com/tools/downloads) contains individual and team data for most of the competitive match history of 2022. Many professional league data, such as LCS, LCK, LPL, and more can be found, and in particular, it contains all match data for the 2022 World Championship (WCS), which much of our analysis will be centered around.

We are concerned with the following question: "Are 2022 League of Legends World Championship Games More Competitive Than Regular Season Games?" Of course, it is difficult to quantify competitiveness of League of Legends matches because of the complexity and variability of matches. To simplify this question, we want to utilize the statistic, "Gold Difference at 15min", to show how close two teams are after 15 minutes into a match, which affects the remaining length and, typically, the result of the game.

In the raw dataset we analyze, there are 21262 rows, where each row represents a single team's match data during a given match. The columns of interest are:
- `gameid` : (str) unique id for each match
- `league` : (str) competitive league where the match took place 
- `patch` : (float) current patch during which the match took place
- `teamname` : (str) name of team
- `result` : (int) win represented by 1 or loss represented by 0
- `gamelength` : (int) match duration in seconds
- `golddiffat15` : (float) difference in gold at 15min between a team and its opponent

---

## Cleaning and EDA

### Data Cleaning

Upon first view of the dataset, it was clear that many values under `golddiffat15` were missing for particular rows. To avoid issues when aggregating on this column in the future, the simplest solution was to simply drop rows containing missing data for this column. 

Additionally, every match in the dataset is represented in two rows for each team in the match. The values of `golddiffat15` are symmetric for each game, since one team will be ahead in gold, and the opponent behind. However, in most cases for this analysis, we want to observe the absolute difference in gold at 15min. A new column was added to represent these strictly positive values. 

Since our question involves teams/matches during WCS, a final column was added to the dataframe with boolean values indicating whether or not a particular match took place during WCS. The main use of this column was for data visualization later.

Below, we see the head of the cleaned dataframe:

| gameid                | league   |   patch | teamname                      |   result |   gamelength |   golddiffat15 |   abs_golddiffat15 | is_wcs   |
|:----------------------|:---------|--------:|:------------------------------|---------:|-------------:|---------------:|-------------------:|:---------|
| ESPORTSTMNT01_2690210 | LCK CL   |   12.01 | Fredit BRION Challengers      |        0 |         1713 |            107 |                107 | False    |
| ESPORTSTMNT01_2690210 | LCK CL   |   12.01 | Nongshim RedForce Challengers |        1 |         1713 |           -107 |                107 | False    |
| ESPORTSTMNT01_2690219 | LCK CL   |   12.01 | T1 Challengers                |        0 |         2114 |          -1763 |               1763 | False    |
| ESPORTSTMNT01_2690219 | LCK CL   |   12.01 | Liiv SANDBOX Challengers      |        1 |         2114 |           1763 |               1763 | False    |
| ESPORTSTMNT01_2690227 | LCK CL   |   12.01 | KT Rolster Challengers        |        1 |         1972 |           1191 |               1191 | False    |

---

### Univariate Analysis

In order to understand the distribution of gold differences at 15min, we observe the histogram below, where the y-axis represents the number of matches with a particular gold difference at 15min. In general, we notice most games closer to 0 difference in gold at 15min, and less games with larger absolute differences.

<iframe src="assets/golddiff-plot.html" width=800 height=600 frameBorder=0></iframe>

---

### Bivariate Analysis

The following scatterplot displays the game length in seconds versus the absolute gold difference between teams at 15min, including the subset of matches from WCS. We notice a general negative trend, where the length of games seem to decrease when the absolute gold difference at 15min was larger, which follows under our overall understanding of gold leads and the impact of gold on match outcomes.

<iframe src="assets/gamelen-golddiff-plot.html" width=800 height=600 frameBorder=0></iframe>

To transition toward analysis of WCS matches specifically, we examine the gold difference at 15min for the matches played by each team in WCS. By ordering the WCS teams in order of their final placement, we notice a general negative trend between gold difference and placement. Top teams such as DRX and T1 seem to have higher, positive gold differences at 15min verses their opponents, whereas bottom teams can be seen to have more negative gold differences, indicating a relationship between early-game gold leads and the final outcome of matches.

<iframe src="assets/wcs-golddiff-plot.html" width=800 height=600 frameBorder=0></iframe>

In relation to our question, one might consider the absolute difference for particular matches between teams in WCS to give insight on the competitiveness between teams. We could guess that matches between two top teams might have gold differences at 15min closer to 0, whereas unbalanced matches between a better team and a worse team might have larger absolute gold differences.

---

### Interesting Aggregates

We group the WCS dataset by each team and observe the mean gold difference at 15min. One might notice that teams with better placement in WCS have more positive mean gold difference at 15min compared to lower placed teams, indicating a possible relationship between gold difference at 15min and win-rate.

| teamname            |   golddiffat15 |
|:--------------------|---------------:|
| 100 Thieves         |       -731.833 |
| Beyond Gaming       |       -274.8   |
| CTBC Flying Oyster  |      -1299.67  |
| Chiefs Esports Club |      -2973.4   |
| Cloud9              |      -1957.17  |
| DRX                 |       1006.85  |
| DWG KIA             |       2261.67  |
| DetonatioN FocusMe  |         -9     |
| EDward Gaming       |        672.909 |
| Evil Geniuses       |        364.125 |

---

## Assessment of Missingness

The missing data from our dataset under `golddiffat15` does not seem to be **NMAR**. It wouldn't make sense for the values themselves to give any indication on whether or not they would be missing from our dataset. Each match data covers a wide range of gold differences at 15min, and there should not be a reason for why a particular value would be missing while another would not. We could easily observe the dependency of missingness of this column to other columns, such as when or where the match took place, to potentially explain the missingness of `golddiffat15`, making these missing values MAR.

After observing the missingness of `golddiffat15` in relation to the result of the game (win or loss) and performing a permutation test, we likely conclude that the column `golddiffat15` is **MCAR**, especially since missing values of `golddiffat15` come from entire matches, not particular teams. 

However, if we observe the missingness of `golddiffat15` related to `patch`, we notice that the observed distributions of missingness are different enough such that many permutations on the `patch` of each game does not see enough Total Variation Distances as or more extreme than the observed. We can safetly reject the null and conclude that the missingness in the `golddiffat15` column does depend on `patch`, so the values are **MAR**.

<iframe src="assets/patch-missingness-plot.html" width=800 height=600 frameBorder=0></iframe>

---

## Hypothesis Testing

- **Null Hypothesis:** The mean absolute gold difference at 15min during WCS games is **the same** as the mean absolute gold difference at 15min during any competitive League game.

- **Alternative Hypothesis:** The mean absolute gold difference at 15min during WCS games **is less** than the mean absolute gold difference at 15min during any competitive League game.

- **Test Statistic:** Mean Gold Difference at 15min

Since we hypothesized, based on observation, that WCS matches have smaller gold differences at 15min and therefore are more competitive than regular League matches, we chose our alternative hypothesis accordingly, and our null hypothesis to simply be that the mean gold differences are statistically the same as the rest of the leagues.

In the figure below, we see the distribution of test statistics, and the P-value resulting from the test is approximately 0.22. At a significance level of 0.01, we can likely conclude that the mean absolute gold difference at 15min during WCS games is the same as that of any other competitive League game. 

<iframe src="assets/hyp-test-plot.html" width=800 height=600 frameBorder=0></iframe>

From our initial question, this conclusion indicates that WCS matches aren't significantly more competitive than other games, based on gold difference at 15min alone. It should be noted that we might come to a different conclusion under some other test statistic, though; this question involves many nuances and variables in each league of legends match, and gold differences may not be able to accurately summarize such data.

<br>

