---
title: What NBA Data Really Tells Us, An Analysis of Efficiency and Home-Court Advantage
---

<script src="https://polyfill.io/v3/polyfill.min.js?features=es6"></script>
<script id="MathJax-script" async
  src="https://cdn.jsdelivr.net/npm/mathjax@3/es5/tex-mml-chtml.js">
</script>

# What does NBA data really tell us?

The game of basketball has changed in many ways throughout the years. From the style of play to the athletes themselves, the NBA has evolved into a fast-paced, high-scoring league that emphasizes athleticism and skill. But what does the data tell us about these changes? Is it effective to rely purely on scoring, or does defense really win championships? How has the trend of shooting three pointers changed? Has Stephen Curry really ruined basketball? In this blog post, we will explore some key statistics and trends in the NBA to uncover what the data really tells us about the game.


# Motivating Question

What do we want to learn from this data? From this analysis, I am looking to answer three main questions, how does points per game impact your winning percentage, how has the points and 3's attempted a game changed, and if you are leading at the half does that mean you are winning the games. This will break my blog into a few different sections, one for collection of the data and one for each question. 


# Data Collection

To gather my dataset, I used the official **NBA Stats API**, which provides public access to [stats.nba.com](https://stats.nba.com). This approach is both ethical and efficient compared to scraping raw web pages.

The Python package nba_api makes this easy to use. After installing it with pip install nba_api using terminal, I imported the endpoint LeagueStandings to pull team statistics from each regular season between 2010 and 2025. To respect API rate limits, I included a one second pause (time.sleep(1)) between requests.

```python
from nba_api.stats.endpoints import League Standings
```
The code stores each season's standings in a list and combines them into a single dtatframe with over 4,000 rows and 10+ variables, then exports it to a CSV file (nba_team_standings_historical.csv).

after downloading, I merged this data with a second file of 3-point attempt totals to add shooting informatoin, calculated per-game stats, and cleaned up home/road win columns. The final dataset, nba_clean_for_eda.csv, is ready for exploratory analysis.


# Finding 1 - Does Scoring More Really Lead to Winning More?

My first question was whether a team's offensive output, specifically points per game, predicts its winning percentage.

Using my cleaned dataset, I calculated the correlation between PointPG and WinPCT and created a scatterplot with a regression line to visualize the relationship:

```python
correlation = df['PointsPG'].corr(df['WinPCT'])
```

The result was surprising: the correlation coefficient was only 0.32, suggesting that scoring a lot doesn't necessarily guarantee success. Some teams score plenty but still lose often, likely due to poor defense or inconsistency.

![NBA Success vs. Offensive Efficiency](eda_points_vs_winpct.png)

To dig deeper, I replaced raw points with point differential per game (average points minus points allowed). This time, the correlation jumped to an almost perfect linear relationship.

![NBA Success vs. Scoring Margin](eda_diffpoints_vs_winpct.png)

In other words, it's not about simply scoring the most, it's about outscoring you opponents consistently. Teams that win big tend to defend well, not just score more.

# Finding 2 - How Have Scoring and 3-Point Attempts Changed Over Time?

Next, I wanted to see how the NBA's style of play has changed over the last 15 seasons. Specifically, how average points per game and 3-point attempts per game have evolved.

To explore this, I grouped the data by season and calculated the league averages for both stats:

``` python
df.groupby('SEASON_YEAR_FULL').agg(
  Avg_PointsPG=('PointsPG', 'mean'),
  Avg_FG3A_PG=('FG3A_PG', 'mean')
)
```
Then, I visualized both metrics on the same chart, points per game in blue, and 3-point attempts in red, to show how they've moved together over time.

![Average NBA Points Per Game and 3 Pointers Attempted](eda_points_vs_3pt_time_series_combined.png)

The results are striking: since the 2010-11 season, the average team has increased is scoring by nearly 15 points per game, and the number of 3-pointers attempted has more than doubled.

This confirms what fans already sense, the NBA has become a three-point dominated league. The rise of players like Stephen Curry and the league's emphasis on spacing and efficiency have completely reshaped how teams approach offense.

# Finding  3 - Does Home-Court Advantage Still Matter?

Finally, I wanted to test whether the long-held belief in home-court advantage still holds true in modern basketball.

Using the cleaned dataset, I separated each team's home and road records to find the league-wide averages. Then I compared the two to see if teams consistantly perform better in their own arenas:

```python
ave_home_wins = df['Home_Wins'].mean()
avg_road_wins = df['Road_Wins'].mean()
```
The visualization below makes the difference clear:

![Average NBA Home Wins vs Road Wins](eda_home_road_comparison.png)

Across all seasons from 2010 to 2025, teams win about six more games per year at home than on the road. Even in an era of private jets, advanced analytics, and neutralized travel schedules, the home crowd and familiarity of your own court, still gives teams a measurable edge.

# Conclusion

Through this analysis, we can see that NBA data tells a much richer story than the box score alone. Scoring a lot helps, but it's not the main driver in success - what really matters is scoring efficiently and defensing well enough to create a strong point differential.

Over the last 15 years, the league has experienced a clear offensive revolution. Teams are shooting and making far more three-pointers than ever before, and that shift has pushed scoring averages to historic highs. What started as a strategy used by a few sharp-shooting teams has now become the foundation of modern basketball.

Lastly, even with all the changed in pace, analytics, and travel, the home-court advantage still exists. Teams continue to win noticeably more often in their home arenas, proof that crown energy and comfort still influence performance.

Altogether, the data reveals a league that's faster, more three-point heavy, and still deeply human. While numbers and trands shape how the game evolves, the fundamentals - teamwork, defense, and energy - continue to define what it takes to win.

# My other repository

Here is a link to my other repository that has all of my data collection and EDA: [Github](https://github.com/emclayburn/Stat386-data-acquisition).