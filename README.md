##### I pulled the last 11 years of player ratings with >100 possessions (10 years to measure improvement with 2014 needed as a reference for 2015 improvement)
```r
library(dplyr)
years = c(15:24)
```
##### I saved those 11 years of data locally as CSV files with the format of `paste0("miya", year, ".csv")`. That's why the years are just 2 digit numeric
##### From there, would read in a year and the previous year's data with a `left_join`. My biggest issue is that because of transfers, I couldn't `join_by` both player and team. So I had to just `join_by` player and deal with the many-to-many issue. Not sure if you have a player ID under the hood to deal with that
```r
fullmiya = data.frame()

for(i in years){
  miya = read.csv(paste0("/Users/connorbradley/Desktop/basketball data/miya/miya",i,".csv"))
  miya = miya %>% select(name,team,obpr,dbpr,bpr)
  
  miyalag = read.csv(paste0("/Users/connorbradley/Desktop/basketball data/miya/miya",i-1,".csv"))
  miyalag = miyalag %>% 
    select(name,team,obpr,dbpr,bpr) %>% 
    rename(team_lag = team, 
           obpr_lag = obpr, 
           dbpr_lag = dbpr, 
           bpr_lag = bpr)
  
  miya = miya %>% 
    left_join(miyalag,join_by(name),relationship="many-to-many")
  
  fullmiya = rbind(fullmiya,miya)
}
```
##### Once all those years have been put together into a single data frame, filter out missing `obpr_lag` (just to get rid of freshmen year data) and create variable for improvements to OBPR, DBPR, and BPR
```r
fullmiya = fullmiya %>% filter(!is.na(obpr_lag)) %>% 
  mutate(obpr_change = obpr-obpr_lag,
         dbpr_change = dbpr-dbpr_lag,
         bpr_change = bpr-bpr_lag)
```

##### `group_by` team in and find averages to judge which teams saw most (average) improvement
```r
miyasum = fullmiya %>% group_by(team) %>%
  summarise(players = n(), 
            avg_obpr = mean(obpr_change),
            avg_dbpr = mean(obpr_change),
            avg_bpr = mean(bpr_change)) %>% 
  filter(players>=20) %>% 
  arrange(desc(avg_bpr))
```

<img width="493" alt="Screenshot 2024-05-31 at 8 28 25 PM" src="https://github.com/cobrastats/miya_player_improvement/assets/109628356/d3490c3d-c847-48ca-9def-300454936a80">

