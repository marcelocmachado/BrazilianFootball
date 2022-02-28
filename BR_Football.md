# Brazilian Football League (2003-2021) EDA

![alt text](http://s.glbimg.com/es/ge/f/original/2012/11/18/festa_fred_fluminense3_andur.jpg)

This is an article about the history of the Brazilian Football League's first division between the year of 2003 and 2021. 

In 2003 the tournament changed its system to a _**round-robin**_ tournament (or all-play-all tournament), which is a competition in which each team meets every other participant. Before the round-robin system, the Brazilian Football League was played in a cup system, with a group stage - where only a few teams would get qualified to the next round - followed by a knock-out stage - where the teams that lost would get eliminated.

In the current system, at the end of a match, the winning team is rewarded with three points and the losing team with zero. If the match ends up as a tie, both teams are rewarded with one point.

The team that accumulates the highest sum of points will automatically be the champion. The bottom four teams are automatically relegated to the second division.

## Cleaning and Formatting Data

For this analysis, two datasets were utilized. The first provided data from the seasons of 2003-2020. For the most up-to-date analysis, the second dataset was scraped from the web, with the data of the season of 2021.



```r
# Checking names of columns
head(br)

# Dropping unused columns
br = br[-c(3:4,7:8,11:13)]

# Changing (translating) column names
colnames(br) = c('Round','Season','Home','Away','HG','AG')

# Extracting the year of the date
br$Season = as.numeric(str_sub(br$Season, -4, -1))

# Part of the 2020 Season was played in 2021, due to COVID. So it is necessary
# to fix the season year
br$Season[br$Season == 2021] = 2020

# Cleaning HOME names

br$Home = gsub('Ã£', 'a', br$Home)
br$Home = gsub('Ã©', 'e', br$Home)
br$Home = gsub('Ã¡', 'a', br$Home)
br$Home = gsub('Ãº', 'u', br$Home)
br$Home = gsub('Ãª', 'e', br$Home)
br$Home = gsub('Ã³', 'o', br$Home)
br$Home = gsub('Ã', 'i', br$Home)

br$Home = str_to_title(tolower(br$Home))


# Cleaning HOME names

br$Away = gsub('Ã£', 'a', br$Away)
br$Away = gsub('Ã©', 'e', br$Away)
br$Away = gsub('Ã¡', 'a', br$Away)
br$Away = gsub('Ãº', 'u', br$Away)
br$Away = gsub('Ãª', 'e', br$Away)
br$Away = gsub('Ã³', 'o', br$Away)
br$Away = gsub('Ã', 'i', br$Away)

br$Away = str_to_title(tolower(br$Away))

# Function to determine if Home Team or Away Team won the match
resFunc = function (HG, AG){
    if (HG > AG)
        br$Result = 'H'
    else if (HG < AG)
        br$Result = 'A'
    else
        br$Result = 'D'
}

# Assingning results to new column
br$Result = mapply(resFunc, br$HG, br$AG)

# Exporting clean table
# write.csv(br, 'BRA-03-20_clean.csv')

# Since this dataset only covers the seasons of 2003 until 2020, we will add
# the data for the season of 2021 now, which is already formatted
br = rbind(br,br21)

```
## APPEARENCES

Not all clubs have same amount of games played in the First Division tournament. Some got relegated and some were promoted to the First Division after 2003.

The following table shows the teams with the most appearences in the First Division.

|    |    **Team**   | **Appearences** |
|:--:|:-------------:|:---------------:|
|  1 |    Flamengo   |       742       |
|  2 |   Fluminense  |       742       |
|  3 |     Santos    |       742       |
|  4 |   Sao Paulo   |       742       |
|  5 |  Athletico-PR |       704       |
|  6 |  Atletico-MG  |       704       |
|  7 |  Corinthians  |       704       |
|  8 | Internacional |       704       |
|  9 |     Gremio    |       700       |
| 10 |    Cruzeiro   |       666       |

> We can notice that only 4 teams - **Flamengo**, **Fluminense**, **Santos** and **Sao Paulo** have participated in all First Division Seasons (2003-2021).

### Insert Graph Here

## GOALS

# Goals intro

| **Season** | **homeGoals** | **awayGoals** | **totalGoals** | **homeGoals %**     | **awayGoals %**    |
|:--------:|:-----------:|:-----------:|:------------:|:-------------:|:-------------:|
| 2003   | 982       | 610       | 1592       | 61.68341709 | 38.31658291 |
| 2004   | 947       | 587       | 1534       | 61.73402868 | 38.26597132 |
| 2005   | 835       | 616       | 1451       | 57.54651964 | 42.45348036 |
| 2006   | 604       | 426       | 1030       | 58.6407767  | 41.3592233  |
| 2007   | 634       | 413       | 1047       | 60.55396371 | 39.44603629 |
| 2008   | 658       | 377       | 1035       | 63.57487923 | 36.42512077 |
| 2009   | 660       | 434       | 1094       | 60.32906764 | 39.67093236 |
| 2010   | 581       | 397       | 978        | 59.40695297 | 40.59304703 |
| 2011   | 610       | 407       | 1017       | 59.98033432 | 40.01966568 |
| 2012   | 559       | 380       | 939        | 59.5314164  | 40.4685836  |
| 2013   | 558       | 378       | 936        | 59.61538462 | 40.38461538 |
| 2014   | 540       | 320       | 860        | 62.79069767 | 37.20930233 |
| 2015   | 555       | 342       | 897        | 61.8729097  | 38.1270903  |
| 2016   | 564       | 348       | 912        | 61.84210526 | 38.15789474 |
| 2017   | 526       | 397       | 923        | 56.98808234 | 43.01191766 |
| 2018   | 525       | 302       | 827        | 63.48246675 | 36.51753325 |
| 2019   | 525       | 351       | 876        | 59.93150685 | 40.06849315 |
| 2020   | 536       | 408       | 944        | 56.77966102 | 43.22033898 |
| 2021   | 483       | 359       | 842        | 57.36342043 | 42.63657957 |

### Average of Home and Away goals across all seasons

| **homeGoals AVG (%)** | **awayGoals AVG (%)** |
|:---------------------:|:---------------------:|
|        60.19198       |        39.80802       |

60% of goals scored across all seasons were scored by teams playing at home - high influence

We can see that the year with the lowest home influence in goal scoring was 2020, which makes total sense since most of the games had no public due to COVID.

| **Season** | **homeGoals (%)** | **awayGoals (%)** |
|:----------:|:-----------------:|:-----------------:|
|    2008    |      63.57488     |      36.42512     |
|    2020    |      56.77966     |      43.22034     |

### plot with hg ag tg per season

### Highest scoring teams across all seasons

|    | **Team**          | **HG**  | **AG**  | **Total** |
|:----:|:---------------:|:-----:|:-----::|-------:|
| 1  | Santos        | 674 | 421 | 1095  |
| 2  | Sao Paulo     | 624 | 444 | 1068  |
| 3  | Flamengo      | 616 | 437 | 1053  |
| 4  | Atletico-MG   | 624 | 410 | 1034  |
| 5  | Fluminense    | 555 | 429 | 984   |
| 6  | Cruzeiro      | 590 | 388 | 978   |
| 7  | Gremio        | 606 | 350 | 956   |
| 8  | Palmeiras     | 567 | 380 | 947   |
| 9  | Internacional | 565 | 372 | 937   |
| 10 | Athletico-PR  | 580 | 353 | 933   |

Obviously, teams haven't played the same amount of games. To evaluate the team's attacking power, we have to consider their average of goals scored per game played.

### Highest avg

|    | **Team**        | **HG**  | **AG**  | **Total** | **Appearences** | **GoalsPerGame** |
|:----:|:-------------:|:-----:|:-----:|:-------:|:-------------:|:--------------:|
| 1  | Barueri     | 35  | 24  | 59    | 38          | 1.552631579  |
| 2  | Santos      | 674 | 421 | 1095  | 742         | 1.47574124   |
| 3  | Atletico-MG | 624 | 410 | 1034  | 704         | 1.46875      |
| 4  | Cruzeiro    | 590 | 388 | 978   | 666         | 1.468468468  |
| 5  | Paysandu    | 132 | 61  | 193   | 134         | 1.440298507  |
| 6  | Sao Paulo   | 624 | 444 | 1068  | 742         | 1.4393531    |
| 7  | Palmeiras   | 567 | 380 | 947   | 658         | 1.439209726  |
| 8  | Flamengo    | 616 | 437 | 1053  | 742         | 1.419137466  |
| 9  | Goias       | 439 | 274 | 713   | 514         | 1.387159533  |
| 10 | Bragantino  | 62  | 43  | 105   | 76          | 1.381578947  |

Curiously, Barueri has the highest average of goals per game. This team took part into the Brazilian League for only one season (2009), ending at the 11st position. After 2009, due to internal issues the team had to switch names and location.

### Most ceded goals

|    | **Team**        | **cededGoalsHome**  | **cededGoalsAway**  | **Total** |
|:----:|:-------------:|:-----:|:-----:|:-------:|
| 1  | Fluminense   | 380 | 564 | 944 |
| 2  | Atletico-Mg  | 374 | 535 | 909 |
| 3  | Flamengo     | 350 | 531 | 881 |
| 4  | Santos       | 342 | 535 | 877 |
| 5  | Athletico-Pr | 321 | 553 | 874 |
| 6  | Vasco        | 344 | 488 | 832 |
| 7  | Cruzeiro     | 356 | 462 | 818 |
| 8  | Botafogo-Rj  | 344 | 457 | 801 |
| 9  | Sao Paulo    | 318 | 481 | 799 |
| 10 | Palmeiras    | 344 | 436 | 780 |

### Highest ceded goal avg

### Insert bar blot here

