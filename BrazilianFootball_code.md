```r

# Load packages
library(stringr)
library(readxl)
library(ggplot2)
library(tidyverse)
library(ggh4x)
library(reshape)


#### CLEANING AND FORMATTING DATA ####

# import datasets 
setwd('C:\\Users\\SAMSUNG\\Desktop\\PROJECTS\\BRASILEIRAO')
br = read.csv('BRA-2003-2020.csv', sep = ';')
br21 = read_xlsx('BRA-2021.xlsx')


# Checking names of columns
head(br)

# Dropping unnused columns
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


# Cleaning AWAY names
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

# Assigning results to new column
br$Result = mapply(resFunc, br$HG, br$AG)


# Since this data set only covers the seasons of 2003 until 2020, we will add
# the data for the season of 2021 now, which is already formatted
br = rbind(br,br21)


#### APPEARENCES ####

# Counting number of Seasons
numSeasons = aggregate(data=br, Season ~ Home, function(Season) length(unique(Season)))
colnames(numSeasons) = c('Team', 'numSeasons')
numSeasons = numSeasons[order(-numSeasons$numSeasons),]

# Top 15 teams with most participations in the first division
numSeasonstop15 = numSeasons[1:15,]

# Number of seasons played plot
numSeasonsPlot = ggplot(numSeasonstop15, 
                        aes(x = numSeasons, 
                            y = reorder(Team,numSeasons), fill = Team)) +
    geom_bar(stat = 'identity') +
    geom_text(aes(label = numSeasons), 
              vjust = +0.5, hjust = +1.2, size = 5, color = '#ffffff') +
    labs(title = 'Number of 1st Division Seasons \nPlayed by Team') +
    ylab('') +
    xlab('Number of Seasons') +
    theme(legend.position="none") +
    theme(panel.background = element_blank(),
          plot.title = element_text(hjust = 0.5, face = "bold"),
          axis.text.x = element_text(size=10, color = "black"),
          axis.title.x = element_text(size=11, face="bold")) +
    scale_fill_manual(values = c("#52CC71",
                                 "#52CC71",
                                 "#52CC71",
                                 "#52CC71",
                                 "#52CC71",
                                 "#52CC71",
                                 "#D83917",
                                 "#D83917",
                                 "#52CC71",
                                 "#52CC71",
                                 "#52CC71",
                                 "#52CC71",
                                 "#D83917",
                                 "#D83917",
                                 "#52CC71"))

numSeasonsPlot

# Number of games played
homeAppearences = aggregate(list(homeAp = br$Home), by = list(Team = br$Home), FUN = length)
awayAppearences = aggregate(list(awayAp = br$Away), by = list(Team = br$Away), FUN = length)

totalAppearences = merge(homeAppearences,awayAppearences)
totalAppearences$Appearences = (totalAppearences$homeAp + totalAppearences$awayAp)
totalAppearences = totalAppearences[-c(2:3)]
totalAppearences = totalAppearences[order(-totalAppearences$Appearences),]

# Teams with the most appearences in the First Division
topAppearences = totalAppearences[1:20,]


#### GOAL ANALYSIS ####

# Total sum of Home Goals and Away Goals per Season
homeAwayGoals = aggregate(x = list(homeGoals = br$HG, awayGoals = br$AG),
                          by = list(Season = br$Season), FUN = sum)


homeAwayGoals$totalGoals = homeAwayGoals$homeGoals + homeAwayGoals$awayGoals

# Percentages of Home Goals and Away Goals per Season
homeAwayGoals$HG_perc = 100*(homeAwayGoals$homeGoals/homeAwayGoals$totalGoals) 

homeAwayGoals$AG_perc = 100*(homeAwayGoals$awayGoals/homeAwayGoals$totalGoals) 

#### 

# Average of Home & Away Goals percentage across all Seasons


avg_HG_perc = mean(homeAwayGoals$HG_perc)
avg_AG_perc = mean(homeAwayGoals$AG_perc)

percMeanGoals = data.frame('HomeGoalsAVG' = avg_HG_perc,'AwayGoalsAVG' = avg_AG_perc)
percMeanGoals


# Highest and lowest influence playing home

highHome = max(homeAwayGoals$HG_perc)
lowHome = min(homeAwayGoals$HG_perc)

highLowHG = data.frame('HighHG' = highHome,'LowHG' = lowHome)
highLowHG

# HOME VS AWAY PER YEAR

# long format

HAGoalsMelt = homeAwayGoals[,1:3]
HAGoalsMelt = melt(HAGoalsMelt, id.vars = 'Season', variable.name = 'HomeVsAway')

# Line plot with difference fill
homeVsAway = ggplot(homeAwayGoals, aes(x = Season)) +
    # One line for HG
    geom_line(aes(y = homeGoals, color = "Home Goals"), size = 1) +
    # Another line for AG
    geom_line(aes(y = awayGoals, color = "Away Goals"), size = 1) +
    geom_point(aes(y = homeGoals, color = "Home Goals"), size = 2.5)+
    geom_point(aes(y = awayGoals, color = "Away Goals"), size = 2.5)+
    # stat_difference() from ggh4x package applies the conditional fill
    stat_difference(aes(ymin = awayGoals, ymax = homeGoals), alpha = 0.3) +
    labs(title = 'Home Goals vs Away Goals') +
    ylab('Goals') +
    labs(fill = "", color = '') +
    scale_fill_discrete(labels = c("", "Home Goals", "Total Goals"))+
    theme(panel.background = element_blank(),
          plot.title = element_text(hjust = 0.5, face = "bold"),
          axis.text.x = element_text(size=10, color = "black"),
          axis.title.x = element_text(size=11, face="bold"),
          axis.title.y = element_text(size=11, face="bold"))

homeVsAway

# Reorganizing data for the next graph 
hgPlot = homeAwayGoals[1:2]
hgPlot$type = 'HG'
colnames(hgPlot) = c('Season','Goals','Type')
agPlot = homeAwayGoals[c(1,3)]
agPlot$type = 'AG'
colnames(agPlot) = c('Season','Goals','Type')
ttPlot = homeAwayGoals[c(1,4)]
ttPlot$type = 'TT'
colnames(ttPlot) = c('Season','Goals','Type')

fullPlot = rbind(hgPlot,agPlot,ttPlot)

# Plot comparing Home, Away and Total goals per season
allGoalsPlot = ggplot(fullPlot, aes(x = Goals, y = reorder(as.factor(Season),-Season))) +
    geom_bar(aes (fill= Type), width = 0.8,
             stat = 'identity',
             position = position_dodge()) +
    labs(title = 'Goal Comparison') +
    theme(panel.background = element_blank(),
          plot.title = element_text(hjust = 0.5, face = "bold")) +
    ylab('') +
    xlab('Goals') +
    theme(axis.text.x = element_text(size=10, angle=0, hjust = .5),
          axis.text.y = element_text(face="bold",size=10),
          axis.title.x = element_text(size=11, face="bold"))+
    labs(fill = "") +
    theme(legend.position = c(.85,0.5)) + 
    scale_fill_discrete(labels = c("Away Goals", "Home Goals", "Total Goals")) # rename labels

allGoalsPlot

# Teams that scored the highest sum of goals across all seasons
homeGoals_team = aggregate(list(HG = br$HG), by = list(Team = br$Home), FUN = sum)
awayGoals_team = aggregate(list(AG = br$AG), by = list(Team = br$Away), FUN = sum)
goalsByTeam = merge(homeGoals_team, awayGoals_team)
goalsByTeam$Total = goalsByTeam$HG + goalsByTeam$AG
goalsByTeam = goalsByTeam[order(-goalsByTeam$Total),]

goalsByTeam = merge(goalsByTeam,totalAppearences)
goalsByTeam$GoalsPerGame = goalsByTeam$Total / goalsByTeam$Appearences
goalsByTeam = goalsByTeam[order(-goalsByTeam$GoalsPerGame),]

# Top 10 goals per game
top10Goals = goalsByTeam[1:10,]

# Highest average goal scoring per game by team plot
top10Goals %>%
    mutate(highlight = ifelse(Team == 'Barueri', T,F)) %>%
    ggplot(aes(x = GoalsPerGame, y = reorder(Team,GoalsPerGame), fill = Team)) +
    geom_bar(stat = 'identity', aes(fill =  highlight)) +
    scale_fill_manual(values = c('#52CC71', '#D83917',
                                 '#52CC71', '#D83917',
                                 '#52CC71', '#D83917',
                                 '#52CC71', '#D83917',
                                 '#52CC71', '#D83917',
                                 '#52CC71', '#D83917'))
    theme(panel.background = element_blank(),
          plot.title = element_text(hjust = 0.5, face = "bold")) +
    geom_text(aes(label = format(round(GoalsPerGame,2))),
              vjust = +0.5, hjust = +1.2,
              size = 5, color = '#ffffff') +
    labs(title = 'Goals Scored per Game Played') + 
    ylab('') +
    xlab('Goals Scored per Game') +
    theme(legend.position="none",
          axis.text.x = element_text(face="bold",size=10, angle=0, hjust = .5),
          axis.text.y = element_text(size=10),
          axis.title.x = element_text(size=11, face="bold"))

# Teams that conceded the highest sum of goals across all seasons
cededGoals_homeTeam = aggregate(list(cededGoalsHome = br$AG), by = list(Team = br$Home), FUN = sum)
cededGoals_awayTeam = aggregate(list(cededGoalsAway = br$HG), by = list(Team = br$Away), FUN = sum)
cededGoalsByTeam = merge(cededGoals_homeTeam, cededGoals_awayTeam)
cededGoalsByTeam$Total = cededGoalsByTeam$cededGoalsHome + cededGoalsByTeam$cededGoalsAway

cededGoalsByTeam = merge(cededGoalsByTeam,totalAppearences)
cededGoalsByTeam$cededGoalsPerGame = cededGoalsByTeam$Total / cededGoalsByTeam$Appearences
cededGoalsByTeam = cededGoalsByTeam[order(-cededGoalsByTeam$cededGoalsPerGame),]

# Top 10 conceded goals per game
top10CededGoals = cededGoalsByTeam[1:10,]

# Highest average ceded goals per game by team
top10CededGoals %>%
    mutate(highlight = ifelse(Team == 'America-Rn', T,F)) %>%
    ggplot(aes(x = cededGoalsPerGame, 
               y = reorder(Team,cededGoalsPerGame), fill = Team)) +
    geom_bar(stat = 'identity', aes(fill =  highlight)) +
    scale_fill_manual(values = c('#52CC71', '#D83917',
                                 '#52CC71', '#D83917',
                                 '#52CC71', '#D83917',
                                 '#52CC71', '#D83917',
                                 '#52CC71', '#D83917',
                                 '#52CC71', '#D83917'))
    theme(panel.background = element_blank(),
          plot.title = element_text(hjust = 0.5, face = "bold")) +
    geom_text(aes(label = format(round(cededGoalsPerGame,2))), 
              vjust = +0.5, hjust = +1.2, size = 5, color = '#ffffff') +
    labs(title = 'Ceded Goals per Game Played') +
    ylab('') +
    xlab('Ceded Goals') +
    theme(legend.position="none",
          axis.text.x = element_text(face="bold",size=10, angle=0, hjust = .5),
          axis.text.y = element_text(size=10),
          axis.title.x = element_text(size=11, face="bold"))

# Best Scoring Teams per Season
homeScoreSeason = aggregate(list(Home = br$HG), by = list(Team = br$Home, Season = br$Season), FUN = sum)
awayScoreSeason = aggregate(list(Away = br$AG), by = list(Team = br$Away, Season = br$Season), FUN = sum)

# Total goals every team scored by season
totalScoreSeason = merge(homeScoreSeason,awayScoreSeason)
totalScoreSeason$Goals = totalScoreSeason$Home + totalScoreSeason$Away

topScorersPerSeason = aggregate(list(Goals = totalScoreSeason$Goals), 
                                by = list(Season = totalScoreSeason$Season), FUN = max)

bestAttacksPerSeason = merge(topScorersPerSeason, totalScoreSeason, by = c('Goals','Season'), all = F)

# Best attack graph
bestAttack = ggplot(bestAttacksPerSeason, aes(x = as.factor(Season), 
                                              y = Goals, fill = Team)) +
    geom_bar(stat = 'identity', width = 0.8) +
    theme(panel.background = element_blank(),
          plot.title = element_text(hjust = 0.5, face = "bold")) +
    labs(title = "Highest Total Goals per Season", fill = '') +
    ylab('Goals') +
    xlab('Season') +
    geom_text(aes(label = Goals), 
              vjust = +1, hjust = +0.5, size = 3.5, color = '#ffffff') +
    scale_x_discrete(labels=c("03","04","05","06","07","08","09","10","11","12",
                              "13","14","15","16","17","18","19","20","21")) +
    theme(axis.text.x = element_text(face="bold",size=10, angle=0, hjust = .5),
          axis.text.y = element_text(size=10),
          axis.title.x = element_text(size=11, face="bold"),
          axis.title.y = element_text(size=11, face="bold"))

bestAttack

#### RESULTS ####

brPoints = br

# Assigning Home Points
homePoints = function (result){
    if (result == 'H') {
        brPoints$homePoints = 3
    } else if (result == 'A'){
        brPoints$homePoints = 0
    } else {
        brPoints$homePoints = 1
    }
}

brPoints$homePoints = mapply(homePoints, brPoints$Result)

# Assigning Away Points
awayPoints = function (result){
    if (result == 'A') {
        brPoints$awayPoints = 3
    } else if (result == 'H'){
        brPoints$awayPoints = 0
    } else {
        brPoints$awayPoints = 1
    }
}

brPoints$awayPoints = mapply(awayPoints, brPoints$Result)

#### CHAMPIONS ####

# Champions per season (highest sum of points)
homePerSeason = aggregate(list(HP = brPoints$homePoints), 
                          list(Team = brPoints$Home, 
                               Season = brPoints$Season), FUN = sum)

awayPerSeason = aggregate(list(AP = brPoints$awayPoints), 
                          list(Team = brPoints$Away, 
                               Season = brPoints$Season), FUN = sum)

# Points summed across seasons by team
totalPoints = merge(homePerSeason, awayPerSeason)
totalPoints$Points = totalPoints$HP + totalPoints$AP


# Champions
champions = aggregate(list(Points = totalPoints$Points), 
                      list(Season = totalPoints$Season), FUN = max)
champions = merge(champions, totalPoints)

homeGames = aggregate(list(homeGames = br$Home), by = list(Team = br$Home, Season = br$Season), FUN = length)
awayGames = aggregate(list(awayGames = br$Away), by = list(Team = br$Away, Season = br$Season), FUN = length)

champions = merge(champions, homeGames)
champions = merge(champions, awayGames)
champions$totalGames = champions$homeGames + champions$awayGames
champions$pointsPossible = champions$totalGames*3
champions$performanceRate = champions$Points/champions$pointsPossible

# Team that won with the most points
highestChamp = data.frame(Points = max(champions$Points))
highestChamp = merge(highestChamp, champions, by = c('Points'), all = F)

# Team that won with fewer points
lowestChamp = data.frame(Points = min(champions$Points))
lowestChamp = merge(lowestChamp, champions, by = c('Points'), all = F)

# Total points per champion plot
champPoints = ggplot(champions, aes(x = as.factor(Season), y = Points, fill= Team)) +
    geom_bar(stat = 'identity', width = 0.8) +
    theme(panel.background = element_blank(),
          plot.title = element_text(hjust = 0.5, face = "bold")) +
    labs(title = "Champions' Total Points", fill ='') +
    ylab('Total Points') +
    xlab('Season') +
    geom_text(aes(label = Points), 
              vjust = +1, hjust = +0.5, size = 3.5, color = '#ffffff') +
    # Shortening season year
    scale_x_discrete(labels=c("03","04","05","06","07","08","09","10","11","12",
                              "13","14","15","16","17","18","19","20","21"))+
    theme(axis.text.x = element_text(face="bold",size=10, angle=0, hjust = .5),
          axis.text.y = element_text(size=10),
          axis.title.x = element_text(size=11, face="bold"),
          axis.title.y = element_text(size=11, face="bold"))

champPoints

# Total points per champion plot highlighted
champPointsHL = ggplot(champions, aes(x = as.factor(Season), 
                                      y = Points, 
                                      fill = as.factor(Points), label = Team)) +
    geom_bar(stat = 'identity', width = 0.8) +
    theme(panel.background = element_blank(),
          plot.title = element_text(hjust = 0.5, face = "bold")) +
    labs(title = "Champions' Total Points", fill ='') +
    ylab('Total Points') +
    xlab('Season') +
    # Displaying team names
    geom_text(aes(label = ifelse(Points == 100, Team, "")),
              vjust = +0.3, angle=90, hjust = +3, size = 4, color = '#ffffff') +
    geom_text(aes(label = ifelse(Points == 67, Team, "")),
              vjust = +0.3, angle=90, hjust = +2, size = 4, color = '#ffffff') +
    # Displaying points
    geom_text(aes(label = ifelse(Points == 100, Points, "")),
              vjust = +1, hjust = +0.5, size = 3.5, color = '#ffffff') +
    geom_text(aes(label = ifelse(Points == 67, Points, "")),
              vjust = +1, hjust = +0.5, size = 3.5, color = '#ffffff') +
    # Shortening season year
    scale_x_discrete(labels=c("03","04","05","06","07","08","09","10","11","12",
                              "13","14","15","16","17","18","19","20","21"))+
    theme(legend.position="none",
          axis.text.x = element_text(face="bold",
                                     size=10, angle=0, hjust = .5),
          axis.text.y = element_text(size=10),
          axis.title.x = element_text(size=11, face="bold"),
          axis.title.y = element_text(size=11, face="bold")) +
    scale_fill_manual(values = c("#A4160D",
                                 "#52CC71",
                                 "#52CC71",
                                 "#52CC71",
                                 "#52CC71",
                                 "#52CC71",
                                 "#52CC71",
                                 "#52CC71",
                                 "#52CC71",
                                 "#52CC71",
                                 "#52CC71",
                                 "#52CC71",
                                 "#D83917"))
champPointsHL

# Team that won with the highest performance rate
highestPercChamp = data.frame(performanceRate = max(champions$performanceRate))
highestPercChamp = merge(highestPercChamp, champions, by = c('performanceRate'), all = F)

# Team that won with the lowest performance rate
lowestPercChamp = data.frame(performanceRate = min(champions$performanceRate))
lowestPercChamp = merge(lowestPercChamp, champions, by = c('performanceRate'), all = F)

# Performance rate per champion plot
champPerc = ggplot(champions, aes(x = as.factor(Season), y = performanceRate, fill= Team)) +
    geom_bar(stat = 'identity', width = 0.8) +
    theme(panel.background = element_blank(),
          plot.title = element_text(hjust = 0.5, face = "bold")) +
    labs(title = "Champions' Performance Rate", fill = '') +
    ylab('% Points') +
    xlab('Season') +
    geom_text(aes(label = (100*round(performanceRate,3))), 
              vjust = +1, hjust = +0.5, size = 3, color = '#ffffff') +
    # Shortening season year
    scale_x_discrete(labels=c("03","04","05","06","07","08","09","10","11","12",
                              "13","14","15","16","17","18","19","20","21")) +
    theme(axis.text.x = element_text(face="bold",size=10, angle=0, hjust = .5),
          axis.text.y = element_text(size=10),
          axis.title.x = element_text(size=11, face="bold"),
          axis.title.y = element_text(size=11, face="bold"))

champPerc

# Performance rate per champion plot highlighted
champPerc_HL = ggplot(champions, aes(x = as.factor(Season), 
                                     y = performanceRate, 
                                     fill = as.factor(performanceRate), label = Team)) +
    geom_bar(stat = 'identity', width = 0.8) +
    theme(panel.background = element_blank(),
          plot.title = element_text(hjust = 0.5, face = "bold")) +
    labs(title = "Champions' Performance Rate", fill ='') +
    ylab('% Points') +
    xlab('Season') +
    # Displaying team names
    geom_text(aes(label = ifelse((100*round(performanceRate,3)) > 78, Team, "")),
              vjust = +0.3, angle=90, hjust = +3, size = 4, color = '#ffffff') +
    geom_text(aes(label = ifelse((100*round(performanceRate,3)) < 60, Team, "")),
              vjust = +0.3, angle=90, hjust = +2, size = 4, color = '#ffffff') +
    # Displaying points
    geom_text(aes(label = ifelse((100*round(performanceRate,3)) > 78, 
                                 (100*round(performanceRate,3)), "")),
              vjust = +1, hjust = +0.5, size = 3.5, color = '#ffffff') +
    geom_text(aes(label = ifelse((100*round(performanceRate,3)) < 60, 
                                 (100*round(performanceRate,3)), "")),
              vjust = +1, hjust = +0.5, size = 3.5, color = '#ffffff') +
    # Shortening season year
    scale_x_discrete(labels=c("03","04","05","06","07","08","09","10","11","12",
                              "13","14","15","16","17","18","19","20","21"))+
    theme(legend.position="none",
          axis.text.x = element_text(face="bold",
                                     size=10, angle=0, hjust = .5),
          axis.text.y = element_text(size=10),
          axis.title.x = element_text(size=11, face="bold"),
          axis.title.y = element_text(size=11, face="bold")) +
    scale_fill_manual(values = c("#A4160D",
                                 "#52CC71",
                                 "#52CC71",
                                 "#52CC71",
                                 "#52CC71",
                                 "#52CC71",
                                 "#52CC71",
                                 "#52CC71",
                                 "#52CC71",
                                 "#52CC71",
                                 "#52CC71",
                                 "#52CC71",
                                 "#52CC71",
                                 "#D83917"))

champPerc_HL

# Titles
titles = aggregate(x = list(numTitles = champions$Team), by = list(Team = champions$Team), FUN = length)

# Pie Chart Titles
allChamps = ggplot(titles, aes(x = '', y = reorder(numTitles,Team), fill = Team)) +
    geom_bar(stat="identity", width=1, color = 'white') +
    coord_polar("y", start=0) + 
    theme_void() +
    labs(title = 'League Titles', fill = '')+
    theme(panel.background = element_blank(),
          plot.title = element_text(hjust = 0.5, face = "bold")) +
    geom_text(aes(label = numTitles, x = 1.3),
              position = position_stack(vjust = .5), size = 6, color = '#ffffff')

allChamps

```
