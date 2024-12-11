# Predicting the Position for a Champion in League of Legends

This is an extensive data science project that covers the data science process, from data analysis with hypothesis testing, using permutation tests, and making predictive models with the use of League of Legends matches.

Author: Alexander Kuo

## Introduction
This dataset covers professional matches from the multiplayer game League of Legends (League/LOL). The game has 2 teams, each with 5 players on each team, aiming to win the game by destroyed the enemy Nexus. Each player play as a unique champion with different abilities while playing together as a team to strategically power through and conquer the enemy team. It is one of the most popular games in the world, leading to its growth in esports, resulting in the matches being documented. The dataset, created by Oracle's Elixir, uses games from the professional scene in 2022. 

Data being collected include game elements that define the performance of players and their teams, as well as general information about the matches being played. The game has over 150 unique champions, which are characters the player can choose to play as. There are also five possible roles, one for each team member, for each character, and the main question I have is: **Given a character, what position did they play?** While playing the game I have played many characters, but I enjoy a select few. However, I want to know which position they played in order to have the most success and fun playing. As a result, the model I am creating will predict what position(s) my chosen character would play.

The original dataframe has 150,180 rows, representing individual players and team summaries, and many columns describing what events have occured in each game. However, only a select few of these columns are relevant to the analysis. These columns include:

|Column                |Description|
|---                |---        |
|`'league'`                |Professional league/region where the match happened|
|`'side'`                |The side of the map the player played on ('Red' or 'Blue')|
|`'position'`                |Position the player was assigned and played as ('top', 'jng', 'mid', 'bot', 'sup')|
|`'champion'`                |Character the player chose for the match|
|`'gamelength'`                |Length in seconds of the game|
|`'result'`                |The outcome of the game (0 if the player lost, 1 if the player won)|
|`'kills'`                |Number of enemy champions killed by the player or team|
|`'deaths'`                |Number of times a player or team died during the game|
|`'assists'`                |Number of times a player helped achieve a kill while not getting the kill|
|`'damagetochampions'`                |Total damage to enemy champions dealt during the match|
|`'totalgold'`                |Total gold the player obtained during the match|
|`'total cs'`                |Total amount of minion or monster kills the player got during the match|

These columns will be used to explore the data. Since my favorite champion is Karma, the data and models will focus on Karma. So the final question is **"What is the best role for Karma to play?"**

## Data Cleaning and Exploratory Data Analysis
The data needs to be cleaned in order to analyze it efficiently.

### Cleaning
Many of the columns were given as 0s and 1s, which required casting them to type `bool` for better interpretation. Some of these columns include `playoffs`, `result`, or columns that represented the first of something like `firstdragon`.

However, several of these columns were irrelevant to the analysis for this question, so only the following columns were kept:
`league`, `side`, `position`, `champion`, `gamelength`, `result`, `kills`, `deaths`, `assists`, `damagetochampions`, `totalgold`, and `total cs`. Furthermore, I am only interested in individual players performances, so I dropped rows that describe the team summaries.

The head of the cleaned dataframe is:

| league   | side   | position   | champion   |   gamelength | result   |   kills |   deaths |   assists |   damagetochampions |   totalgold |   total cs |
|:---------|:-------|:-----------|:-----------|-------------:|:---------|--------:|---------:|----------:|--------------------:|------------:|-----------:|
| LCKC     | Blue   | top        | Renekton   |         1713 | False    |       2 |        3 |         2 |               15768 |       10934 |        231 |
| LCKC     | Blue   | jng        | Xin Zhao   |         1713 | False    |       2 |        5 |         6 |               11765 |        9138 |        148 |
| LCKC     | Blue   | mid        | LeBlanc    |         1713 | False    |       2 |        2 |         3 |               14258 |        9715 |        193 |
| LCKC     | Blue   | bot        | Samira     |         1713 | False    |       2 |        4 |         2 |               11106 |       10605 |        226 |
| LCKC     | Blue   | sup        | Leona      |         1713 | False    |       1 |        5 |         6 |                3663 |        6678 |         42 |

### Univariate Analysis
To find columns to use for prediction, I performed univariate analysis on the distribution of two single columns. The first one is totalgold, where I wanted to see if there were any unusual trends.

<iframe
  src="assets/totalgold.html"
  width="900"
  height="600"
  frameborder="0"
></iframe>

The other distribution is for the total damage to champions.

<iframe
  src="assets/totaldmg.html"
  width="900"
  height="600"
  frameborder="0"
></iframe>

These graphs show roughly normal distributions with slight right-skew in both. Some high outliers are present but that is expected given long game lengths typically lead to larger numbers.

### Bivariate Analysis
Next I wanted to see how `position` and `result` would compare with each other for the champion Karma. The bar graph below shows the number of games played and how many were won depending on the position.

<iframe
  src="assets/karmapos.html"
  width="900"
  height="600"
  frameborder="0"
></iframe>

Bars for mid and sup are the most prominent in the graph, showing that those 2 positions are played the most, even though they have quite different jobs throughout the game.

### Interesting Aggregates
Different positions do different jobs, but they are still all measured by the same columns. Therefore I wanted to see what what happen when I grouped by position and display the sum of some of the cleaned columns. The resulting dataframe is:

| position   |   totalgold |   damagetochampions |   kills |   assists |         total cs |
|:-----------|------------:|--------------------:|--------:|----------:|-----------------:|
| bot        |   341633723 |         4.52077e+08 |  106658 |    134494 |      6.97024e+06 |
| jng        |   269993816 |         2.55852e+08 |   76486 |    171563 |      4.3294e+06  |
| mid        |   315153199 |         4.37203e+08 |   87658 |    147556 |      6.52568e+06 |
| sup        |   190159827 |         1.35259e+08 |   21891 |    229965 | 885734           |
| top        |   307687833 |         3.88651e+08 |   69963 |    125713 |      6.19105e+06 |

It shows that bot lane often has the most combined stats, but assists are led by the support role.

## Assessment of Missingness
Some columns include missing values, but most of them are just missing by design. However, there are some that are missing in other ways.

### NMAR Analysis
All the ban columns (`ban1`, `ban2`, `ban3`, `ban4`, `ban5`) I believe are NMAR (Not Missing At Random) because besides the bans being in groups due to design, some values are missing seemingly for no reason. This is the case because bans are optional actions decided by the player during the pregame phase (champ select) portion of match, although in a professional setting you would always want to ban. The values are missing because either the player chose to not ban either on purpose or by accident, such as the timer for banning running out, making these 5 columns NMAR. In order to make these columns MAR (Missing At Random), there needs to be a column that tracks total bans, either per team or per game. There are normally 5 bans per team, so 10 per game, and if the number deviates from this, it shows one of the bans are missing, making it MAR on this new column.

### Missingness Dependency
One column that has some missing values is the `monsterkillsenemyjungle` column, which represents the amount of monsters the player killed in the enemy's jungle (where monsters spawn).
I want to see if this column is MAR depending on `league`, so my hypotheses are: <br>

**Null Hypothesis**: The distribution of `league` when `monsterkillsenemyjungle` is missing has the same distribution as when `monsterkillsenemyjungle` is not missing. <br>
**Alternate Hypothesis**: The distribution of `league` when `monsterkillsenemyjungle` is missing is different when `monsterkillsenemyjungle` is not missing. <br>
**Test Statistic**: `TVD`s because I am comparing categorical distributions. <br>
**Significance Level**: 0.05

I performed a permutation test by shuffling the `league` column and completed 500 repetitions. The result of the empirical distribution of the TVDs is below:

<iframe
  src="assets/marleague.html"
  width="900"
  height="600"
  frameborder="0"
></iframe>

The p-value is 0.0, which is less than 0.05, so I rejected the null hypothesis, showing that `monsterkillsenemyjungle` is MAR depending on `league`.

After, I wanted to see if another column, `side` had dependency for `monsterkillsenemyjungle`, so my new hypotheses are: <br>

**Null Hypothesis**: The distribution of `side` when `monsterkillsenemyjungle` is missing has the same distribution as when `monsterkillsenemyjungle` is not missing. <br>
**Alternate Hypothesis**: The distribution of `side` when`monsterkillsenemyjungle` is missing is different when `monsterkillsenemyjungle` is not missing. <br>
**Test Statistic**: `TVD`s because I am comparing categorical distributions. <br>
**Significance Level**: 0.05

Once again, I performed a permutation test by shuffling the `side` column and repeated the same process as above. The result of the empirical distribution of TVDs is below:

<iframe
  src="assets/notmarside.html"
  width="900"
  height="600"
  frameborder="0"
></iframe>

The p-value is 1.0, which is greater than 0.05, so I failed to reject the null hypothesis, meaning that `monsterkillsenemyjungle` is not MAR depending on `side`.

## Hypothesis Testing
Back to the original question, I want to see what position Karma is best in. However, I want to keep the comparison similar.

*Note: position and lane mean the same thing for "mid", "top", and "bot"
ex. when position is mid, it means "mid lane"*

Since mid and top lanes are both solo lanes, where players play 1v1 against their opponent, I made my hypotheses based on these lanes:

**Null Hypothesis**: Karma played in the mid lane has the same gold distribution as Karma played in the top lane. <br>
**Alternate Hypothesis**: Karma played in the mid lane has a higher gold distribution than Karma played in the top lane. <br>
**Test Statistic**: Difference in means <br>
**Significance Level**: 0.05

I performed a permutation test by shuffling the `mid` and `top` labels while plotting `totalgold`. I repeated this process 500 times and the following histogram shows the results:

<iframe src="assets/meandifftotalgold.html" width="900" height="600" frameborder="0"></iframe>

The p-value is 0.0, which is less than 0.05 so I reject the null hypothesis. This implies that there is a difference in total gold distribution between the mid and top lanes for Karma.

## Framing a Prediction Problem

### Findings
From previous hypothesis testing, it suggests that there is a difference in Karma played mid and Karma played top, but there are many other columns than just total gold. I also want to generalize this to any champion, but the model will only focus on the champion Karma. So my prediction problem will be:
**Given the champion Karma, can I predict what position they played?

### Execution
This model will be a multi-class classification where the response variable is `position`. This variable is chosen because it represents the position the champion played in the game. F1-score will be the chosen metric because although accuracy is a good overall metric, each position is not played equally by most champions, including Karma as shown earlier, resulting in the data being imbalanced. This means that F1-score will give better results. I am trying to predict what position they played after the game completed, so all columns are available to be used for prediction.

## Baseline Model

Given the champion Karma, I used `RandomForestClassifier` to predict the position that the champion was played in. The dataset is quite large and I want to avoid overfitting if I just used a `DecisionTreeClassifier`. The features I am using are `damagetochampions` (quantitative), `totalgold` (quantitative), and `result` (nominal and given in binary form).` Since `gamelength` varies with each match, I need to transform `damagetochampions` and `totalgold` using `StandardScaler`. No encondings are necessary since result is already in the correct form.

`damagetochampions` was chosen because by doing more damage to the enemy, a player is able to apply pressure to them and `totalgold` was chosen because higher gold translates to a higher power level of the champion. `result` is chosen because a win would be better than a loss in that role.

The model resulted in an overall F1-score of 0.2850. Both *precision* and **recall** are low, so that means there is a large amount of false positives and false negatives. This could imply that the predictions are too wide and need to be narrowed, which will be done by improving the model.

## Final Model

The model is kept as a `RandomForestClassifier`, but features added were `cspm` (which translates to minion and monster kills per minute) because the higher the number, the more gold the player is able to generate, so they can get stronger faster than the enemy. I will also be adding `visionscore` because the higher the score, the more the player is able to take more control of the map and the game. Since `cspm` is already scaled to per minute, the feature will be left as is while `visionscore` will be transformed using `StandardScaler` because it usually increases as the game goes on. These values are more specific aspects of the game, so they should help determine which position could do these better than the others.

Another way to optimize the model is by tuning the hyperparameters, which were done with `GridSearchCV` to find the optimal hyperparameters, `max_depth` and `n_estimators`. The resulting combination from the grid search was that the max depth is 10, and the number of estimators is 30.

As a result of these adjustments to the model, the F1-score of the final model increased to 0.9927 from 0.2850. This is a large increase from the baseline model, showing that the final model will perform much better. The nearly 1 F1-score shows that both precision and recall are much better than they original were, so the predictions made by the model improved as well.

## Fairness Analysis

I want to see if the model is fair for each `side`, so my different groups will be Blue side and Red side. Each game of League of Legends has 2 teams, which are on different sides (blue and red). More specifically, my question for fairness is: **Does my model perform worse for players on blue side than it does for players on Red side?**
My hypotheses are the following:

**Null Hypothesis**: My model is fair. The accuracy for players on blue side or red side are roughly the same, and any differences are due to random chance. <br>
**Alternate Hypothesis**: My model is unfair. The accuracy for players on blue side is not the same as the accuracy for players on red side. <br>
**Test statistic**: The absolute difference in accuracy between players on blue or red side <br>
**Significance level**: 0.01 

The following histogram shows the distribution of absolute differences in accuracy between sides as a result of a permutation test where the `side` column was shuffled.

<iframe
  src="assets/fairness.html"
  width="900"
  height="600"
  frameborder="0"
></iframe>

The resulting p-value is 0.394, which is greater than 0.01, so I failed to reject the null hypothesis. This suggests that the model is fair for both blue and red sides.
