---
layout: post
title:  "Modeling the average goals per match of the English Premier League"
categories: jekyll update
---
Introduction & Data
-------------------

This analysis is an implementation of the ideas presented in the first
chapter of
[Soccermatics](https://www.goodreads.com/book/show/26073086-soccermatics)
“I Never Predict Anything and I Never Will”, a book written by David
Sumpter about the mathematics of football.

We will be modeling the average goals scored per match during the
2018/2019 English Premier League season using a Poison distribution.

The data comes from [Football UK](http://www.football-data.co.uk/) and
can be downloaded
[here](https://www.football-data.co.uk/englandm.php). The
downloaded file is named **E0.csv**.

For this analysis, we will be using the tidyverse packages: **readr and dplyr**
for data wrangling and **ggplot2** for visualization.

The analysis
------------

Let’s start first by loading the packages and setting the visual theme
of our future plots.

``` r
# Load the libraries
library(readr)
library(dplyr)
library(ggplot2)

# Set the theme for the plots
theme_set(theme_classic())
```

We can now read the downloaded data set. I usually place my data sets in
a **input** folder, located in my working directory.

``` r
# Read the data
raw_data <- read_csv("../input/E0.csv")
```

In order to have a quick look on the loaded data set, I use the function
**glimpse** to get information such as the size and the columns of the
data set.

``` r
# Have a quick look of the data set
glimpse(raw_data)
```

    ## Rows: 380
    ## Columns: 62
    ## $ Div        <chr> "E0", "E0", "E0", "E0", "E0", "E0", "E0", "E0", "E0", "E0"…
    ## $ Date       <chr> "10/08/2018", "11/08/2018", "11/08/2018", "11/08/2018", "1…
    ## $ HomeTeam   <chr> "Man United", "Bournemouth", "Fulham", "Huddersfield", "Ne…
    ## $ AwayTeam   <chr> "Leicester", "Cardiff", "Crystal Palace", "Chelsea", "Tott…
    ## $ FTHG       <dbl> 2, 2, 0, 0, 1, 2, 2, 0, 4, 0, 0, 3, 2, 2, 3, 1, 3, 1, 6, 0…
    ## $ FTAG       <dbl> 1, 0, 2, 3, 2, 0, 2, 2, 0, 0, 0, 2, 1, 0, 1, 2, 2, 3, 1, 2…
    ## $ FTR        <chr> "H", "H", "A", "A", "A", "H", "D", "A", "H", "D", "D", "H"…
    ## $ HTHG       <dbl> 1, 1, 0, 0, 1, 1, 1, 0, 2, 0, 0, 2, 2, 2, 1, 1, 3, 1, 3, 0…
    ## $ HTAG       <dbl> 0, 0, 1, 2, 2, 0, 1, 1, 0, 0, 0, 2, 0, 0, 0, 0, 1, 1, 1, 1…
    ## $ HTR        <chr> "H", "H", "A", "A", "A", "H", "D", "A", "H", "D", "D", "D"…
    ## $ Referee    <chr> "A Marriner", "K Friend", "M Dean", "C Kavanagh", "M Atkin…
    ## $ HS         <dbl> 8, 12, 15, 6, 15, 19, 11, 9, 18, 18, 12, 24, 13, 6, 25, 11…
    ## $ AS         <dbl> 13, 10, 10, 13, 15, 6, 6, 17, 5, 16, 12, 15, 15, 11, 10, 1…
    ## $ HST        <dbl> 6, 4, 6, 1, 2, 5, 4, 3, 8, 3, 1, 11, 7, 2, 11, 5, 3, 3, 14…
    ## $ AST        <dbl> 4, 1, 9, 4, 5, 0, 5, 8, 2, 6, 6, 6, 4, 3, 3, 5, 3, 6, 1, 6…
    ## $ HF         <dbl> 11, 11, 9, 9, 11, 10, 8, 11, 14, 10, 14, 12, 8, 10, 9, 14,…
    ## $ AF         <dbl> 8, 9, 11, 8, 12, 16, 7, 14, 9, 9, 16, 9, 20, 8, 5, 10, 13,…
    ## $ HC         <dbl> 2, 7, 5, 2, 3, 8, 3, 2, 5, 8, 5, 5, 2, 1, 5, 6, 3, 5, 10, …
    ## $ AC         <dbl> 5, 4, 5, 5, 5, 2, 6, 9, 4, 5, 5, 1, 5, 9, 2, 4, 5, 2, 3, 7…
    ## $ HY         <dbl> 2, 1, 1, 2, 2, 2, 0, 2, 1, 0, 2, 0, 0, 2, 0, 6, 1, 1, 0, 1…
    ## $ AY         <dbl> 1, 1, 2, 1, 2, 2, 1, 2, 2, 1, 2, 2, 5, 1, 0, 2, 1, 2, 2, 1…
    ## $ HR         <dbl> 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1, 0, 0, 0, 0, 0, 1…
    ## $ AR         <dbl> 0, 0, 0, 0, 0, 0, 1, 0, 0, 0, 1, 0, 0, 0, 0, 0, 0, 0, 0, 0…
    ## $ B365H      <dbl> 1.57, 1.90, 2.50, 6.50, 3.90, 2.37, 2.37, 4.00, 1.25, 1.85…
    ## $ B365D      <dbl> 3.9, 3.6, 3.4, 4.0, 3.5, 3.2, 3.3, 3.8, 6.5, 3.5, 3.1, 4.0…
    ## $ B365A      <dbl> 7.50, 4.50, 3.00, 1.61, 2.04, 3.40, 3.30, 1.95, 14.00, 5.0…
    ## $ BWH        <dbl> 1.53, 1.90, 2.45, 6.25, 3.80, 2.35, 2.35, 3.70, 1.20, 1.80…
    ## $ BWD        <dbl> 4.00, 3.40, 3.30, 3.90, 3.50, 3.10, 3.20, 3.75, 6.75, 3.50…
    ## $ BWA        <dbl> 7.50, 4.40, 2.95, 1.57, 2.00, 3.30, 3.20, 1.95, 14.00, 4.7…
    ## $ IWH        <dbl> 1.55, 1.90, 2.40, 6.20, 3.70, 2.20, 2.25, 3.60, 1.25, 1.80…
    ## $ IWD        <dbl> 3.80, 3.50, 3.30, 4.00, 3.35, 3.30, 3.35, 3.60, 6.10, 3.60…
    ## $ IWA        <dbl> 7.00, 4.10, 2.95, 1.55, 2.05, 3.40, 3.20, 2.00, 11.00, 4.5…
    ## $ PSH        <dbl> 1.58, 1.89, 2.50, 6.41, 3.83, 2.43, 2.36, 4.00, 1.27, 1.86…
    ## $ PSD        <dbl> 3.93, 3.63, 3.46, 4.02, 3.57, 3.22, 3.40, 3.97, 6.35, 3.51…
    ## $ PSA        <dbl> 7.50, 4.58, 3.00, 1.62, 2.08, 3.33, 3.28, 1.93, 13.25, 4.9…
    ## $ WHH        <dbl> 1.57, 1.91, 2.45, 5.80, 3.80, 2.38, 2.30, 3.80, 1.25, 1.83…
    ## $ WHD        <dbl> 3.80, 3.50, 3.30, 3.90, 3.20, 3.00, 3.20, 3.80, 5.50, 3.25…
    ## $ WHA        <dbl> 6.00, 4.00, 2.80, 1.57, 2.05, 3.30, 3.20, 1.91, 12.00, 4.8…
    ## $ VCH        <dbl> 1.57, 1.87, 2.50, 6.50, 3.90, 2.40, 2.38, 3.90, 1.25, 1.85…
    ## $ VCD        <dbl> 4.00, 3.60, 3.40, 4.00, 3.40, 3.20, 3.30, 4.00, 6.50, 3.40…
    ## $ VCA        <dbl> 7.00, 4.75, 3.00, 1.62, 2.10, 3.40, 3.30, 1.91, 13.00, 5.2…
    ## $ Bb1X2      <dbl> 39, 39, 39, 38, 39, 39, 38, 39, 38, 39, 41, 41, 40, 41, 41…
    ## $ BbMxH      <dbl> 1.60, 1.93, 2.60, 6.85, 4.01, 2.48, 2.41, 4.15, 1.29, 1.90…
    ## $ BbAvH      <dbl> 1.56, 1.88, 2.47, 6.09, 3.83, 2.36, 2.33, 3.83, 1.25, 1.84…
    ## $ BbMxD      <dbl> 4.20, 3.71, 3.49, 4.07, 3.57, 3.30, 3.40, 4.00, 6.79, 3.61…
    ## $ BbAvD      <dbl> 3.92, 3.53, 3.35, 3.90, 3.40, 3.14, 3.27, 3.80, 6.22, 3.43…
    ## $ BbMxA      <dbl> 8.05, 4.75, 3.05, 1.66, 2.12, 3.42, 3.40, 2.00, 15.00, 5.2…
    ## $ BbAvA      <dbl> 7.06, 4.37, 2.92, 1.61, 2.05, 3.31, 3.23, 1.92, 12.30, 4.8…
    ## $ BbOU       <dbl> 38, 38, 38, 37, 38, 37, 36, 36, 33, 37, 38, 39, 38, 39, 38…
    ## $ `BbMx>2.5` <dbl> 2.12, 2.05, 2.00, 2.05, 2.10, 2.46, 2.20, 1.60, 1.49, 2.45…
    ## $ `BbAv>2.5` <dbl> 2.03, 1.98, 1.95, 1.98, 2.01, 2.35, 2.09, 1.55, 1.44, 2.34…
    ## $ `BbMx<2.5` <dbl> 1.85, 1.92, 1.96, 1.90, 1.88, 1.67, 1.83, 2.55, 2.88, 1.67…
    ## $ `BbAv<2.5` <dbl> 1.79, 1.83, 1.87, 1.84, 1.81, 1.59, 1.75, 2.42, 2.72, 1.60…
    ## $ BbAH       <dbl> 17, 20, 22, 23, 20, 22, 22, 20, 21, 20, 21, 21, 20, 21, 21…
    ## $ BbAHh      <dbl> -0.75, -0.75, -0.25, 1.00, 0.25, -0.25, -0.25, 0.75, -1.75…
    ## $ BbMxAHH    <dbl> 1.75, 2.20, 2.18, 1.84, 2.20, 2.07, 2.04, 1.78, 1.95, 2.19…
    ## $ BbAvAHH    <dbl> 1.70, 2.13, 2.11, 1.80, 2.12, 2.01, 1.98, 1.74, 1.90, 2.11…
    ## $ BbMxAHA    <dbl> 2.29, 1.80, 1.81, 2.13, 1.80, 1.90, 1.92, 2.21, 2.06, 1.82…
    ## $ BbAvAHA    <dbl> 2.21, 1.75, 1.77, 2.06, 1.76, 1.86, 1.88, 2.15, 1.97, 1.76…
    ## $ PSCH       <dbl> 1.55, 1.88, 2.62, 7.24, 4.74, 2.58, 2.44, 4.43, 1.25, 2.03…
    ## $ PSCD       <dbl> 4.07, 3.61, 3.38, 3.95, 3.53, 3.08, 3.23, 4.13, 6.95, 3.19…
    ## $ PSCA       <dbl> 7.69, 4.70, 2.90, 1.58, 1.89, 3.22, 3.32, 1.81, 12.00, 4.6…

The columns of interest for this analysis will be **FTHG**, which is the
number of goals scored by the home team and **FTAG** which is the the
number of goals scored by the away team. These 2 numbers represent the
score of the match.

The number of rows of this data set is equal to the number of matches
played in the season.

``` r
# Get the number of matches played in the season
nb_matches <- nrow(raw_data)
```

To continue, we can compute the total goals scored per match using the
**mutate** function. At the same time, we can get the minimum and
maximum goals scored in the season.

``` r
# Get the total goals scored per match
goals_per_match <- raw_data %>%
  select(FTHG, FTAG) %>%
  mutate(ID = row_number(), total_goals = FTHG + FTAG)

# Compute the minimum and maximum goals per match score in the season
min_goals <- goals_per_match %>%
  pull(total_goals) %>%
  min()

max_goals <- goals_per_match %>%
  pull(total_goals) %>%
  max()

print(paste0("The minimum goals scored in a match was ", min_goals, "."))
```

    ## [1] "The minimum goals scored in a match was 0."

``` r
print(paste0("The maximum goals scored in a match was ", max_goals, "."))
```

    ## [1] "The maximum goals scored in a match was 8."

Let’s now plot the distribution of the total goals scored per match.

``` r
# Plot the distribution of total goals scored per match
ggplot(goals_per_match, aes(x = total_goals)) +
  geom_histogram(binwidth = 1,
                 fill = "cyan4",
                 color = "cyan4") +
  scale_x_continuous(breaks = seq(min_goals, max_goals, 1)) +
  stat_bin(
    binwidth = 1,
    aes(label = ..count..),
    geom = "text",
    vjust = 1
  ) +
  labs(
    x = "Total goals per match",
    y = "Number of matches",
    title = "Distribution of the total numer of goals score per match",
    subtitle = paste0("Season 2018/2019 - N = ", nb_matches)
  ) 
```

![](/assets/2021-01-14-modeling-average-goals/unnamed-chunk-5-1.png)<!-- -->

We can observe a peak in the distribution at 2 & 3, reflecting the
number of matches with a score of 1-1 or 2-1.

Let’s compute the average goals score per match.

``` r
# Compute the average goals scored per match
mean_gpm <- goals_per_match %>%
  pull(total_goals) %>%
  mean() %>%
  round(2)

print(paste0("The average goals score per match is ", mean_gpm, "."))
```

    ## [1] "The average goals score per match is 2.82."

With this parameter, we can now simulate the total goals scored per
match over the season with a Poison distribution using the **dpois**
function.

``` r
# Simulate the total goals scored per match over the season with a Poison distribution of parameter = average total goals scored per match
poisson_simu <- tibble(total_goals = min_goals:max_goals) %>%
  mutate(simulation = round(dpois(total_goals, lambda = mean_gpm) * nb_matches))
```

We can now plot the simulation.

``` r
# Plot the simulated curve
ggplot(poisson_simu, aes(x = total_goals, y = simulation)) +
  geom_point(color = "purple") +
  geom_line(color = "purple") +
  geom_text(aes(label = simulation), hjust = -0.25) +
  scale_x_continuous(breaks = seq(min_goals, max_goals, 1)) +
  labs(x = "Total goals", y = "Simulation",
       title = "Poisson simulation of the total goals scored per match") 
```

![](/assets/2021-01-14-modeling-average-goals/unnamed-chunk-8-1.png)<!-- -->

If we superpose the two plots, we can see that the simulated curve
follows the distribution.

``` r
# Plot the histogram and the simulated curve together
goals_per_match %>%
  count(total_goals) %>%
  left_join(poisson_simu, by = "total_goals") %>%
  ggplot(aes(x = total_goals)) +
  geom_col(aes(y = n), color = "cyan4", fill = "cyan4") +
  geom_line(aes(y = simulation), color = "purple") +
  scale_x_continuous(breaks = seq(min_goals, max_goals, 1)) +
  labs(x = "Total goals", y = "Number of matches",
       title = "Real vs simulated distribution of total goals scored per match")
```

![](/assets/2021-01-14-modeling-average-goals/unnamed-chunk-9-1.png)<!-- -->

We could quantify how well this simulation models the reality by perfoming a
chi-squared test.
