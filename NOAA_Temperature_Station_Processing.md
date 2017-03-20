# 2016 Weather Data Exploratory Analysis
  
  

![](C:/Users/tterry/Documents/NOAA Data/Logo/T2Logo.jpg)

***
***

###NOAA Observation Station Processing

####Creating Master Observation Station Lists

***
***


###Project Synopsis

According to a report published on January 18, 2017 by the **National Aeronautics and Space Administration** (**NASA**) and the **National Oceanic and Atmospheric Administration** (**NOAA**):

> ...(the) Earth's 2016 surface temperatures were the warmest since modern record keeping began in 1880.

> Globally-averaged temperatures in 2016 were 1.78 degrees Fahrenheit (0.99 degrees Celsius) warmer than the mid-20th century mean. This makes 2016 the third year in a row to set a new record for global average surface temperatures.

Source: https://www.nasa.gov/press-release/nasa-noaa-data-show-2016-warmest-year-on-record-globally

***

The **2016 Weather Data Exploratory Analysis** project was started to review the raw data from **NOAA** and identify areas of uncertainty and their potential impact on reaching a greater than 95% scientific certainty.

This is **Part 1** of the **2016 Weather Data Exploratory Analysis**. 

###Stations Data

The data will be coming from the **NOAA** FTP site at the following links:

+ ftp://ftp.ncdc.noaa.gov/pub/data/ghcn/daily/

The specific files we will be using are:

+ ftp://ftp.ncdc.noaa.gov/pub/data/ghcn/daily/ghcnd-countries.txt
+ ftp://ftp.ncdc.noaa.gov/pub/data/ghcn/daily/ghcnd-inventory.txt
+ ftp://ftp.ncdc.noaa.gov/pub/data/ghcn/daily/ghcnd-stations.txt
+ ftp://ftp.ncdc.noaa.gov/pub/data/ghcn/daily/ghcnd-states.txt

The files are in a text format with specific layouts which are defined in the `readme.txt` file located at the following:

+ ftp://ftp.ncdc.noaa.gov/pub/data/ghcn/daily/readme.txt

The specific instructions for each file will be documented in *Appendix A: Station Files* and will come from the `readme.txt` file to make it clear how and why the file is being processed in a particular manner.

The master lists will consist of observation stations that collect temperature data, their respective geographic coordinates, country, state and location information, and years of operation. Three master lists will be created for use in future exploratory analysis and saved in `.Rds` format:

+ `master_stations_ww.rds`
+ `master_stations_us50.rds`
+ `master_stations_us48.rds`

***

**Libraries Required**

```r
library(dplyr)          # Data manipulation
library(knitr)          # Dynamic report generation
library(readr)          # Reading tabular data
library(stringi)        # String processing
```


  
***

####Read Station Files

All of the station files are `Fixed Width Format` text files so the `read_fwf` function from the `readr` package will be used.

Within each code chunk the file will be read using the beginning and ending positions of the variables defined, column names will be assigned, and any preliminary processing will be performed. This includes converting data to uppercase or selecting only the variables needed for the master lists. In some cases, needed variables exist in multiple files (`Latitude`, `Longitude`). Redundant variables will be removed from the latter data tables as part of the preliminary processing.

A brief view of the data will follow each processing step for clarification purposes.

**Read Inventory File**



```r
inputFile <- "~/RAW Stations/ghcnd-inventory.txt"
```


```r
ghcnd_inventory <- read_fwf(inputFile, fwf_positions(c(1,13,22,32,37,42),
                                                     c(11,20,30,35,40,45)))

colnames(ghcnd_inventory) <- c("ID",
                               "Latitude",
                               "Longitude",
                               "Element",
                               "FirstYear",
                               "LastYear")

kable(head(ghcnd_inventory, 5))
```



ID             Latitude   Longitude  Element    FirstYear   LastYear
------------  ---------  ----------  --------  ----------  ---------
ACW00011604     17.1167    -61.7833  TMAX            1949       1949
ACW00011604     17.1167    -61.7833  TMIN            1949       1949
ACW00011604     17.1167    -61.7833  PRCP            1949       1949
ACW00011604     17.1167    -61.7833  SNOW            1949       1949
ACW00011604     17.1167    -61.7833  SNWD            1949       1949

**Read Country File**



```r
inputFile <- "~/RAW Stations/ghcnd-countries.txt"
```


```r
ghcnd_countries <- read_fwf(inputFile, fwf_positions(c(1,4),
                                                     c(2,50)))

colnames(ghcnd_countries) <- c("CCode",
                               "Country")

ghcnd_countries <- ghcnd_countries %>%
                   transform(Country = toupper(Country))        ## Convert Country to uppercase   

kable(head(ghcnd_countries, 5))
```



CCode   Country              
------  ---------------------
AC      ANTIGUA AND BARBUDA  
AE      UNITED ARAB EMIRATES 
AF      AFGHANISTAN          
AG      ALGERIA              
AJ      AZERBAIJAN           

**Read Stations File**



```r
inputFile <- "~/RAW Stations/ghcnd-stations.txt"
```


```r
ghcnd_stations <- read_fwf(inputFile, fwf_positions(c( 1, 13, 22, 32, 39, 42, 73, 77, 81),
                                                     c(11, 20, 30, 37, 40, 71, 75, 79, 85)))

colnames(ghcnd_stations)  <- c("ID",
                               "Latitude",
                               "Longitude",
                               "Elevation",
                               "SCode",
                               "Location",
                               "GSNFlag",
                               "HCN-CRNFlag",
                               "WMOid")

ghcnd_stations <- ghcnd_stations %>%
                  transform(Location = toupper(Location)) %>%    ## Convert Location to uppercase
                  select(ID, Elevation, SCode, Location)         ## Select only the variables needed

kable(head(ghcnd_stations, 5))
```



ID             Elevation  SCode   Location              
------------  ----------  ------  ----------------------
ACW00011604         10.1  NA      ST JOHNS COOLIDGE FLD 
ACW00011647         19.2  NA      ST JOHNS              
AE000041196         34.0  NA      SHARJAH INTER. AIRP   
AEM00041194         10.4  NA      DUBAI INTL            
AEM00041217         26.8  NA      ABU DHABI INTL        

**Read States File**



```r
inputFile <- "~/RAW Stations/ghcnd-states.txt"
```


```r
ghcnd_states <- read_fwf(inputFile, fwf_positions(c( 1,  4),
                                                  c( 2, 50)))

colnames(ghcnd_states)  <- c("SCode",
                             "State")

ghcnd_states <- ghcnd_states %>%
                transform(State = toupper(State))                ## Convert State to uppercase

kable(head(ghcnd_states, 5))
```



SCode   State          
------  ---------------
AB      ALBERTA        
AK      ALASKA         
AL      ALABAMA        
AR      ARKANSAS       
AS      AMERICAN SAMOA 

***

### Process Stations Data

The project is focused on worldwide temperature data. Observation stations in the **NOAA** inventory record many different observation types, `Elements`, but this project is concerned with the `TAVG`, `TMAX`, and `TMIN` observations.

**Set Element Vector**

```r
temp_elements <- c("TAVG",
                   "TMAX",
                   "TMIN")
```

The `inventory` file is structured to have multiple entries for each observation station if they record more than one type of observation. The master list will contain a single entry for each station that meets the criteria of recording any of the values in the `temp_elements` vector.

The `dplyr` package will be used to filter, group, and reduce the entries to a single observation. Upon completion, the `Element` variable will be removed from the table since it will no longer be used. The final instruction will create a two character country code using the first two characters of the station `ID`. This variable will be used to join the data from the `countries` data.

**Process Inventory Data**

```r
ghcnd_inventory <- ghcnd_inventory %>%
                   filter(Element %in% temp_elements) %>%      ## Filter for temperature values
                   group_by(ID) %>%                            ## Group data by ID
                   slice(1L) %>%                               ## Reduce entries to a single observation
                   ungroup() %>%
                   select(-Element) %>%                        ## Remove the Element variable
                   mutate(CCode = substr(ID,1,2))              ## Create country code from ID

kable(head(ghcnd_inventory, 5))
```



ID             Latitude   Longitude   FirstYear   LastYear  CCode 
------------  ---------  ----------  ----------  ---------  ------
ACW00011604     17.1167    -61.7833        1949       1949  AC    
ACW00011647     17.1333    -61.7833        1961       1961  AC    
AE000041196     25.3330     55.5170        1944       2017  AE    
AEM00041194     25.2550     55.3640        1983       2017  AE    
AEM00041217     24.4330     54.6510        1983       2017  AE    

The `countries` data will be joined with the `inventory` data by the variable `CCode` using `dplyr`. Once complete, the `CCode` can be removed and `ghcnd_countries` can be removed from the working environment. 

**Join Countries Data to Inventory Data**

```r
ghcnd_inventory <- left_join(ghcnd_inventory, ghcnd_countries, by = "CCode") %>%
                   select(-CCode)

rm(ghcnd_countries)

kable(head(ghcnd_inventory, 5))
```



ID             Latitude   Longitude   FirstYear   LastYear  Country              
------------  ---------  ----------  ----------  ---------  ---------------------
ACW00011604     17.1167    -61.7833        1949       1949  ANTIGUA AND BARBUDA  
ACW00011647     17.1333    -61.7833        1961       1961  ANTIGUA AND BARBUDA  
AE000041196     25.3330     55.5170        1944       2017  UNITED ARAB EMIRATES 
AEM00041194     25.2550     55.3640        1983       2017  UNITED ARAB EMIRATES 
AEM00041217     24.4330     54.6510        1983       2017  UNITED ARAB EMIRATES 

The `stations` data can be joined to the `inventory` data using the variable `ID`. Once complete, `ghcnd_stations` can be removed from the working environment.

**Join Stations Data to Inventory Data**

```r
ghcnd_inventory <- left_join(ghcnd_inventory, ghcnd_stations, by = "ID")

rm(ghcnd_stations)

kable(head(ghcnd_inventory, 5))
```



ID             Latitude   Longitude   FirstYear   LastYear  Country                 Elevation  SCode   Location              
------------  ---------  ----------  ----------  ---------  ---------------------  ----------  ------  ----------------------
ACW00011604     17.1167    -61.7833        1949       1949  ANTIGUA AND BARBUDA          10.1  NA      ST JOHNS COOLIDGE FLD 
ACW00011647     17.1333    -61.7833        1961       1961  ANTIGUA AND BARBUDA          19.2  NA      ST JOHNS              
AE000041196     25.3330     55.5170        1944       2017  UNITED ARAB EMIRATES         34.0  NA      SHARJAH INTER. AIRP   
AEM00041194     25.2550     55.3640        1983       2017  UNITED ARAB EMIRATES         10.4  NA      DUBAI INTL            
AEM00041217     24.4330     54.6510        1983       2017  UNITED ARAB EMIRATES         26.8  NA      ABU DHABI INTL        

The final step in the station processing is to join the `states` data with the `inventory` data using the variable `SCode`. Once compete, the table will be reorganized to make viewing cleaner and `ghcnd_states` can be removed from the working environment.

**Join States Data to Inventory Data**

```r
ghcnd_inventory <- left_join(ghcnd_inventory, ghcnd_states, by = "SCode") %>%
                   select(ID, Latitude, Longitude, Elevation, Country,
                          State, Location, FirstYear, LastYear)

rm(ghcnd_states)

kable(head(ghcnd_inventory, 5))
```



ID             Latitude   Longitude   Elevation  Country                State   Location                 FirstYear   LastYear
------------  ---------  ----------  ----------  ---------------------  ------  ----------------------  ----------  ---------
ACW00011604     17.1167    -61.7833        10.1  ANTIGUA AND BARBUDA    NA      ST JOHNS COOLIDGE FLD         1949       1949
ACW00011647     17.1333    -61.7833        19.2  ANTIGUA AND BARBUDA    NA      ST JOHNS                      1961       1961
AE000041196     25.3330     55.5170        34.0  UNITED ARAB EMIRATES   NA      SHARJAH INTER. AIRP           1944       2017
AEM00041194     25.2550     55.3640        10.4  UNITED ARAB EMIRATES   NA      DUBAI INTL                    1983       2017
AEM00041217     24.4330     54.6510        26.8  UNITED ARAB EMIRATES   NA      ABU DHABI INTL                1983       2017

***

#### Clean Inventory Data Elevation Values

The recorded elevation is set to -999.9 for stations with missing values.

**Check For Missing Elevations**

```r
nrow(ghcnd_inventory %>%
     filter(Elevation == -999.9))
```

```
## [1] 447
```

There are `447` stations with missing `Elevation` recordings. These readings will be converted to `NA` (Not Available). If the value recorded is valid it will be converted from *meters* to *feet*. The conversion factor is `1` meter is equal to `3.28084` feet. 

**Convert Elevations**

```r
ghcnd_inventory <- ghcnd_inventory %>%
                   transform(Elevation = ifelse(Elevation == -999.9,
                                                NA,
                                                Elevation * 3.28084))

summary(ghcnd_inventory$Elevation)
```

```
##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max.    NA's 
## -1148.0   290.0   899.9  1776.0  2379.0 16510.0     447
```

The maximum and minimum elevation levels are within the known ranges of the highest and lowest points on Earth.

***

### Save Master Stations Data

The worlwide master list is complete and can now be saved.

**Save Master Worldwide Station List**



```r
saveRDS(ghcnd_inventory, "~/Temp Stations/master_stations_ww.rds")
```

***
  
To create the master lists for the 50 United States and 48 Contiguous Unites States the `ghcnd_inventory` file will need to be filtered. The first check to make is the number of states that are recorded as part of the U.s. 

**Check U.S. States**

```r
temp_us <- ghcnd_inventory %>%
           filter(Country == "UNITED STATES")

table(temp_us$State)
```

```
## 
##                     ALABAMA                      ALASKA 
##                         245                         850 
##                     ARIZONA                    ARKANSAS 
##                         554                         223 
##                  CALIFORNIA                    COLORADO 
##                        1252                         633 
##                 CONNECTICUT                    DELAWARE 
##                          80                          24 
##        DISTRICT OF COLUMBIA                     FLORIDA 
##                           1                         368 
##                     GEORGIA                      HAWAII 
##                         330                         308 
##                       IDAHO                    ILLINOIS 
##                         536                         313 
##                     INDIANA                        IOWA 
##                         260                         252 
##                      KANSAS                    KENTUCKY 
##                         362                         290 
##                   LOUISIANA                       MAINE 
##                         295                         205 
##                    MARYLAND               MASSACHUSETTS 
##                         227                         233 
##                    MICHIGAN                   MINNESOTA 
##                         514                         380 
##                 MISSISSIPPI                    MISSOURI 
##                         216                         360 
##                     MONTANA                    NEBRASKA 
##                         768                         359 
##                      NEVADA               NEW HAMPSHIRE 
##                         444                         159 
##                  NEW JERSEY                  NEW MEXICO 
##                         116                         542 
##                    NEW YORK              NORTH CAROLINA 
##                         548                         383 
##                NORTH DAKOTA                        OHIO 
##                         245                         340 
##                    OKLAHOMA                      OREGON 
##                         348                         759 
##             PACIFIC ISLANDS                PENNSYLVANIA 
##                           1                         493 
##                RHODE ISLAND              SOUTH CAROLINA 
##                          36                         190 
##                SOUTH DAKOTA                   TENNESSEE 
##                         304                         282 
##                       TEXAS U.S. MINOR OUTLYING ISLANDS 
##                         961                           1 
##                        UTAH                     VERMONT 
##                         633                         122 
##                    VIRGINIA                  WASHINGTON 
##                         332                         566 
##               WEST VIRGINIA                   WISCONSIN 
##                         273                         357 
##                     WYOMING 
##                         468
```

```r
rm(temp_us)
```


There are three "states" that can be removed from the data for the purposes of the future uses of the U.s. master lists.


**Create U.S. 50 Master List**

```r
rem_states <- c("DISTRICT OF COLUMBIA", "PACIFIC ISLANDS", "U.S. MINOR OUTLYING ISLANDS")

ghcnd_inventory <- ghcnd_inventory %>%
                   filter(Country == "UNITED STATES") %>%
                   filter(!State %in% rem_states)
```

**Save U.S. 50 Master List**



```r
saveRDS(ghcnd_inventory, "~/Temp Stations/master_stations_us50.rds")
```

***

The master list for the 48 Contiguous United States requires Alaska and Hawaii being removed.

**Create U.S. 48 Master List**

```r
rem_states <- c("ALASKA", "HAWAII")

ghcnd_inventory <- ghcnd_inventory %>%
                   filter(!State %in% rem_states)
```

**Save U.S. 48 Master List**



```r
saveRDS(ghcnd_inventory, "~/Temp Stations/master_stations_us48.rds")
```

***

**Clean Working Environment**

```r
rm(ghcnd_inventory)
rm(inputFile, rem_states, temp_elements)
```

***

### Next Steps

Using the master observation lists created in this project, a data exploration and analysis will be performed to review station densities and distributions around the world. 

In **Part 2**, the focus will be on Hemispheric, Quadrant, and country diversity along with land mass comparisons for density.

In **Part 3**, the focus will be on the stations present during the Mid-Century baseline years (1951-1980) compared to 2016. 

In **Part 4**, a review of the methodology that **NOAA** uses to aggregate data across geographic sectors will be performed. 

***

### Appendix A: Station Files

***

**FORMAT OF "ghcnd-countries.txt"**

| Variable | Columns |   Type    |
|----------|:-------:|-----------|
|CODE      |     1-2 | Character |
|NAME      |    4-50 | Character |

These variables have the following definitions:

+ CODE      - FIPS country code of the country where the station is located
+ NAME      - Name of the country

***

**FORMAT OF "ghcnd-inventory.txt"**

| Variable | Columns |   Type    |
|----------|:-------:|-----------|
|ID        |   1-11  | Character |
|LATITUDE  |  13-20  | Real      |
|LONGITUDE |  22-30  | Real      |
|ELEMENT   |  32-35  | Character |
|FIRSTYEAR |  37-40  | Integer   |
|LASTYEAR  |  42-45  | Integer   |

These variables have the following definitions:

+ ID         - station identification code
+ LATITUDE   - latitude of the station (in decimal degrees)
+ LONGITUDE  - longitude of the station (in decimal degrees)
+ ELEMENT    - element type
+ FIRSTYEAR  - first year of unflagged data for the given element
+ LASTYEAR   - last year of unflagged data for the given element

***

**FORMAT OF "ghcnd-stations.txt"**

| Variable    | Columns |   Type    |
|-------------|:-------:|-----------|
|ID           |   1-11  | Character |
|LATITUDE     |  13-20  | Real      |
|LONGITUDE    |  22-30  | Real      |
|ELEVATION    |  32-37  | Real      |
|STATE        |  39-40  | Character |
|NAME         |  42-71  | Character |
|GSN FLAG     |  73-75  | Character |
|HCN/CRN FLAG |  77-79  | Character |
|WMO ID       |  81-85  | Character |

***

**FORMAT OF "ghcnd-states.txt"**

| Variable    | Columns |   Type    |
|-------------|:-------:|-----------|
|CODE         |    1-2  | Character |
|NAME         |   4-50  | Character |

***
***

![](C:/Users/tterry/Documents/NOAA Data/Logo/T2Logo.jpg)

***


```r
sessionInfo()
```

```
## R version 3.3.2 (2016-10-31)
## Platform: x86_64-w64-mingw32/x64 (64-bit)
## Running under: Windows 10 x64 (build 14393)
## 
## locale:
## [1] LC_COLLATE=English_United States.1252 
## [2] LC_CTYPE=English_United States.1252   
## [3] LC_MONETARY=English_United States.1252
## [4] LC_NUMERIC=C                          
## [5] LC_TIME=English_United States.1252    
## 
## attached base packages:
## [1] stats     graphics  grDevices utils     datasets  methods   base     
## 
## other attached packages:
## [1] stringi_1.1.2 readr_1.0.0   knitr_1.15.1  dplyr_0.5.0  
## 
## loaded via a namespace (and not attached):
##  [1] Rcpp_0.12.9     digest_0.6.12   rprojroot_1.2   assertthat_0.1 
##  [5] R6_2.2.0        DBI_0.5-1       backports_1.0.5 magrittr_1.5   
##  [9] evaluate_0.10   highr_0.6       lazyeval_0.2.0  rmarkdown_1.3  
## [13] tools_3.3.2     stringr_1.2.0   yaml_2.1.14     htmltools_0.3.5
## [17] tibble_1.2
```



