---
title: What NBA Data Really Tells Us, An Analysis of Efficiency and Home-Court Advantage
---

<script src="https://polyfill.io/v3/polyfill.min.js?features=es6"></script>
<script id="MathJax-script" async
  src="https://cdn.jsdelivr.net/npm/mathjax@3/es5/tex-mml-chtml.js">
</script>

# What does NBA data really tell us?

The game of basketball has changed in many ways throughout the years. From the style of play to the athletes themselves, the NBA has evolved into a fast-paced, high-scoring league that emphasizes athleticism and skill. But what does the data tell us about these changes? Is it effective to rely purely on scoring, or does defense really win championships? In this blog post, we will explore some key statistics and trends in the NBA to uncover what the data really tells us about the game.


# Motivating Question

What do we want to learn from this data? What I am going to explore is how does points per game impact your winning percentage? Also, if you are leading at the half, does that mean you are winning the games? How consistant of a team are you from the start of the game to the end. 


# Data Collection

I got all my data directly from the **NBA API** that is available as a python package. This is an easy and ethical way to gather the data you need for analysis. There are a couple other ways to get the data, like by webscraping, but since the API is available, this is the easiest way. The important package to have installed on your computer is `nba_api`. You will need to run something like `pip3 install nba_api` to get this package. Once you have it installed, you can type `from nba_api.stats.endpoints import LeagueStandings`. This will access the NBA's API on **stats.nba.com**. From there, you can run a python code to get all the data needed from the website.

Here is the code I ran to get the data from this analysis:

```python
import pandas as pd
from nba_api.stats.endpoints import LeagueStandings
import time

START_YEAR = 2010
END_YEAR = 2025

seasons = [f'{i}-{i+1-2000}' for i in range(START_YEAR, END_YEAR)]

all_seasons_df = []

for season in seasons:
    standings = LeagueStandings(season=season, season_type='Regular Season')
    season_df = standings.get_data_frames()[0]
    season_df['SEASON_YEAR_FULL'] = season
    all_seasons_df.append(season_df)
    time.sleep(1) 
        
df_final = pd.concat(all_seasons_df, ignore_index=True)

filename = 'nba_team_standings_historical.csv'
df_final.to_csv(filename, index=False)
```

At this point, the data you have collected will be saved as "nba_team_standings_historical.csv". Now that we have all of the data, we want to select the variables of interest and clean the data. We can run the following code to do so:

```python
import pandas as pd
import numpy as np

df_main = pd.read_csv('nba_team_standings_historical.csv')
df_3pt = pd.read_csv('nba_3pt_attempts_historical.csv')

df_3pt['FG3A_PG'] = df_3pt['FG3A'] / df_3pt['GP']

CORE_FEATURES = [
    'SEASON_YEAR_FULL',
    'TeamID',
    'TeamName', 'WINS', 'WinPCT', 'PointsPG', 'DiffPointsPG',
    'AheadAtHalf', 'HOME', 'ROAD'
]

df_filtered = df_main[CORE_FEATURES].copy()

df_filtered['WinPCT'] = pd.to_numeric(df_filtered['WinPCT'], errors='coerce')
home_split = df_filtered['HOME'].str.split('-', expand=True)
df_filtered['Home_Wins'] = pd.to_numeric(home_split[0], errors='coerce')
df_filtered['Home_Losses'] = pd.to_numeric(home_split[1], errors='coerce')
road_split = df_filtered['ROAD'].str.split('-', expand=True)
df_filtered['Road_Wins'] = pd.to_numeric(road_split[0], errors='coerce')
df_filtered['Road_Losses'] = pd.to_numeric(road_split[1], errors='coerce')
df_filtered = df_filtered.drop(columns=['HOME', 'ROAD'])

df_final = pd.merge(
    df_filtered, 
    df_3pt[['TEAM_ID', 'SEASON_YEAR_FULL', 'FG3A_PG']],
    left_on=['TeamID', 'SEASON_YEAR_FULL'], 
    right_on=['TEAM_ID', 'SEASON_YEAR_FULL'], 
    how='left'
)
df_final = df_final.drop(columns=['TEAM_ID'])

df_final.to_csv('nba_clean_for_eda.csv', index=False)
```

# Finding Number 1

The first question I want to answer with the data is does having a higher offensive presence i.e scoring more point, lead to a higher win percentage? The results I found were quite shocking. Use this code, I was able to get a graph and a correlation coefficient:

``` python
new_filename = 'nba_clean_for_eda.csv'
df_final = pd.read_csv(new_filename)

X_VAR = 'PointsPG'
Y_VAR = 'WinPCT'

correlation = df[X_VAR].corr(df[Y_VAR])

plt.figure(figsize=(10, 6))

plt.scatter(df[X_VAR], df[Y_VAR], alpha=0.6, s=50)

plt.title(f'NBA Success vs. Offensive Efficiency (2010-11 to 2023-24)\nCorrelation (r): {correlation:.4f}', fontsize=14)
plt.xlabel('Points Per Game (PointsPG)', fontsize=12)
plt.ylabel('Winning Percentage (WinPCT)', fontsize=12)
plt.grid(axis='y', linestyle='--', alpha=0.7)

z = np.polyfit(df[X_VAR], df[Y_VAR], 1)
p = np.poly1d(z)
plt.plot(df[X_VAR], p(df[X_VAR]), "r--", label=f'Trend Line')

plot_filename = 'eda_points_vs_winpct.png'
plt.savefig(plot_filename)
plt.close()
```
![NBA Success vs. Offensive Efficiency](eda_points_vs_winpct.png)

From the chart, we can see that the correlation coefficient between team success and offenseive efficiency is only .3232, which is not strong. While having a high powered offense does help you win games, it doesn't make that you win every game. There are other factors that play into winning a game. Perhaps defense really does win championships. I wanted to dive a little deeper into this stat. I decided to look at point differential vs winning percentage instead of just pure scoring. When I looked at that result, something more predictable happened:

``` python
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt

file_path = 'nba_clean_for_eda.csv'
df = pd.read_csv(file_path)

X_VAR_DIFF = 'DiffPointsPG'
Y_VAR = 'WinPCT'

correlation_diff = df[X_VAR_DIFF].corr(df[Y_VAR])

print(f"--- Correlation Analysis for {X_VAR_DIFF} vs. {Y_VAR} ---")
print(f"The Pearson correlation coefficient (r) is: {correlation_diff:.4f}")

plt.figure(figsize=(10, 6))

plt.scatter(df[X_VAR_DIFF], df[Y_VAR], alpha=0.6, s=50)

plt.title(f'NBA Success vs. Scoring Margin (2010-11 to 2023-24)\nCorrelation (r): {correlation_diff:.4f}', fontsize=14)
plt.xlabel('Point Differential Per Game (DiffPointsPG)', fontsize=12)
plt.ylabel('Winning Percentage (WinPCT)', fontsize=12)
plt.grid(axis='y', linestyle='--', alpha=0.7)

z = np.polyfit(df[X_VAR_DIFF], df[Y_VAR], 1)
p = np.poly1d(z)
plt.plot(df[X_VAR_DIFF], p(df[X_VAR_DIFF]), "r--", label=f'Trend Line')

plot_filename = 'eda_diffpoints_vs_winpct.png'
plt.savefig(plot_filename)
plt.close()

print(f"\nScatter plot saved as: {plot_filename}")
```
![NBA Success vs. Scoring Margin](eda_diffpoints_vs_winpct.png)

As you can see from the plot, there correlation coefficient between team success and scoring margin is .9647, this tells us a lot more that the previous number. Teams need to outscore their opponent by a wider margin to win more games. This takes into account defense and offense.

# Finding Number 2

The next thing that I want to look at is how the points per game for the entire has changed over the last 15 years. In that time, the 3 point shot attemts have sky rocketed. Using the following code, we can find the trend line:

``` python
import pandas as pd
import matplotlib.pyplot as plt

df_final = pd.read_csv('nba_clean_for_eda.csv')

df_combined_avg = df_final.groupby('SEASON_YEAR_FULL').agg(
    Avg_PointsPG=('PointsPG', 'mean'),
    Avg_FG3A_PG=('FG3A_PG', 'mean')
).reset_index()

df_combined_avg['Season_End_Year'] = df_combined_avg['SEASON_YEAR_FULL'].str.split('-').str[1]
x_labels = df_combined_avg['Season_End_Year'].tolist()

fig, ax1 = plt.subplots(figsize=(12, 6))

color1 = 'tab:blue'
ax1.set_xlabel('NBA Season End Year', fontsize=12)
ax1.set_ylabel('Avg. Points Per Game (PointsPG)', color=color1, fontsize=12)
ax1.plot(x_labels, df_combined_avg['Avg_PointsPG'], marker='o', linestyle='-', color=color1, label='Avg. PointsPG')
ax1.tick_params(axis='y', labelcolor=color1)
ax1.tick_params(axis='x', rotation=45)

ax2 = ax1.twinx()
color2 = 'tab:red'
ax2.set_ylabel('Avg. 3-Point Attempts Per Game (FG3A_PG)', color=color2, fontsize=12)
ax2.plot(x_labels, df_combined_avg['Avg_FG3A_PG'], marker='s', linestyle='--', color=color2, label='Avg. FG3A_PG')
ax2.tick_params(axis='y', labelcolor=color2)
ax2.grid(axis='y', linestyle=':', alpha=0.5, color=color2)

plt.title('The Evolving NBA: Points Per Game vs. 3-Point Attempts Over Time', fontsize=16)
fig.legend(loc="upper left", bbox_to_anchor=(0.1, 0.95))

plt.tight_layout()
plot_filename = 'eda_points_vs_3pt_time_series_combined.png'
plt.savefig(plot_filename)
plt.close()
```
![Average NBA Points Per Game and 3 Pointers Attempted](eda_points_vs_3pt_time_series_combined.png)


Looking at the trendline, we can see that the league average of points per game has risen by almost 15 points since the 2010-11 season. We can also see that the amount of 3 pointers attempted a game follows almost the exact same trend. The "three ball" has revolutionized the scoring in the NBA.

# Finding Number 3

The last thing I wanted to look at was whether or not home court advantage really existed. Using the following code, we can see if there is a difference between home and road win totals:

``` python
file_path = 'nba_clean_for_eda.csv'
df = pd.read_csv(file_path)

avg_home_wins = df['Home_Wins'].mean()
avg_road_wins = df['Road_Wins'].mean()

home_advantage_wins = avg_home_wins - avg_road_wins

print("--- Situational Analysis: Home-Court Advantage ---")
print(f"Average Home Wins (per season): {avg_home_wins:.2f}")
print(f"Average Road Wins (per season): {avg_road_wins:.2f}")
print(f"Average Home-Court Advantage (Difference): {home_advantage_wins:.2f} wins")

labels = ['Average Home Wins', 'Average Road Wins']
wins = [avg_home_wins, avg_road_wins]

plt.figure(figsize=(8, 6))
plt.bar(labels, wins, color=['darkblue', 'gray'])

for i, win_count in enumerate(wins):
    plt.text(i, win_count + 0.5, f'{win_count:.2f}', ha='center', fontsize=12)

plt.title('Home vs. Road Wins (NBA 2010-11 to 2023-24)', fontsize=14)
plt.ylabel('Average Wins Per Team Per Season', fontsize=12)
plt.xlabel(f'Average Home-Court Advantage: {home_advantage_wins:.2f} Wins', fontsize=12)
plt.grid(axis='y', linestyle='--', alpha=0.5)

plot_filename = 'eda_home_road_comparison.png'
plt.savefig(plot_filename)
plt.close()
```

![Average NBA Home Wins vs Road Wins](eda_home_road_comparison.png)

From the graph, we can see that there is such of a thing as home court advantage. Teams tend to win almost 6 more games a year at home compared to on the road.