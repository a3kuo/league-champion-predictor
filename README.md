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

