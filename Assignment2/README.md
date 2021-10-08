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

## Step1: Merge data and replace NA

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
#check which column have NA value
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
indi_reg<- indi_reg %>%
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
    smoke=means_hispanicman$smoke,
    gasstove=means_hispanicman$gasstove,
    fev=means_hispanicman$fev,
    fvc=means_hispanicman$fvc,
    mmef=means_hispanicman$mmef))
```

## Step2: Create new variable and summarise

``` r
indi_reg<-indi_reg %>%
  mutate(obesity_level=
           case_when(bmi<14 ~"underweight",
                     bmi<=22 ~"normal",
                     bmi<=24~"overweight",
                     bmi>24~"obese"))
datasum <- indi_reg[,.(
  maxbmi = max(bmi),
  minbmi = min(bmi),
  count = .N
  ), by = obesity_level
]

knitr::kable(datasum)
```

| obesity\_level |   maxbmi |   minbmi | count |
| :------------- | -------: | -------: | ----: |
| normal         | 21.96387 | 14.00380 |   975 |
| overweight     | 23.99650 | 22.02353 |    87 |
| obese          | 41.26613 | 24.00647 |   103 |
| underweight    | 13.98601 | 11.29640 |    35 |

## Step3: create smoke\_gas\_exposure

``` r
#Not really sure about how to deal with those "NAs" which replace by the mean at step 2 but since they are not equal to 0 treat them as exposed. 
indi_reg[, smoke_gas_exposure := fifelse(smoke != 0 & gasstove == 0, "expose to smoke",
                                                   fifelse(smoke == 0 & gasstove != 0, "expose to gas",
                                                           fifelse(smoke == 0 & gasstove == 0, "no exposure","expose to both")))]
```

    ## Warning in `[.data.table`(indi_reg, , `:=`(smoke_gas_exposure, fifelse(smoke !
    ## = : Invalid .internal.selfref detected and fixed by taking a (shallow) copy
    ## of the data.table so that := can add this new column by reference. At an
    ## earlier point, this data.table has been copied by R (or was created manually
    ## using structure() or similar). Avoid names<- and attr<- which in R currently
    ## (and oddly) may copy the whole data.table. Use set* syntax instead to avoid
    ## copying: ?set, ?setnames and ?setattr. If this message doesn't help, please
    ## report your use case to the data.table issue tracker so the root cause can be
    ## fixed or this message improved.

## Step4: create summary table

``` r
# Not sure what asthma indicator means... Since it's a binary value, use mean to get asthma proportion for each variable
townsum<- indi_reg %>%
  group_by(townname) %>%
  mutate(fev=fev/60)%>%
  summarise(mean(fev),sd(fev),mean(asthma))
knitr::kable(townsum)
```

| townname      | mean(fev) |  sd(fev) | mean(asthma) |
| :------------ | --------: | -------: | -----------: |
| Alpine        |  34.84294 | 4.844276 |    0.1148047 |
| Atascadero    |  34.69030 | 5.391881 |    0.2532031 |
| Lake Elsinore |  34.11717 | 5.061896 |    0.1280078 |
| Lake Gregory  |  34.91352 | 5.310580 |    0.1516016 |
| Lancaster     |  33.61127 | 5.305750 |    0.1648047 |
| Lompoc        |  34.09351 | 5.852351 |    0.1148047 |
| Long Beach    |  33.22404 | 5.343944 |    0.1364062 |
| Mira Loma     |  33.23689 | 5.439632 |    0.1580078 |
| Riverside     |  33.30462 | 4.648922 |    0.1100000 |
| San Dimas     |  33.82758 | 5.316175 |    0.1716016 |
| Santa Maria   |  33.88836 | 5.211392 |    0.1348047 |
| Upland        |  33.91205 | 5.723363 |    0.1216016 |

``` r
sexsum<- indi_reg %>%
  rename("sex" = "male") %>%
  mutate(sex = if_else(sex == 1, "male", "female")) %>%
  group_by(sex) %>%
  mutate(fev=fev/60)%>%
  summarise(mean(fev),sd(fev),mean(asthma))
knitr::kable(sexsum)
```

| sex    | mean(fev) |  sd(fev) | mean(asthma) |
| :----- | --------: | -------: | -----------: |
| female |  32.89833 | 5.255702 |    0.1217085 |
| male   |  35.08177 | 5.125261 |    0.1724113 |

``` r
obssum<- indi_reg %>%
  group_by(obesity_level) %>%
  mutate(fev = fev/60) %>%
  summarise(mean(fev),sd(fev),mean(asthma))
knitr::kable(obssum)
```

| obesity\_level | mean(fev) |  sd(fev) | mean(asthma) |
| :------------- | --------: | -------: | -----------: |
| normal         |  33.49398 | 4.940887 |    0.1406811 |
| obese          |  37.79747 | 5.403976 |    0.2085482 |
| overweight     |  37.07203 | 5.290434 |    0.1646013 |
| underweight    |  28.31975 | 5.076308 |    0.0857143 |

``` r
expsum<- indi_reg %>%
  group_by(smoke_gas_exposure) %>%
  mutate(fev = fev/60) %>%
  summarise(mean(fev),sd(fev),mean(asthma))
knitr::kable(expsum)
```

| smoke\_gas\_exposure | mean(fev) |  sd(fev) | mean(asthma) |
| :------------------- | --------: | -------: | -----------: |
| expose to both       |  33.85903 | 5.121660 |    0.1458333 |
| expose to gas        |  33.83787 | 5.318528 |    0.1460865 |
| expose to smoke      |  34.68843 | 4.800238 |    0.1541540 |
| no exposure          |  34.40755 | 5.497146 |    0.1478534 |

### Looking at the data (EDA)

``` r
# Already check and wrangling data at steps above, start graphong now.
ggplot(indi_reg,mapping=aes(x=bmi,y=fev,color=townname)) +
  geom_point() +
  geom_smooth(method = "lm", se= FALSE, color = "black") +
  labs(title = "BMI vs fev", x = "BMI kg/m^2", y="fev")+
  facet_wrap(~townname, nrow=4)
```

    ## `geom_smooth()` using formula 'y ~ x'

![](README_files/figure-gfm/sctterplots%20with%20regression%20of%20BMI%20vs%20fev%20by%20townname-1.png)<!-- -->
This graph illustrates the scatter plot with linear regression for BMI
vs fev. Group by region (town) Generally speaking, regardless of
region(town), there is a trend that the larger BMI a person has, the
greater force expiratory volume is. But since majority data crowded at
10-20 interval, it would be a good idea if we could sample based on BMI
strata.

``` r
ggplot(indi_reg) +
  geom_histogram(mapping = aes(x= fev, fill=obesity_level)) +
  scale_fill_brewer(palette = "Spectral") +
  labs(title = "histogram of fev colored by BMI (obesiry_level)", x="fev")
```

    ## `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.

![](README_files/figure-gfm/histograms%20for%20BMI-1.png)<!-- --> This
histogram shows the frequency of fev grouped by BMI (obesity\_level).
According to this histogram, we could see that the mean and median for
underweight is at over 1500 while the median and mean for normal,
overweight and obese shift rightward. But it’s obvious that data size
for three groups other than normal group are quiet small, this trend is
not very convincing.

``` r
ggplot(indi_reg) +
  geom_histogram(mapping = aes(x= fev, fill=smoke_gas_exposure)) +
  scale_fill_brewer(palette = "Set3") +
  labs(title = "histogram of fev colored by smoke_gas_exposure", x="fev")
```

    ## `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.

![](README_files/figure-gfm/histogram%20for%20exposure-1.png)<!-- -->
This histogram indicates the frequency of fev grouped by smoke/gas
exposure. This histogram above show all four group mean and median (even
mode) are locate just at 2100ish. All four groups have similar fev
distribution. We could not tell apparent assciation from this histogram.

``` r
ggplot(indi_reg) +
  geom_bar(mapping = aes(x= obesity_level, fill=smoke_gas_exposure)) +
  scale_fill_brewer(palette = "Accent")
```

![](README_files/figure-gfm/barchart%20of%20BMI%20by%20smoke/gas%20exposure-1.png)<!-- -->

``` r
  labs(title = "barchart of BMI (obesity level) colored by smoke/gas exposure", x="BMI")
```

    ## $x
    ## [1] "BMI"
    ## 
    ## $title
    ## [1] "barchart of BMI (obesity level) colored by smoke/gas exposure"
    ## 
    ## attr(,"class")
    ## [1] "labels"

This bar graph show the frenquency of obesity\_level grouped by
smoke/gas exposure. Regardless of the obesity level, majority people
expose to gas, followed by no exposure, exposure to both and least
expose to smoke, in other words, the distribution of exposure is
basically same among these four group. And as we noticed before, most
people fall into normal group and least people fall into underweight.
