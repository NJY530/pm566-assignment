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
  geom_jitter() +
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
distribution. We could not tell apparent association from this
histogram.

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

``` r
#I‘m not sure whether a boxplot is statistical summary but I think it's equivalent to stat_summary
ggplot(indi_reg) + 
    stat_summary(mapping = aes(x = obesity_level, y = fev),
    fun.min = min,
    fun.max = max,
    fun = median) +
  labs(title = "summary graph for fev vs BMI (obesity level)", x = "obesity_level", y ="fev")
```

![](README_files/figure-gfm/statistical%20summary%20graph%20of%20FEV%20by%20BMI-1.png)<!-- -->
This summary graph indicates the minimum, maximum and median for fev in
each BMI group. According to this graph, obesity group has the highest
fev followed by overweight, normal and underweight group.

``` r
ggplot(indi_reg) + 
    stat_summary(mapping = aes(x = smoke_gas_exposure, y = fev),
    fun.min = min,
    fun.max = max,
    fun = median) +
  labs(title = "summary graph for fev vs smoke/gas exposure", x = "exposure status", y ="fev")
```

![](README_files/figure-gfm/statistical%20summary%20graph%20of%20fev%20by%20exposu-1.png)<!-- -->
This summary graph indicates the minimun, maximun and median for fev in
each exposure group. According to this graph, the median and
distribution is similary for all four group. No obvious difference.

``` r
library(leaflet)

mapdata <- region %>%
  select(pm25_mass, lon, lat, townname)

pm25.pal <- colorNumeric(palette = "Reds", domain = mapdata$pm25_mass)


leaflet(mapdata) %>%
  addProviderTiles('CartoDB.Positron') %>%
  addCircles(
    lat = ~lat, lng = ~lon,
    label = ~paste0(round(pm25_mass,4), ' µg/m^−3', ' ', townname),
    color = ~pm25.pal(mapdata$pm25_mass),
    opacity = 1, fillOpacity = 1, radius = 500
    ) %>%
  addLegend('bottomleft', pal = pm25.pal, values = indi_reg$pm25_mass,
            title = 'PM2.5 mass', opacity = 1)
```

<div id="htmlwidget-4e0a456851d726a2b84a" style="width:672px;height:480px;" class="leaflet html-widget"></div>
<script type="application/json" data-for="htmlwidget-4e0a456851d726a2b84a">{"x":{"options":{"crs":{"crsClass":"L.CRS.EPSG3857","code":null,"proj4def":null,"projectedBounds":null,"options":{}}},"calls":[{"method":"addProviderTiles","args":["CartoDB.Positron",null,null,{"errorTileUrl":"","noWrap":false,"detectRetina":false}]},{"method":"addCircles","args":[[32.8350521,33.6680772,34.242901,34.6867846,34.6391501,33.7700504,33.9845417,33.9806005,34.1066756,35.4894169,34.9530337,34.09751],[-116.7664109,-117.3272615,-117.275233,-118.1541632,-120.4579409,-118.1937395,-117.5159449,-117.3754942,-117.8067257,-120.6707255,-120.4357191,-117.6483876],500,null,null,{"interactive":true,"className":"","stroke":true,"color":["#FEE2D4","#FCB69B","#FFE9DF","#FEE3D7","#FFF5F0","#F75A3E","#67000D","#DE2C25","#F14330","#FFEAE1","#FFECE4","#DD2B24"],"weight":5,"opacity":1,"fill":true,"fillColor":["#FEE2D4","#FCB69B","#FFE9DF","#FEE3D7","#FFF5F0","#F75A3E","#67000D","#DE2C25","#F14330","#FFEAE1","#FFECE4","#DD2B24"],"fillOpacity":1},null,null,["8.74 µg/m^−3 Alpine","12.35 µg/m^−3 Lake Elsinore","7.66 µg/m^−3 Lake Gregory","8.5 µg/m^−3 Lancaster","5.96 µg/m^−3 Lompoc","19.12 µg/m^−3 Long Beach","29.97 µg/m^−3 Mira Loma","22.39 µg/m^−3 Riverside","20.52 µg/m^−3 San Dimas","7.48 µg/m^−3 Atascadero","7.19 µg/m^−3 Santa Maria","22.46 µg/m^−3 Upland"],{"interactive":false,"permanent":false,"direction":"auto","opacity":1,"offset":[0,0],"textsize":"10px","textOnly":false,"className":"","sticky":true},null,null]},{"method":"addLegend","args":[{"colors":["#FFF5F0 , #FED3C1 16.826322365681%, #FC9272 37.6509787588505%, #F34C36 58.47563515202%, #BE151A 79.3002915451895%, #67000D "],"labels":["10","15","20","25"],"na_color":null,"na_label":"NA","opacity":1,"position":"bottomleft","type":"numeric","title":"PM2.5 mass","extra":{"p_1":0.16826322365681,"p_n":0.793002915451895},"layerId":null,"className":"info legend","group":null}]}],"limits":{"lat":[32.8350521,35.4894169],"lng":[-120.6707255,-116.7664109]}},"evals":[],"jsHooks":[]}</script>

This leaflet shows the PM2.5 concentration in each region. It shows that
the closer the town is to inland and cities (more roads shown on the
map), the worse the air quality (higher PM2.5 mass).

``` r
#Since pm25_mass is 12 discrete number in each community, we use histogram to fill
ggplot(indi_reg) +
  geom_point(mapping = aes(x = pm25_mass,y=fev,color = pm25_mass)) +
#  scale_color_brewer(palette="Dark2")+ this line reports error and I don't know why... seems like pm24_mass is a discrete scale but the palette is continous... it works on the example websit (ref: http://www.sthda.com/english/wiki/ggplot2-colors-how-to-change-colors-automatically-and-manually#change-colors-by-groups) I don't know what's the difference...
  
  labs(title = "fev distribution colored by pm2.5 mass", x= "pm2.5", y = "fev")
```

![](README_files/figure-gfm/a%20bar%20chart%20of%20fev%20vs%20pm25_mass-1.png)<!-- -->
This scatter plot indicates that the distribution of fev under each
pm2.5 level (actually each region since we only have 12 town and 12
observation). There is no obvious trend or association between fev and
pm2.5.

EDA report: According to the graph above, we can conclude for now that
1.the larger BMI a person have, the greater FEC(forced expiratory
volume) he would have. 2. there is no obvious association between smoke
and gas exposure and FEV 3. there is no obvious assocaition between
pm2.5 exposure and FEV.
