Assignment2
================
Jiayi Nie
06/10/2021

### Data Wrangling

``` r
library(data.table)
if(!file.exists("chs_individual.csv")){
  download.file("https://raw.githubusercontent.com/USCbiostats/data-science-data/master/01_chs/chs_individual.csv", "chs_individual.csv", method="libcurl", timeout = 60)
}

individual <- data.table::fread("chs_individual.csv")

if(!file.exists("chs_regional.csv")){
  download.file("https://raw.githubusercontent.com/USCbiostats/data-science-data/master/01_chs/chs_regional.csv", "chs_regional.csv", method="libcurl", timeout = 60)
}

region <- data.table::fread("chs_regional.csv")
```

``` r
indi_reg <- merge(
  x=individual,
  y=region,
  by.x = "townname",
  by.y = "townname"
  )
```

\#\#Step1:

``` r
nrow(indi_reg)
```

    ## [1] 1200

``` r
nrow(individual)
```

    ## [1] 1200

``` r
nrow(region)
```

    ## [1] 12

no duplication after merge

``` r
summary(is.na(indi_reg))
```

    ##   townname          sid             male            race        
    ##  Mode :logical   Mode :logical   Mode :logical   Mode :logical  
    ##  FALSE:1200      FALSE:1200      FALSE:1200      FALSE:1200     
    ##                                                                 
    ##   hispanic         agepft          height          weight       
    ##  Mode :logical   Mode :logical   Mode :logical   Mode :logical  
    ##  FALSE:1200      FALSE:1111      FALSE:1111      FALSE:1111     
    ##                  TRUE :89        TRUE :89        TRUE :89       
    ##     bmi            asthma        active_asthma   father_asthma  
    ##  Mode :logical   Mode :logical   Mode :logical   Mode :logical  
    ##  FALSE:1111      FALSE:1169      FALSE:1200      FALSE:1094     
    ##  TRUE :89        TRUE :31                        TRUE :106      
    ##  mother_asthma     wheeze         hayfever        allergy       
    ##  Mode :logical   Mode :logical   Mode :logical   Mode :logical  
    ##  FALSE:1144      FALSE:1129      FALSE:1082      FALSE:1137     
    ##  TRUE :56        TRUE :71        TRUE :118       TRUE :63       
    ##  educ_parent       smoke            pets          gasstove      
    ##  Mode :logical   Mode :logical   Mode :logical   Mode :logical  
    ##  FALSE:1136      FALSE:1160      FALSE:1200      FALSE:1167     
    ##  TRUE :64        TRUE :40                        TRUE :33       
    ##     fev             fvc             mmef         pm25_mass      
    ##  Mode :logical   Mode :logical   Mode :logical   Mode :logical  
    ##  FALSE:1105      FALSE:1103      FALSE:1094      FALSE:1200     
    ##  TRUE :95        TRUE :97        TRUE :106                      
    ##   pm25_so4        pm25_no3        pm25_nh4        pm25_oc       
    ##  Mode :logical   Mode :logical   Mode :logical   Mode :logical  
    ##  FALSE:1200      FALSE:1200      FALSE:1200      FALSE:1200     
    ##                                                                 
    ##   pm25_ec         pm25_om         pm10_oc         pm10_ec       
    ##  Mode :logical   Mode :logical   Mode :logical   Mode :logical  
    ##  FALSE:1200      FALSE:1200      FALSE:1200      FALSE:1200     
    ##                                                                 
    ##   pm10_tc          formic          acetic           hcl         
    ##  Mode :logical   Mode :logical   Mode :logical   Mode :logical  
    ##  FALSE:1200      FALSE:1200      FALSE:1200      FALSE:1200     
    ##                                                                 
    ##     hno3           o3_max          o3106           o3_24        
    ##  Mode :logical   Mode :logical   Mode :logical   Mode :logical  
    ##  FALSE:1200      FALSE:1200      FALSE:1200      FALSE:1200     
    ##                                                                 
    ##     no2             pm10          no_24hr         pm2_5_fr      
    ##  Mode :logical   Mode :logical   Mode :logical   Mode :logical  
    ##  FALSE:1200      FALSE:1200      FALSE:1100      FALSE:900      
    ##                                  TRUE :100       TRUE :300      
    ##    iacid           oacid         total_acids        lon         
    ##  Mode :logical   Mode :logical   Mode :logical   Mode :logical  
    ##  FALSE:1200      FALSE:1200      FALSE:1200      FALSE:1200     
    ##                                                                 
    ##     lat         
    ##  Mode :logical  
    ##  FALSE:1200     
    ## 

``` r
#creat a new dataset of all numeric column means
means_hispanicman <- indi_reg %>%
  filter(male == 1 & hispanic == 1) %>%
  select_if(is.numeric) %>%
  colMeans(na.rm = TRUE)
means_hispanicman<- as.data.frame(t(means_hispanicman))
#repleace the NA for each column
indi_reg %>%
  replace_na(list(
    agepft=means_hispanicman$agepft,
    height=means_hispanicman$height,
    weight=means_hispanicman$weight,
    bmi=means_hispanicman$bmi,
    asthma=means_hispanicman$asthma,
    active_asthma=means_hispanicman$active_asthma,
    father_asthma=means_hispanicman$father_asthma,
    mother_asthma=means_hispanicman$mother_asthma,
    wheeze=means_hispanicman$wheeze,
    hayfever=means_hispanicman$hayfever,
    allergy=means_hispanicman$allergy,
    educ_parent=means_hispanicman$educ_parent,
    somke=means_hispanicman$somke,
    gasstove=means_hispanicman$gasstove,
    fev=means_hispanicman$fev,
    fvc=means_hispanicman$fvc,
    mmef=means_hispanicman$mmef))
```

    ##       townname  sid male race hispanic    agepft   height   weight      bmi
    ##    1:   Alpine  835    0    W        0 10.099932 143.0000 69.00000 15.33749
    ##    2:   Alpine  838    0    O        1  9.486653 133.0000 62.00000 15.93183
    ##    3:   Alpine  839    0    M        1 10.053388 142.0000 86.00000 19.38649
    ##    4:   Alpine  840    0    W        0  9.965777 146.0000 78.00000 16.63283
    ##    5:   Alpine  841    1    W        1 10.548939 150.0000 78.00000 15.75758
    ##   ---                                                                      
    ## 1196:   Upland 1867    0    M        1  9.618070 140.0000 71.00000 16.46568
    ## 1197:   Upland 2031    1    W        0  9.798768 135.0000 83.00000 20.70084
    ## 1198:   Upland 2032    1    W        0  9.549624 137.0000 59.00000 14.28855
    ## 1199:   Upland 2033    0    M        0 10.121834 130.0000 67.00000 18.02044
    ## 1200:   Upland 2053    0    W        0  9.966942 138.5984 82.76707 19.41148
    ##       asthma active_asthma father_asthma mother_asthma wheeze hayfever allergy
    ##    1:      0             0             0             0      0        0       1
    ##    2:      0             0             0             0      0        0       0
    ##    3:      0             0             0             1      1        1       1
    ##    4:      0             0             0             0      0        0       0
    ##    5:      0             0             0             0      0        0       0
    ##   ---                                                                         
    ## 1196:      0             0             1             0      0        0       0
    ## 1197:      0             0             0             0      1        0       1
    ## 1198:      0             0             0             1      1        1       1
    ## 1199:      0             1             0             0      1        1       0
    ## 1200:      0             0             0             0      0        0       0
    ##       educ_parent smoke pets  gasstove      fev      fvc     mmef pm25_mass
    ##    1:    3.000000     0    1 0.0000000 2529.276 2826.316 3406.579      8.74
    ##    2:    4.000000    NA    1 0.0000000 1737.793 1963.545 2133.110      8.74
    ##    3:    3.000000     1    1 0.0000000 2121.711 2326.974 2835.197      8.74
    ##    4:    2.423868    NA    0 0.8156863 2466.791 2638.221 3466.464      8.74
    ##    5:    5.000000     0    1 0.0000000 2251.505 2594.649 2445.151      8.74
    ##   ---                                                                      
    ## 1196:    3.000000     0    1 0.0000000 1733.338 1993.040 2072.643     22.46
    ## 1197:    3.000000     0    1 1.0000000 2034.177 2505.535 1814.075     22.46
    ## 1198:    3.000000     0    1 1.0000000 2077.703 2275.338 2706.081     22.46
    ## 1199:    3.000000     0    1 1.0000000 1929.866 2122.148 2558.054     22.46
    ## 1200:    3.000000     0    1 0.0000000 2120.266 2443.876 2447.494     22.46
    ##       pm25_so4 pm25_no3 pm25_nh4 pm25_oc pm25_ec pm25_om pm10_oc pm10_ec
    ##    1:     1.73     1.59     0.88    2.54    0.48    3.04    3.25    0.49
    ##    2:     1.73     1.59     0.88    2.54    0.48    3.04    3.25    0.49
    ##    3:     1.73     1.59     0.88    2.54    0.48    3.04    3.25    0.49
    ##    4:     1.73     1.59     0.88    2.54    0.48    3.04    3.25    0.49
    ##    5:     1.73     1.59     0.88    2.54    0.48    3.04    3.25    0.49
    ##   ---                                                                   
    ## 1196:     2.65     7.75     2.96    6.49    1.19    7.79    8.32    1.22
    ## 1197:     2.65     7.75     2.96    6.49    1.19    7.79    8.32    1.22
    ## 1198:     2.65     7.75     2.96    6.49    1.19    7.79    8.32    1.22
    ## 1199:     2.65     7.75     2.96    6.49    1.19    7.79    8.32    1.22
    ## 1200:     2.65     7.75     2.96    6.49    1.19    7.79    8.32    1.22
    ##       pm10_tc formic acetic  hcl hno3 o3_max o3106 o3_24   no2  pm10 no_24hr
    ##    1:    3.75   1.03   2.49 0.41 1.98  65.82 55.05 41.23 12.18 24.73    2.48
    ##    2:    3.75   1.03   2.49 0.41 1.98  65.82 55.05 41.23 12.18 24.73    2.48
    ##    3:    3.75   1.03   2.49 0.41 1.98  65.82 55.05 41.23 12.18 24.73    2.48
    ##    4:    3.75   1.03   2.49 0.41 1.98  65.82 55.05 41.23 12.18 24.73    2.48
    ##    5:    3.75   1.03   2.49 0.41 1.98  65.82 55.05 41.23 12.18 24.73    2.48
    ##   ---                                                                       
    ## 1196:    9.54   2.67   4.73 0.46 4.03  63.83 46.50 22.20 37.97 40.80   18.48
    ## 1197:    9.54   2.67   4.73 0.46 4.03  63.83 46.50 22.20 37.97 40.80   18.48
    ## 1198:    9.54   2.67   4.73 0.46 4.03  63.83 46.50 22.20 37.97 40.80   18.48
    ## 1199:    9.54   2.67   4.73 0.46 4.03  63.83 46.50 22.20 37.97 40.80   18.48
    ## 1200:    9.54   2.67   4.73 0.46 4.03  63.83 46.50 22.20 37.97 40.80   18.48
    ##       pm2_5_fr iacid oacid total_acids       lon      lat
    ##    1:    10.28  2.39  3.52        5.50 -116.7664 32.83505
    ##    2:    10.28  2.39  3.52        5.50 -116.7664 32.83505
    ##    3:    10.28  2.39  3.52        5.50 -116.7664 32.83505
    ##    4:    10.28  2.39  3.52        5.50 -116.7664 32.83505
    ##    5:    10.28  2.39  3.52        5.50 -116.7664 32.83505
    ##   ---                                                    
    ## 1196:    27.73  4.49  7.40       11.43 -117.6484 34.09751
    ## 1197:    27.73  4.49  7.40       11.43 -117.6484 34.09751
    ## 1198:    27.73  4.49  7.40       11.43 -117.6484 34.09751
    ## 1199:    27.73  4.49  7.40       11.43 -117.6484 34.09751
    ## 1200:    27.73  4.49  7.40       11.43 -117.6484 34.09751
