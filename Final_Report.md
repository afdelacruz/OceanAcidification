# Ocean Acidification Report
Landon Kleinbrodt and Andrew de la Cruz  
ESPM 157: Prof. Carl Boettiger   



#Introduction

In 1910, the world emitted less than 5 billion metrics tons of carbon dioxide into the atmosphere; 100 years later that number has skyrocketed by more than 800%. Anthropogenic green house gases have been linked to countless examples of climate change. The ocean, being the largest ecosystem on the planet, has not escaped these negative effects. In this report, we will analyze three particularly worrisome properties: temperature, dissolved organic carbon, and acidity. First, we will visualize how these attributes have changed over time and how widespread the damage has become. Next, we will explore the negative consequences of these climate changes; specifically the damage done to coral reefs. These reefs foster more biodiversity than any other shallow-water ecosystem despite only covering .1% of the ocean floor. The chemical connection between temperature, acidity, and the health of marine life has been established by numerous scientific studies (See NOAA's Acidification page for more info). This report will model and visualize the drastic changes in ocean chemistry, as well as the health of coral reefs, to display the rapid changes in climate our planet is currently experiencing and their harmful effects.

###Data 

The data sets used in this study were provided by  [NOAA/OCADS](https://www.nodc.noaa.gov/archive/arc0107/0162565/1.1/data/0-data/), [ReefGIS](http://www.reefbase.org/gis_maps/datasets.aspx) and  [NOAA/OAR/ESRL](https://www.esrl.noaa.gov/psd/data/gridded/data.cobe2.html).
NOA/OCADS provides daily ocean acidity and units of dissolved organic carbon from 1991 to 2012. The data generated by Reefbase gives us an annual report from 1973 to 2012 of various coral reefs around the world, most importantly the severity of bleaching they are experiencing. The NOAA data reports monthly sea surface temperature means from 1850 to 2017 for the entire globe on a 1 degree latitude x 1 degree longitude grid.  

Also, many custom functions were created during this process to streamline analysis. The package repository is located [here](https://github.com/lkleinbrodt/AcidificationTools)

##Temperature

Much attention in climate change is given to the phenomenon known as Global Warming. In fact, the global temperature has increased by about 1 degree Celsius since 1880, and the rate at which this warming occurs is increasing at an alarming rate. The negative effects of this warming are widespread and range from small perturbations in natural cycles to catastrophic changes to an ecosystem. 

The ocean accounts for the majority of the Earth's surface area, and is thus exposed to this warming. The ocean holds almost all of the water on Earth, is a major source of food, and contains a staggering amount of biodiversity. Thus, understanding how the ocean warms and how it responds is critical. Fortunately, there are several services that been tracking ocean temperature and other qualities for well over century.

We begin by loading in the data. Note: the data came originally in the form of a NetCDF database. The following code extracts the necessary values and arranges it into an r friendly, multi-dimensional array

###Loading Data

```r
download.file(url = 'ftp://ftp.cdc.noaa.gov/Datasets/COBE2/sst.mon.mean.nc',
              destfile = './Temp/sst.mon.mean.nc')

nc = nc_open(filename = './Temp/sst.mon.mean.nc')
lon <- ncvar_get(nc, "lon")
lat <- ncvar_get(nc, "lat")
time <- ncvar_get(nc, "time")
time = days(time) + lubridate::date('1891-1-1')
sst = ncvar_get(nc, varid = 'sst')

temperature.array = array(sst, dim = c(length(lon), length(lat), length(time)))

temperature.array[1:10,1:5,1]
```

```
##         [,1]   [,2]   [,3]   [,4]   [,5]
##  [1,] -1.712 -1.730 -1.742 -1.760 -1.760
##  [2,] -1.712 -1.730 -1.745 -1.760 -1.760
##  [3,] -1.712 -1.730 -1.747 -1.762 -1.762
##  [4,] -1.710 -1.732 -1.747 -1.765 -1.765
##  [5,] -1.710 -1.732 -1.750 -1.765 -1.772
##  [6,] -1.710 -1.735 -1.750 -1.770 -1.775
##  [7,] -1.712 -1.735 -1.750 -1.767 -1.777
##  [8,] -1.712 -1.735 -1.752 -1.767 -1.775
##  [9,] -1.712 -1.735 -1.747 -1.767 -1.777
## [10,] -1.712 -1.735 -1.750 -1.765 -1.777
```

This arrangement can be thought of as 2004 stacked matrices, where each matrix corresponds a specific month. The values of each matrix correspond to the mean SST at the location specified by that grid location (rows:longitude, columns = latitudel)

This results in a lot of stored values (almost 130 million elements), so we will begin by selecting specific decades for our initial analysis. To do this, we use the custom built functions `slice_data` and `array_to_df` which are a part of the custom `AcidificationTools` package. `slice_data` will take our initial master array and return to us only the data for a desired span of years. Then, `array_to_df` will take that reduced array and convert it into a dataframe by expanding out the third dimension (time) and mappinglattitude and longitude keys to their true values. 

We will first look at four decades: 1860's, 1900's, 1960's, and 2000's. For each decade we will find the mean temperature at each grid location (latitude/longitude intersection), and log those four dataframes into a list.

###Slicing Data

```r
start_date = min(time)

time_periods = list(c(1860,1870), c(1900,1910), c(1960,1970), c(2000, 2010))

temperature.data.list = list()

for (i in 1:length(time_periods)){
  period = time_periods[[i]]
  temperature.data.list[[i]] = slice_data(my.array = temperature.array, years = period, origin = year(start_date)) %>%
    array_to_df(lons = lon, lats = lat) %>%
    group_by(x,y) %>%
    summarise(avg_temp = mean(temperature, na.rm=T))
}
```

```
## Joining, by = "lonkey"
```

```
## Joining, by = "latkey"
```

```
## Joining, by = "lonkey"
```

```
## Joining, by = "latkey"
```

```
## Joining, by = "lonkey"
```

```
## Joining, by = "latkey"
```

```
## Joining, by = "lonkey"
```

```
## Joining, by = "latkey"
```

```r
head(temperature.data.list[[1]])
```

```
## # A tibble: 6 x 3
## # Groups:   x [1]
##       x     y  avg_temp
##   <dbl> <dbl>     <dbl>
## 1   0.5 -70.5 -1.848306
## 2   0.5 -69.5 -1.842645
## 3   0.5 -68.5 -1.355107
## 4   0.5 -67.5 -1.062455
## 5   0.5 -66.5 -0.892438
## 6   0.5 -65.5 -0.806157
```

Here we can see what the first decade's dataframe looks like, the column `x` represents longitude, while `y` represents latitude

Now we will visualize these heat maps by converting them to a raster and then plotting them.

###Plotting those decades

```r
temperature.raster.list = list()
for (i in 1:length(temperature.data.list)){
  data = temperature.data.list[[i]]
  temperature.raster.list[[i]] = rasterFromXYZ(data)
}

colors = rev(c('#b01111', '#b4451f', '#dd9f40', '#e7d87d', '#62a1db', '#5DADE2','#85C1E9','#AED6F1'))

for (i in 1:length(temperature.raster.list)){
  raster = temperature.raster.list[[i]]
  time = time_periods[[i]]
  plot(raster, breaks = c(-5,0, 5, 10,15, 20, 25, 30), col = colors, main = paste(time[1], time[2], sep = ' to '))
}
```

![](Final_Report_files/figure-html/unnamed-chunk-3-1.png)<!-- -->![](Final_Report_files/figure-html/unnamed-chunk-3-2.png)<!-- -->![](Final_Report_files/figure-html/unnamed-chunk-3-3.png)<!-- -->![](Final_Report_files/figure-html/unnamed-chunk-3-4.png)<!-- -->

This initial visualization makes it seem like the temperature of the ocean has not changed much over the decades. But this might be because we are aggregating our data, and since our graph must be able to plot the colder warmers of the arctic as well as the warm waters of the tropics, the scale of the plots may be hiding some trends.

So, first we will ask the basic question:

###Is the ocean heating up?

```r
avg_temps = data.frame(dates = time, avg_temperature = 0)
for(i in 1:dim(temperature.array)[3]){
  avg_temps[i,'avg_temperature'] = mean(temperature.array[,,i], na.rm = T)
}

ggplot(data = avg_temps, aes(x = dates, y = avg_temperature)) + geom_point(alpha = 1, aes(col = avg_temperature)) + geom_smooth(col = 'red') + scale_color_gradientn(colours = c('#2E86C1','#F1C40F','#FF5733'))
```

```
## `geom_smooth()` using method = 'gam' and formula 'y ~ s(x, bs = "cs")'
```

```
## Warning: Removed 2002 rows containing non-finite values (stat_smooth).
```

```
## Warning: Computation failed in `stat_smooth()`:
## x has insufficient unique values to support 10 knots: reduce k.
```

```
## Warning: Removed 2002 rows containing missing values (geom_point).
```

![](Final_Report_files/figure-html/unnamed-chunk-4-1.png)<!-- -->

So, while the global average SST fluctuates, it is certainly increasing since 1900 and in fact the rate of that increase is itself increasing. 

Now we will visualize that heating geographically. Using the same periods as before we will see how the ocean warmed between the 1860's and the 1900's, and compare that to how the ocean warmed between the 1960's and 2000's.

###Is the rate of heating increasing?

```r
early_difference = inner_join(temperature.data.list[[1]], temperature.data.list[[2]],
                              suffix = c('_first', '_second'), by = c('x', 'y')) %>%
  mutate(difference = avg_temp_second - avg_temp_first) %>%
  select(x, y, difference)

late_difference = inner_join(temperature.data.list[[3]], temperature.data.list[[4]],
                             suffix = c('_first', '_second'), by = c('x', 'y')) %>%
  mutate(difference = avg_temp_second - avg_temp_first) %>%
  select(x, y, difference)

early_difference.raster = rasterFromXYZ(early_difference)
late_difference.raster = rasterFromXYZ(late_difference)

colors = c('#85C1E9', '#ffdc07','#ff8d00','#ff0000','#ae0000')

colors = rev(c('#ae0000', '#ff0000','#ff8d00', '#ffdc07', '#fff596'))

plot(early_difference.raster, breaks = c(-1,-.5,0,.5,1,1.5), col = colors, xlab = 'Longitude', ylab = 'Latitude',
     main = 'Temperature Difference (1860s to 1900s)')
```

![](Final_Report_files/figure-html/unnamed-chunk-5-1.png)<!-- -->

```r
plot(late_difference.raster, breaks = c(-1,-.5,0,.5,1,1.5), col = colors, main = 'Temperature Difference (1960s to 2000s)', xlab = 'Longitude', ylab = 'Latitude')
```

![](Final_Report_files/figure-html/unnamed-chunk-5-2.png)<!-- -->

We can see from these plots that the warming between 1960-2000 was more intense in magnitude than the warming between 1860-1900 and also more widespread. This is considerably more helpful than our previous plots that simply contained averages rather than differences in averages, because comparing these two graphs allows us to see where the ocean's are warming most quickly, and also where warming is occuring where it previously wasn't. 

For example, the first plot shows us that waters of the west coasts of California and Southern America warmed during the second half of the 19th century. This remained true in the second half of the 20th century, but we see more severe warming, and the warming region has spread to include almost the entire Pacific Ocean.

So, we have shown that the ocean as a whole is at the highest average temperature in recorded history, the rate of that temperature increase is the highest ever recorded, and heating has spread to affect almost every part of the globe, far outpacing historical heating patterns.

We now turn our attention to a similar, yet distinct phenomenon: Ocean Acidification.

#Acidification

Ocean heating and acidification are often thought of as parallel but separate processes. Both are caused by GHG in the atmosphere (especially carbon dioxide), and while they affect the ocean differently, both can be extremely detrimental to marine life. 


###Loading Data

```r
raw.data = fread('http://download752.mediafire.com/i6ycyzqlgyeg/3y3qw1acc4s316t/GLODAPv2+Merged+Master+File.csv', na.strings = '-9999', data.table = F)
```

```
## 
Read 22.0% of 999448 rows
Read 40.0% of 999448 rows
Read 63.0% of 999448 rows
Read 84.0% of 999448 rows
Read 999448 rows and 101 (of 101) columns from 0.410 GB file in 00:00:06
```

```r
master.df = raw.data %>%
  dplyr::select(year, latitude, longitude,  phts25p0, doc)

colnames(master.df) = c("year", "latitude", "longitude", "ph", "doc")
Acid.df = master.df[complete.cases(master.df$doc), ]

coral = fread('http://download1072.mediafire.com/j3wd7xr5t2ng/0h8t920dciie6vg/CoralBleaching.csv', data.table = F)
coral = coral[coral[,"SEVERITY_CODE"] != -1,]
```

We eliminate `SEVERITY_CODE` equal to -1 because it signifies reefs that have no information about bleaching condition.  We will use `Acid.df` to visualize the relation between the oceans decreasing acidity and the levels of dissolved organic carbon present. `coral` will be used in order to analyze the conditions of coral reefs around the world. 


```r
AcidPlot = function(df, region){
  test = GetRegions(df,region)
  grapher = test[c(1,4,5)]
  solver = melt(grapher, id.vars = 'year', variable.name = 'measures')
  ggplot(data = solver, aes(x = year)) +
  ggtitle(region) +
  theme(plot.title = element_text(hjust = 0.5)) +
  geom_smooth(aes(y = value, colour = variable)) +
    facet_wrap(~variable, scale = "free") 
}
```

`AcidPlot` is function that takes in a data frame and region, and then displays a graph of both the phbalance of the region and the `doc`. We use the function `GetRegions` in our package in order to pull data frames of specific latitudinal zones such as the `Tropics` or the `Arctic`. `AcidPlot` also creates a tidytable where `pH` and `doc` are categorical variables of each numeric entry. This allows us to pass our table into `ggplot` where we are able to plot the measurements side-by-side, making it easier to visualize and perform analysis. 


```r
AcidPlot(Acid.df, "Arctic")
```

```
## `geom_smooth()` using method = 'gam' and formula 'y ~ s(x, bs = "cs")'
```

```
## Warning: Removed 2720 rows containing non-finite values (stat_smooth).
```

![](Final_Report_files/figure-html/unnamed-chunk-8-1.png)<!-- -->

```r
AcidPlot(Acid.df, "Temperate")
```

```
## `geom_smooth()` using method = 'gam' and formula 'y ~ s(x, bs = "cs")'
```

```
## Warning: Removed 2660 rows containing non-finite values (stat_smooth).
```

![](Final_Report_files/figure-html/unnamed-chunk-8-2.png)<!-- -->

```r
AcidPlot(Acid.df, "Tropics")
```

```
## `geom_smooth()` using method = 'gam' and formula 'y ~ s(x, bs = "cs")'
```

```
## Warning: Removed 1617 rows containing non-finite values (stat_smooth).
```

![](Final_Report_files/figure-html/unnamed-chunk-8-3.png)<!-- -->

```r
AcidPlot(Acid.df, "All")
```

```
## `geom_smooth()` using method = 'gam' and formula 'y ~ s(x, bs = "cs")'
```

```
## Warning: Removed 6997 rows containing non-finite values (stat_smooth).
```

![](Final_Report_files/figure-html/unnamed-chunk-8-4.png)<!-- -->

Here we observe the relation between doc and Ph balance  in the three different latitude zones over a 20 year period.  In all three graphs, we see that `doc` and `ph` have a direct correlation. Interestingly, there seems to be a bit of lag between changes in `doc` and changes in `ph`. A change in `doc` is matched by a change in `ph`, but the change does not occur until a few years later. 

Looking at the world as a whole, acidity is currently at the same level as in 1990, however it has only recently improved and a decrease in `ph` of almost 0.4 can be seen from 1995 to 2005.  The reason the `ph` in 1990 and the `ph` in 2012 are equal can be explained by the increase in `ph` of the temperate zones offsetting the decreases in the tropics and the arctic. Since every region is different we must examine each individually in order to get a better idea of what exactly is happening to ocean acidity.

We see that the overall `ph` levels in the Arctic decreased by approximately 0.1 in the last 20 years. This is change represents a 30% increase in acidity. There was a large increase in  `ph` from 1995 to 200[0, but since then, `ph`  has fallen by almost 0.2, which increases acidity by ~60%. 

In the Temperate zones, acidity has actually improved over the last 20 years along with an increased amount of `doc`. The graph is quite variable, but the recent trend upward is a good sign. 

Lastly, in the Tropical zones, acidity has dropped dramatically, from being basic with a `ph` of 8.0 in 1992 to an alarming 7.6 in 2002, a 150% increase in acidity. Similar to the temeperate zones, `ph` seems to be treniding upward which is a good sign. We just need to limit the amount of emissions. 

An intersting fact is that in all areas, there seems to be in a dramatic improvement in `p` levels starting at 2000, followed by a dramatic decline. One may assume that there was some event that caused this dramatic effect, and further investigation is needed to find the exact reason(s) why this occurred. 



```r
CoralPlot = function(df, region, year){
  test = GetRegions(df,region)
  want = test %>%
    filter(YEAR > year) 

x = as.factor(want$BLEACHING_SEVERITY)
BLEACHING_SEVERITY = factor(x, levels(x)[c(4,2,3,1)])
SEVERITY_CODE = want$SEVERITY_CODE
bleach = data.frame(BLEACHING_SEVERITY, SEVERITY_CODE )


ggplot(data = data.frame(bleach), aes(x = SEVERITY_CODE, fill = BLEACHING_SEVERITY)) +
  geom_bar(aes(y = (..count..)/sum(..count..))) +
  ggtitle(paste("Bleaching Severity in", region, sep = " ")) +
  theme(plot.title = element_text(hjust = 0.5)) +
  labs(y = "Proportion", x = "Severity") +
  scale_fill_manual("Legend", 
                    values = c("No Bleaching" = "lightyellow", "Low" = "gold",
                               "Medium" = "orange", "HIGH" = "red"))
}
```


`CoralPlot` is used to visualize the proportion of coral that is experiencing some form of bleaching.  Its arguments are a dataframe, a latitudinal zone, and a year to start compiling data from. Coral is essential to marine life and contains the most diverse ecosystem on the planet. Bleaching occurs when coral expel the algae (zooxanthellae) living in their tissues causing the coral to turn completely white. It is caused by high acidity in the waters they live in and also increased water temperature. This causes stress within the coral and subjects them to mortality.  There are 4 categories each reef can be placed in: `No bleaching`, `Low`, `Medium`, and `HIGH`. In a perfect world, we want coral to experience little to no bleaching, but due to the amount of global warming and increased acidification, this is definitely not the case. We observe the proportions below. 




```r
CoralPlot(coral, "Tropics", 1900)
```

![](Final_Report_files/figure-html/unnamed-chunk-10-1.png)<!-- -->

```r
CoralPlot(coral, "Temperate", 1900)
```

![](Final_Report_files/figure-html/unnamed-chunk-10-2.png)<!-- -->

```r
CoralPlot(coral, "All", 1900)
```

![](Final_Report_files/figure-html/unnamed-chunk-10-3.png)<!-- -->

This increase in ocean acidification and temperature is taking a toll on coral reefs.  Coral reefs are mainly concentrated around the tropical regions with some existing in temperate zones and none in the Arctic. Increased ocean acidity prevents coral from absorbing the nutrients they need to maintain their skeletons. Looking at the histogram above, we see that about 69% of coral in the tropics, 78% of coral in Temperate zones, and  71% of coral in the world are experiencing some sort of bleaching.  

The tropical ocean seems to only be getting more acidic and thus those reefs in the `Low` and `Medium` range will only get worse unless something is done to stop this acidification. It is absurd that the proportion of coral experiencing `High` bleaching is almost equal to the proportion experiencing no bleaching. 

Almost all of the coral in the Temperate zones is experiencing some form of bleaching, hopefully the increase in `ph` will be able to counteract the damage that already has been done.  Most of the reefs are experiencing `Low` levels of bleaching, so this may be able to be undone before it gets out of hand. 

The world as a whole must realize how important coral reefs are to our environment. Without them, ocean life would rapidly decline and this would transcend into our daily lives.  It is a good sign that `ph` levels are currently increasing, but we must continue to monitor our levels of emission in order for this trend to continue. 

##Tasks Accomplished

* use of 5 dplyr functions
   + filter
   + group_by
   + summarise
   + mutate
   + select
* Custom R functions
* Custom R package
* Regular Expressions
* Spatial Data + Visualization
* Manipulation of dates and strings
* Used netCDF data type



