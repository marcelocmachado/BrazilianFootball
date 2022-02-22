# Brazilian Football League (2003-2021) EDA

![alt text](http://s.glbimg.com/es/ge/f/original/2012/11/18/festa_fred_fluminense3_andur.jpg)

This is an article about the history of the Brazilian Football League's first division between the year of 2003 and 2021. 

In 2003 the tournament changed its system to a _**round-robin**_ tournament (or all-play-all tournament), which is a competition in which each team meets every other participant. Before the round-robin system, the Brazilian Football League was played in a cup system, with a group stage - where only a few teams would get qualified to the next round - followed by a knock-out stage - where the teams that lost would get eliminated.

In the current system, at the end of a match, the winning team is rewarded with three points and the losing team with zero. If the match ends up as a tie, both teams are rewarded with one point.

The team that accumulates the highest sum of points will automatically be the champion. The bottom four teams are automatically relegated to the second division.

## Cleaning and Formatting Data

For this analysis, two datasets were utilized. The first provided data from the seasons of 2003-2020. For the most up-to-date analysis, the second dataset was scraped from the web, with the data of the season of 2021.



```
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
|Machine|Whole|Cluster|Crunch|Roasted|
|:------:|:------:|:------:|:------:|:------:|
|Hulling|1.00|1.00|1.00|1.00|
|Roasting|2.00|1.50|1.00|4.00|
|Coating|1.00|0.70|0.20|0|
|Packaging|2.50|1.60|1.25|1.00|
