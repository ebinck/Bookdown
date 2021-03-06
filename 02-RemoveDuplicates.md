# Remove Duplicates

I was provided all of the AIM data by the BLM in a tall table for easy analysis.  However, after beginning to work with the data, I realized there were a number of duplicate records for some reason.  As a result, my first step was to remove data from all sites that had any duplicate records.  Even if there was one duplicate reading in a site, the reasons seemed to be variable, and I determined it was more efficient to remove all of the data for those sites than to try to fix the issue.  Additionally, I decided it was better to omit the data than try to alter it in a way that may not be accurate in relation to on the ground field conditions. 



## Load the data


```r
lpi_tall<-readRDS("/Users/elinbinck/Documents/Grad_School/Thesis/R_project/Thesis_Research/data/AIM_tall_tables_export_2021-09-21/lpi_tall.Rdata")%>%
  rename(SpeciesCode = code)

header <- readRDS("/Users/elinbinck/Documents/Grad_School/Thesis/R_project/Thesis_Research/data/AIM_tall_tables_export_2021-09-21/header.Rdata")
```

### Join lpi with corresponding states


```r
PrimKeyState <- header[,c("PrimaryKey", "State")]

lpi_tall2<-lpi_tall %>%
  left_join(PrimKeyState) %>% 
  rename(SpeciesState = State) %>% 
  select(-STATE, -SAGEBRUSH_SPP)
```

```
## Joining, by = "PrimaryKey"
```

## Break up lpi_tall to investigate duplicates


```r
#write a function to calculate the number of unique vs total rows for a given state
state_dups<-function(state) {
  dups<-lpi_tall2%>%
    filter(SpeciesState == state)
  print(n_distinct(dups))
  print(nrow(dups))
}

states<-unique(lpi_tall2$SpeciesState)
states_list<-setNames(vector("list", length(states)), states)

#use a for loop to quickly calculate them for each state
for (i in seq_along(states)){
  N<-state_dups(states[i])
  print(states[i])
  print(N)
}
```

```
## [1] 2329459
## [1] 2329459
## [1] "NV"
## [1] 2329459
## [1] 1291815
## [1] 1291856
## [1] "CO"
## [1] 1291856
## [1] 84073
## [1] 84073
## [1] "AK"
## [1] 84073
## [1] 220373
## [1] 220373
## [1] "AZ"
## [1] 220373
## [1] 622846
## [1] 622846
## [1] "CA"
## [1] 622846
## [1] 82541
## [1] 82546
## [1] "SD"
## [1] 82546
## [1] 1035623
## [1] 1035628
## [1] "ID"
## [1] 1035628
## [1] 1187917
## [1] 1187917
## [1] "WY"
## [1] 1187917
## [1] 10844
## [1] 10844
## [1] "ND"
## [1] 10844
## [1] 984421
## [1] 984421
## [1] "UT"
## [1] 984421
## [1] 804749
## [1] 807959
## [1] "MT"
## [1] 807959
## [1] 1427307
## [1] 1427307
## [1] "OR"
## [1] 1427307
## [1] 361284
## [1] 361284
## [1] "WA"
## [1] 361284
## [1] 0
## [1] 0
## [1] NA
## [1] 0
## [1] 785171
## [1] 785171
## [1] "NM"
## [1] 785171
```

## Create objects for duplicates for each state

#### South Dakota

```r
SDrows<-lpi_tall2%>%
   filter(SpeciesState == "SD")

SDdups<- SDrows[duplicated(SDrows),]
```

#### Montana


```r
MTrows<-lpi_tall2%>%
   filter(SpeciesState == "MT")

MTdups<- MTrows[duplicated(MTrows),]
```

#### Colorado


```r
COrows<-lpi_tall2%>%
   filter(SpeciesState == "CO")

COdups<- COrows[duplicated(COrows),]
```

#### Idaho



```r
IDrows<-lpi_tall2%>%
   filter(SpeciesState == "ID")

IDdups<- IDrows[duplicated(IDrows),]
```


## Remove plots with duplicates


### Create a df with all the PrimaryKeys from each dup file for each state


```r
uniqueSDdups<-SDdups %>% 
  distinct(PrimaryKey)

uniqueMTdups<-MTdups %>% 
  distinct(PrimaryKey)

uniqueCOdups<-COdups %>% 
  distinct(PrimaryKey)

uniqueIDdups<-IDdups %>% 
  distinct(PrimaryKey)

dupPrimaryKeys<-rbind(uniqueSDdups, uniqueMTdups, uniqueCOdups, uniqueIDdups)

dupPrimaryKeys<-as.vector(dupPrimaryKeys$PrimaryKey)
```


### Remove all plots that have any duplicate values


```r
for (i in seq_along(dupPrimaryKeys)){
  lpi_tall2<-lpi_tall2 %>% 
    filter(PrimaryKey !=dupPrimaryKeys[i])
}

#confirm that that worked and removed all entries with those PrimaryKeys
#also check to see how many plots were removed - it looks like less than 100 which is good

n_distinct(lpi_tall$PrimaryKey)
```

```
## [1] 36314
```

```r
n_distinct(lpi_tall2$PrimaryKey)
```

```
## [1] 36232
```


### Check for more duplicates


```r
#No more duplicates exist!
n_distinct(lpi_tall2)
```

```
## [1] 11229988
```

