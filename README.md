# Predicting the Position for a Champion in League of Legends

This is an extensive data science project that covers the data science process, from data analysis with hypothesis testing, using permutation tests, and making predictive models with the use of League of Legends matches.

Author: Alexander Kuo

## Introduction
This dataset covers professional matches from the multiplayer game League of Legends (League/LOL). The game has 2 teams, each with 5 players on each team. It is one of the most popular games in the world, leading to its growth in esports, resulting in the matches being documented. The dataset, created by Oracle's Elixir, uses games from the professional scene in 2022. 

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

These columns will be used to explore the data. Since my favorite champion is Karma, the data and models will focus on Karma. So the final question is "What is the best role for Karma to play?"

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
  width="800"
  height="600"
  frameborder="0"
></iframe>

The other distribution is for the total damage to champions.

<iframe
  src="assets/totaldmg.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

These graphs show roughly normal distributions with slight right-skew in both. Some high outliers are present but that is expected given long game lengths typically lead to larger numbers.

### Bivariate Analysis
Next I wanted to see how `position` and `result` would compare with each other for the champion Karma. The bar graph below shows the number of games played and how many were won depending on the position.

<iframe
  src="assets/karmapos.html"
  width="800"
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

