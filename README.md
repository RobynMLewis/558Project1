# 558Project1
NHL API Project

---
title: "Project 1"
author: "Robyn Lewis"
date: "9/5/2020"
output: 
  rmarkdown::github_document:
    toc: true
---

```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = TRUE)
NHLData.Rmd <- "C:/Users/robyn/Documents/NCSU Statistics/558/Projects/Project1/NHLData.Rmd"
rmarkdown::render("NHLData.Rmd", output_file = "README.md")
```

```{r, echo=FALSE}
library(httr)
library(jsonlite)
library(tidyverse)
library(tibble)
library(rmarkdown)
library(knitr)
library(ggplot2)
library(car)
library(dplyr)
base <- "https://records.nhl.com/site/api"
base2 <- "https://statsapi.web.nhl.com/api/v1/teams"
teams_names <- c("First Season ID", "Last Season ID", "Most Recent Team ID", "Team Common Name", "Team Place Name")
stats_names <- c("Active Franchise", "First Season ID", "Franchise ID", "Game Type", "Games Played", "Goals Against", "Goals For", "Home Losses", "Home OT Losses", "Home Ties", "Home Wins", "Last Season ID", "Losses", "OT Losses", "Penalty Minutes", "Point Percentage", "Points", "Road Losses", "Road OT Losses", "Road Ties", "Road Wins", "Shootout Losses", "Shootout Wins", "Shutouts", "Team Name", "Ties", "TriCode", "Wins")
records_names <- c("Fewest Goals", "Fewest Goals Against", "Fewest Goals Against Seasons", "Fewest Goals Seasons", "Fewest Losses", "Fewest Losses Seasons", "Fewest Points", "Fewest Points Seasons", "Fewest Ties", "Fewest Ties Seasons", "Fewest Wins", "Fewest Wins Seasons", "Franchise ID", "Franchise Name", "Home Loss Streak", "Home Loss Streak Dates", "Home Point Streak", "Home Point Streak Dates", "Home Win Streak", "Home Win Streak Dates", "Home Winless Streak", "Home Winless Streak Dates", "Loss Streak", "Loss Streak Dates", "Most Game Goals", "Most Game Goals Dates", "Most Goals", "Most Goals Against", "Most Goals Against Seasons", "Most Goals Seasons", "Most Losses", "Most Losses Seasons", "Most Penalty Minutes", "Most Penalty Minutes Seasons", "Most Points", "Most Points Seasons", "Most Shutouts", "Most Shutouts Seasons", "Most Ties", "Most Ties Seasons", "Most Wins", "Most Wins Seasons", "Point Streak", "Point Streak Dates", "Road Loss Streak", "Road Loss Streak Dates", "Road Point Streak", "Road Point Streak Dates", "Road Win Streak", "Road Win Streak Dates", "Road Winless Streak", "Road Winless Streak Dates",
"Win Streak", "Win Streak Dates", "Winless Streak", "Winless  Streak Dates")
goalie_names <- c("Active Player", "First Name", "Franchise ID", "Franchise Name", "Game Type ID", "Games Played", "Last Name", "Losses", "Most Goals Against Dates", "Most Goals Against One Game", "Most Saves Date", "Most Saves One Game", "Most Shots Against Dates", "Most Shots Against One Game", "Most Shutouts One Season", "Most Shutouts Season IDs", "Most Wins One Season", "Most Wins Season IDs", "Overtime Losses", "Player ID", "Rookie Games Played", "Rookie Shutouts", "Rookie Wins", "Seasons", "Shutouts", "Ties", "Wins")
skater_names <- c("Active Player", "Assists", "First Name", "Franchise ID", "Franchise Name", "Game Type ID", "Games Played", "Goals", "Last Name", "Most Assists Game Dates", "Most Assists One Game", "Most Assists One Season", "Most Assists Season ID", "Most Goals Game Dates", "Most Goals One Game", "Most Goals One Season", "Most Goals Season IDs", "Most Penalty Minutes One Season", "Most Penalty Minutes Season IDs", "Most Points Game Dates", "Most Points One Game", "Most Points One Season", "Most Points Season IDs", "Penalty Minutes", "Player ID", "Points", "Position Code", "
```
# Creating Some Functions
To retrieve Franchise information about every team in the history of the NHL
```{r franchiseFunction}
#franchise by name OR number
  #if(is.numeric(input)){call franchise by number}
  #else if (is.character(input)){call franchise by name}
#endpoint1: franchise, returns id, firstSeasonId, lastSeasonId, name of every team in the history of the NHL
listTeams <- function(){
  get_teams <- GET(paste0(base, "/franchise"))
  get_teams_text <- content(get_teams, "text")
  get_teams_json <- fromJSON(get_teams_text, flatten= TRUE)
  get_teams_df <- as.data.frame(get_teams_json)
  get_teams_df <- get_teams_df[, -c(1,7)]
    colnames(get_teams_df) <- teams_names
  get_teams_df <- get_teams_df[, c(5, 4, 3, 1, 2)]
}
```
To retrieve stats about every franchise
```{r teamTotalsFunction}
#endpoint2: franchise-team-totals, returns Total stats for every franchise (ex roadTies, roadWins, etc)
teamStats <- function(){
  get_stats <- GET(paste0(base, "/franchise-team-totals"))
  get_stats_text <- content(get_stats, "text")
  get_stats_json <- fromJSON(get_stats_text, flatten= TRUE)
  get_stats_df <- as.data.frame(get_stats_json)
  get_stats_df <- get_stats_df[, -c(1, 26, 31)]
    colnames(get_stats_df) <- stats_names
  get_stats_df <- get_stats_df[, c(25, 27, 5, 4, 28, 13, 14, 26, 24, 6, 7, 17, 16, 23, 22, 15, 11, 8, 9, 10, 21, 18, 19, 20, 3, 2, 12, 1)]
}
```
```{r, echo=FALSE}
compStats <-teamStats()
franchiseIDs <- compStats[, c(1, 25, 28)]
```
To retrieve season records for a specified team
```{r seasonRecords}
#endpoint3: franchise-season-records?cayenneExp=franchiseID=ID (drill down into season records for specific franchise)
seasonRecords <- function(ID){
  get_records <- GET(paste0(base, "/franchise-season-records?cayenneExp=franchiseId=", ID))
  get_records_text <- content(get_records, "text")
  get_records_json <- fromJSON(get_records_text)
  get_records_df <- as.data.frame(get_records_json)
  get_records_df <- get_records_df[, -c(1, 58)]
    colnames(get_records_df) <- records_names
  get_records_df <- get_records_df[, c(14, 13, 41, 42, 11, 12, 53, 54, 19, 20, 49, 50, 31, 32, 5, 6, 23, 24, 15, 16, 45, 46, 39, 40, 9, 10, 37, 38, 35, 36, 7, 8, 43, 44, 17, 18, 47, 48, 55, 56, 21, 22, 51, 52, 27, 30, 25, 26, 1, 4, 28, 29, 2, 3, 33, 34)]
}
```
To retrieve goalie records for a specified team
```{r goalieRecords}
#endpoint4: franchise-goalie-records?cayenneExp=franchiseID=ID (goalie records for specified franchise)
goalieRecords <- function(ID){
  get_goalie <- GET(paste0(base, "/franchise-goalie-records?cayenneExp=franchiseId=", ID))
  get_goalie_text <- content(get_goalie, "text")
  get_goalie_json <- fromJSON(get_goalie_text)
  get_goalie_df <- as.data.frame(get_goalie_json)
  get_goalie_df <- get_goalie_df[, -c(1, 22, 30)]
    colnames(get_goalie_df) <- goalie_names
  get_goalie_df <- get_goalie_df[, c(2, 7, 4, 3, 20, 1, 24, 6, 5, 27, 8, 26, 25, 21, 23, 22, 17, 18, 12, 11, 15, 16, 14, 13, 10, 9, 19)]
}
```
To retrieve skater records for a specified team
```{r skaterRecords}
#endpoint5: franchise-skater-records?cayenneExp=franchiseID=ID (skater records, same interaction as goalie endpoint)
skaterRecords <- function(ID){
  get_skater <- GET(paste0(base, "/franchise-skater-records?cayenneExp=franchiseId=", ID))
  get_skater_text <- content(get_skater, "text")
  get_skater_json <- fromJSON(get_skater_text)
  get_skater_df <- as.data.frame(get_skater_json)
  get_skater_df <- get_skater_df[, -c(1, 31)]
    colnames(get_skater_df) <- skater_names
  get_skater_df <- get_skater_df[, c(3, 9, 5, 4, 25, 27, 29, 1, 7, 6, 8, 15, 14, 16, 17, 2, 11, 10, 12, 13, 26, 28, 21, 20, 22, 23, 24, 18, 19)]
}
```
```{r, echo=FALSE}
statsEndpoints <- list("/roster", "?expand=person.names", "?expand=team.schedule.next", "?expand=team.schedule.previous", "?expand=team.stats", "?expand=team.roster&")
```
To drill down into more stats from the Stats API
```{r statsAPI}
moreStats <- function(ID=NULL, modifier=NULL, season=NULL, moreTeams=NULL){
  if (is.null(ID)){
  get_moreStats <- GET(base2)
  }
  else if (!is.null(ID)){
    if(is.null(modifier)){
      get_moreStats <- GET(paste0(base2, "/", ID))}
    else if (!is.null(modifier)){
      switch(modifier, statsEndpoints)
    get_moreStats <- GET(paste0(base2, "/", ID, modifier))}
    
  get_moreStats_text <- content(get_moreStats, "text")
  get_moreStats_json <- fromJSON(get_moreStats_text)
  get_moreStats_df <- as.data.frame(get_moreStats_json)
  get_moreStats_df <- get_moreStats_df[, -1]
  view(get_moreStats_df)
  }
}
```
Creating a wrapper function to access all previous functions
```{r wrapper}
#function should call appropriate endpoint as per users request
getMyStats <- function(x, teamID=NULL, modifier=NULL, season=NULL, moreTeams=NULL){
  if(is.null(teamID)){x()}
  else if (!is.null(teamID)){x(teamID, modifier, season, moreTeams)}
}
```
## Data Analysis
```{r, echo=FALSE}
CANrecords <- NULL
for (i in CANteams){
  temp <- seasonRecords(i)
  CANrecords <- rbind(CANrecords, temp)
}
CANrecords <- rename(CANrecords, `Team Name`=`Franchise Name`)
```
Creating new variables and joins
```{r join}
#do a join on two returned data sets from different API endpoints
CanPlayoffs <- inner_join(CANrecords, compStats[compStats$`Game Type` == 3, c(1, 3, 5, 6,8)])
CanPlayoffs <- rename(CanPlayoffs, `Playoff Games Played` = `Games Played`)
CanPlayoffs <- rename(CanPlayoffs, `Playoff Wins`=`Wins`)
CanPlayoffs <- rename(CanPlayoffs, `Playoff Losses`=`Losses`)
CanPlayoffs <- rename(CanPlayoffs, `Playoff Ties`=`Ties`)
view(CanPlayoffs)
```
```{r newvars}
#Create at least two new variables
homeWinPct <- compStats$`Home Wins`/(compStats$`Home Wins` + compStats$`Home Losses` + compStats$`Home Ties`)
homeLossPct <- compStats$`Home Losses`/(compStats$`Home Wins` + compStats$`Home Losses` + compStats$`Home Ties`)
avgPointsperWin <- compStats$Points/compStats$Wins
avgPointsperLoss <- compStats$Points/compStats$Losses
compStats1 <-cbind(compStats, homeWinPct, homeLossPct, avgPointsperWin, avgPointsperLoss)
```
###Contingency tables
This shows a summation of how many teams are no longer active, and how many active teams still exist. While this is a very simple table, it was information previously unknown to me. 
```{r table1}
#Create some contingency tables
table1 <- table(recode(compStats1$`Active Franchise`, "'0'= 'Not Active'; '1'='Active'"))
kable(table1, col.names = NULL, caption="Active Franchises")
```
This shows how many games are played in the regular season versus playoff games. I was very suprised by the high number of playoff games compared to regular season games. 
```{r table2}
table2 <- table(recode(compStats1$`Game Type`, "'2'='Regular Season'; '3'='Playoffs'"))
kable(table2, col.names = c("Game Type", "Number of Games"))
```
This table gives an idea of how long non active teams played. This a very small number, so we can assume that non active teams did not play very long. 
```{r table3}
table3 <- table(recode(compStats1$`Active Franchise`, "'0'= 'Not Active'; '1'='Active'"), recode(compStats1$`Game Type`, "'2'='Regular Season'; '3'='Playoffs'"))
kable(table3, caption = "Playoff Games vs Regular Season Games by Active and Non-Active Teams")
```
## Numerical Summaries
Most sports team are assumed to have an advantage when playing on their home court. Here we explore which teams win more than 50% of their home games in the regular season. 
```{r numsum1}
#Create numerical summaries for some quantitative variables at each setting of some of your categorical vars
compStats1 %>% filter(homeWinPct>.5 & compStats1$`Active Franchise` == 1 & compStats1$`Game Type` == 2) %>% select(c(1, 2, 3, 5, 6, 8, 17, 29)) %>% kable(digits=4, caption="Teams with Home Win Percentage > 50% in the Regular Season")
```
Shutouts are the term for when one team prevents the other from scoring a single point. Initially I was surprised by the high number of shutouts, but this can be explained by the low overall final scores in hockey. We may expect to see a similar trend in sports like soccer, but it would be surprising to see this in a high scoring sport like basketball. 
```{r numsum2}
playoffStats <- as.tibble(compStats[compStats$`Active Franchise` == 1 & compStats$Shutouts !=0 & compStats$`Game Type`==3, c(1,9)])
playoffStats %>% arrange(desc(playoffStats$Shutouts)) %>% kable(caption="Number of Shutouts by Team in Playoff Games")
```
This summary reinforces the idea of the previous table. The average maximum points scored for a winning team is only 3 points. Interestingly, the average maximum points scored for a losing team is a bit higher. 
```{r numsum3}
kable(sapply(select(compStats1, homeWinPct:avgPointsperLoss), summary), col.names = c("% Home Game Won", "% Home Game Lost", "Avg Points per Win", "Avg Points per Loss"), digits=3, caption="Numerical Summary")
```
## Plots
```{r, echo=FALSE}
topTen <- compStats1 %>% arrange(desc(`Games Played`)) %>% filter(`Game Type` ==2)
topTen <- topTen[1:10, ]
```
While I knew the idea I intended to get across in this chart, the execution was unsuccessful. I intended to show the number of games won by the ten teams with the most overall wins. 
```{r barplot}
#At least five plots, use coloring and grouping, labels and titles
#At least one barplot, one histogram, one box plot, one scatterplot
ggplot(data=topTen, aes(x=`Games Played`)) +
  geom_bar(aes(color=`Team Name`), position ="dodge") +
  theme_minimal() +
  labs(title= "A Not Very Helpful Bar Chart")
```
Here we see the distribution of the percentage of home games won. We can see that on average, most teams win just a bit over half of their home games. It seems that home court advantage doesn't have much significance in hockey. 
```{r histogram}
ggplot(data=compStats1, aes(x=homeWinPct)) +
  geom_histogram(binwidth = .1, fill="blue") +
  geom_density(kernel="gaussian", color="grey", adjust=0.5) +
  scale_color_gradient() +
  theme_minimal() +
  labs(x="Percentage of Home Games Won", title="Distribution of Home Game Win Percentage")
```
Again, this boxplot reiterates that hockey is a low scoring game, resulting in a very small spread of points. We do see a difference here between scores that won games in the regular season versus scores that won games in the playoffs. The playoff game appear to be much lower scoring. 
```{r boxplot}
ggplot(data=compStats1, aes(x=as.factor(`Game Type`) , y= avgPointsperWin)) +
  geom_boxplot(aes(color=as.factor(`Game Type`))) +
  scale_color_discrete(name="Game Type", labels=c("Regular Season", "Playoffs")) +
  labs(x="Regular Season vs Playoffs", y="Average Points per Win", title="Avg Points per win in Regular Season vs  Playoff Games") +
  theme_minimal()
```
This plot shows the total number of penalty minutes a team has taken versus the number of losses they've had. There is clear evidence of a linear relationship, but it is not safe to assume that teams that lose more have more penalties without exploring further. Teams who have played more games would most likely have both higher losses and higher penalty minutes. 
```{r scatter}
ggplot(data=compStats1, aes(x=Losses, y=`Penalty Minutes`)) +
  theme_minimal() +
  geom_point(aes(group=`Game Type`), color="blue") +
  labs(title="Penalty Minutes vs Total Game Losses")
```
I attempted again to chart the top ten most winning teams, without success. This plot is nicer aesthetically, but still does not convey the correct information. 
```{r plot5}
ggplot(data=topTen, aes(x=`Team Name`)) +
  geom_dotplot(aes(y=`Games Played`, fill=`Team Name`)) +
  theme_minimal() +
  theme(axis.ticks.x=element_blank(), axis.text.x=element_blank()) +
  labs(x="Teams")
```
# Packages Used
```{r citations}
citation(jsonlite)
citation(knitr)
citation(tidyverse)
citation(tibble)
citation(knitr)
citation(ggplot2)
citation(dplyr)
```
