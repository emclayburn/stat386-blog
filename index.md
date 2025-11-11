---
title: Ethan's github page
---

<script src="https://polyfill.io/v3/polyfill.min.js?features=es6"></script>
<script id="MathJax-script" async
  src="https://cdn.jsdelivr.net/npm/mathjax@3/es5/tex-mml-chtml.js">
</script>

# What does NBA data really tell us?

The game of basketball has changed in many ways throughout the years. From the style of play to the athletes themselves, the NBA has evolved into a fast-paced, high-scoring league that emphasizes athleticism and skill. But what does the data tell us about these changes? Is it effective to rely purely on scoring, or does defense really win championships? In this blog post, we will explore some key statistics and trends in the NBA to uncover what the data really tells us about the game.

---

# Motivating Question

What do we want to learn from this data? What I am going to explore is how does points per game impact your winning percentage? Also, if you are leading at the half, does that mean you are winning the games? How consistant of a team are you from the start of the game to the end. 

---

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
import matplotlib.pyplot as plt

file_path = 'nba_team_standings_historical.csv'
df = pd.read_csv(file_path)

CORE_FEATURES = [
    'SEASON_YEAR_FULL',
    'TeamName',
    'WINS', 
    'WinPCT',
    'PointsPG',
    'DiffPointsPG',
    'AheadAtHalf',
    'HOME',
    'ROAD'
]

df_filtered = df[CORE_FEATURES].copy()

df_filtered['WinPCT'] = pd.to_numeric(df_filtered['WinPCT'], errors='coerce')

home_split = df_filtered['HOME'].str.split('-', expand=True)
df_filtered['Home_Wins'] = pd.to_numeric(home_split[0], errors='coerce')
df_filtered['Home_Losses'] = pd.to_numeric(home_split[1], errors='coerce')

road_split = df_filtered['ROAD'].str.split('-', expand=True)
df_filtered['Road_Wins'] = pd.to_numeric(road_split[0], errors='coerce')
df_filtered['Road_Losses'] = pd.to_numeric(road_split[1], errors='coerce')

df_filtered = df_filtered.drop(columns=['HOME', 'ROAD'])

df_filtered.to_csv('nba_clean_for_eda.csv', index=False)
```