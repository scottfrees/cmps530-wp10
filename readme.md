# Historical Winning Streaks - is momentum a thing?
For this project, we'll return to MLB data to examine whether winning streaks in baseball are more or less common than we'd expect based on just the pure mathematics of chance.  You will calculate the frequency of long winning streaks from the raw data, and then compare those results to what would be expected using two slightly different mathematical models.

In each part, I'll provide you some of the answers that you should expect your code to generate, so you know if you are doing things correctly.  There is also a good amount of material online to [check your work](https://en.wikipedia.org/wiki/List_of_Major_League_Baseball_longest_winning_streaks).

This analysis is in-part derived from a much more detailed and complex analysis of winning streaks in baseball, which you can see [here](https://tht.fangraphs.com/the-probability-of-streaks/) if you are interested (note your numbers in this analysis will vary slightly from what they present, for a variety of reasons).  In fact, the mathematics behind [success streaks](https://www.askamathematician.com/2010/07/q-whats-the-chance-of-getting-a-run-of-k-successes-in-n-bernoulli-trials-why-use-approximations-when-the-exact-answer-is-known/) is a subject of much complexity!


## Part 1: Actual Winning Streaks
We will restrict our analysis of winning streaks to the years 1962 through the 2018 season.  This is largely due to the different number of games played in seasons before 1962, along with some fundamental differences in the way the distribution of talent across the leage was prior to the modern era.

As your first task, develop code to construct a listing of how many times winning streaks are observed in the data.  Create a chart (bar graph) to show the counts.
  
- Winning streaks that started in one season, and extended/continued during the following year's season should **not** be considered.
- Do not permit tie games to extend a win streak - a win streak of 10 games means the team won 10 games in a row - no ties or losses.
- Exclude winning streaks of less than 7 games from your histogram - they are too common to be useful in our analysis.
- If there is a win streak of 10 games, it should be included in the 7, 8, 9, and 10 win streak bins.  **This is really important**.

In addition, for each team-season, you should calculate the team's overall winning percentage for the season.  Make sure you store these winning percentages, we'll use them in our later analysis.  Finally, for each winning streak length, also store the winning percentages of the teams that had streaks of that length - this will give us some more insight later as well.

### Key Data
If your computation is correct, you should see the following:
- There were `1518` team-seasons from 1962-2018.  You should retain a list of the winning percentages of all these team-seasons.
- Winning streaks of the following length are observed in the data from 1962-2018:  `[7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 20, 22]`
- The number of 7 game winning streaks observed is 1133, and there were 2 winning streaks of 20 or more games.
- The top 10 winning streaks are as follows, along with the team's winning percentage:

```
CLE 2017 0.630 22
OAK 2002 0.636 20
KCA 1977 0.630 16
MIN 1991 0.586 15
ATL 2000 0.586 15
SEA 2001 0.716 15
SFN 1965 0.583 14
BAL 1973 0.599 14
OAK 1988 0.642 14
TEX 1991 0.525 14
```

## Part 2:  Winning Streak Probability
In a simplified winning streak model, we might assume that each team has a 50/50 chance of winning a given game - basically a coin-flip.  If an outcome has a probability **p** of occuring, the chances of that outcome happening **k** times in a row is p<sup>k</sup>.  Thus, the chances of flipping a coin 7 times and having it come up heads all 7 times is **0.5<sup>7</sup>**, which is 0.0078125, or 0.78%.

Baseball teams aren't equal though, some teams are a lot better than others.  For a given team, their chances of winning a particular game depends on (at least) their own ability, and the ability of the other team.  Figuring out the chances that a team will win a game can become incredibly complex, incredibly quickly.  If you are a fan of baseball, you likely understand this intuitively - even a bad team might have one exceptional starting pitcher, which makes their odds of winning on a *particular* day that player is pitching far higher than their overall team talent would predict.

We aren't going to try to model everything here - if your interested, take a look at [this](https://tht.fangraphs.com/the-probability-of-streaks/).  For our analysis, we will use a team's overall season winning percentage to model the probability of winning any given game - but we'll factor winning percentage into our model using two methods - direct and statistical.

### General Model - Winning Streak Prediction
Let's assume a team has a 60% winning percentage.  How many winning streaks of 7 games would we expect to see?  We know that the chance of a particular 7 game stretch all resulting in wins is 0.6<sup>7</sup>, which is about 2.9%.  Over a 162 game season (we'll assume all teams played 162 games, even though there are some small exceptions), the most common thought would be to multiply 162 by 2.9% and come up with about 4.5 - however this isn't quite right.

We must use the 2.9% and multiply by the number of opportunities a team has to **begin** a 7 game winning streak.  The first game of the year is an obvious opportunity.  The second game of the year **might** be an opportunity to start a 7 game winning streak too - but **only if the team lost the first game!**.  In essense, a team has an opportunity to start a 7 game win streak on the following occasions:
1. First game of season
2. After every loss, except... among the last 6 games of the season.

Opportunity 1 is a given.  Opportunity 2 depends on how many losses a team has - which can be calculated by 162 multiplied by (1-winning percentage).  Note the exception of the last 6 games though, you can't start a 7 game winning streak on the last day of the season!

In conclusion, we can predict the number of 7 game winning streaks a team with a 60% win percentage might have during a 162 game season as follows:

`0.600^7 * ((162-7)*(1-0.600) + 1) = 1.76`

That calculation is for a **single season**, for a **single team**.  Given that there are 1518 team seasons in our data set - **if all of them had a 60% win percentage** you'd expect 2671 winning streaks of 7 games.  Of course, not all teams have a 60% winning percentage!

To complete the model of expected winning streak counts, we need to use a proper winning percentage.

### Model 1:  Normal distribution of winning percentages
Baseball team winning percentages, like so many data sets, follow a *normal probability curve*.  The average winning percentage is indeed 0.500, with many teams right around that mid-point.  There are always some outliers though - the 2001 Seattle Mariners won amost 72% of their games, while the the 1962 New York Mets lost 74% of the time they took the field!

To count the number of win streaks expected, we need to assign a winning percentage to each of the 1518 team seasons, calculate the number of streaks we'd expect for that team, and then sum all of those counts up.  In our first model, you'll use a 1518 random sampling from a normal probability curve whose mean is 0.500 and has a standard deviation of 0.06, which is the observed standard deviation in winning percentages in MLB data.

In Python, you can use `numpy` to create such a list of winning percentages:
```
sample = np.random.normal(0.500, .060, number_of_seasons)
```
Your sample's mean will be roughly (but not exactly) 0.500 - you can check with `np.mean(sample)`.

For each win streak length, calculate the summation of the number of expected win streaks for each sample winning percentage.  You're results will vary - since you are using a random sample from the probability curve, but you should notice that the expected streak totals are actually quite similar to what is observed - likely just a bit less than observed actually.

If you are a baseball fan, you might be able to for some conjecture as to why this is... maybe teams really do gain momentum and confidence, which makes them slightly more likely to extend winning streaks?

### Model 2:  Direct data
Using a normal distribution is simple, and easy to compute.  However, **we actually know the winning percentage of each team**!  We can use that instead, and base our prediction on the real numbers.  For this model, calculate the expected number of win streaks of each length by computing the sum of the expected number of streaks based on each of the 1518 actual winning percentages your calculated in **Part 1**.    You should come out with 1183 expected 7 game streaks, and 113 expected 15 game winning streaks (you should compute them all of course).

Notice something interesting here... when you use the real data, the *expected* number of streaks is actually just a bit more than objected for all but the longest streaks.  If you agreed with the "momentum" theory proposed after Model 1 (normal probability curve), you might feel a bit burned now... because our new model suggests otherwise!

### Which model is right?  
Is a normal probality curve is useless?  Not at all!  

First, often we don't have the real data, so it's an invaluable substitute.  Second, and **most importantly**, there is ALWAYS a difference between observed data and statistically predicted data.  If you see observations that match an exact statistical calculation, you can be pretty sure the data is suspicious.  The key question you must always ask is whether or not the difference between observed and expected is **significant**. 

If the difference is significant, you can start theorizing why that might be - what is missing from your model?  If the difference isn't significant, then move on - generally you shouldn't waste your time wondering about insignificant differences!

So which model is better?  Probably *neither*.  They are both suggesting the same conclusion - there isn't much behind winning streaks other than chance,  and the underlying talent of the team. The statistical calculations we've made seem to model the observed data pretty well.  There are of course other factors - some really great teams might have few win streaks because of one bad starting pitcher, for example.  These variations appear minor however, and over a 50+ year period and 1500+ team seasons, those variations do not appear to skew the overall numbers significantly.  

Statistically determining significance is outside the scope of this class, but is a critical component of your statistics coursework.   For now, make sure you plot the graph of the observed data, and predicted (both models) and note the similarity of the data.

# Part 3:  Which types of teams have long winning streaks?
This part is left as an open ended exercise, but I'll leave you with some advice.  Bad baseball teams frequently win a few games in a row.  Sometimes mediocre teams get lucky and win 7 or 8 games in a row.  Teams that win 15 games in a row generally are quite good though.  One way to see if the numbers bear this out is to take a look at the  *standard deviation* in winning percentages among teams with winning streaks of various lengths.  Since winning 7 games in a row is a lot more common and easy to pull off than going on a 15 game win streak, you'd imagine (1) teams with long win streaks have higher winning percentages, and (2) there will be a less variation in the winning percentages among those highly talented teams.  **Does the data bear this out?**

