Lab 03
================

``` r
#Data set

acs5_2015 <- getCensus(name = "acs5", vintage = 2015, key = cenesuskey, vars = c("NAME", "B19013_001E", "B01002A_001E", "B19025_001E"), region = "tract:*", regionin = "state:36")
syr.census <- acs5_2015[ acs5_2015$county == "067" , ]
syr.census <- mutate( syr.census, tract.num = as.numeric(tract)/100)

syr.shape.census <- merge(syr, syr.census, by.x="CensusTrac", by.y="tract.num")
```

``` r
#Map of median household income--red is low, blue is high

color.function <- colorRampPalette( c("firebrick4","light gray", "steel blue" ) )
col.ramp <- color.function( 5 )

#color.vector <- cut( rank(syr.shape.census$B19013_001E), breaks=5 , labels=col.ramp ) #median household income
color.vector <- cut((syr.shape.census$B19013_001E), breaks=c(0, 20000, 35000, 50000, 65000, 100000), labels = col.ramp )
color.vector <- as.character( color.vector )

plot(syr.shape.census, border=FALSE, col=color.vector)


title( main="Median Household Income")

#map.scale( metric=F, ratio=F, relwidth = 0.15, cex=0.5 )

legend.text=c("less than $20,000", "$20,000 - $35,000", "$35,000 - $50,000", "$50,000 - $65,000", "more than $65,000")

legend( "bottomright", bg="white",
        pch=19, pt.cex=1.5, cex=0.7,
        legend=legend.text, 
        col=col.ramp, 
        box.col="white",
        title="Median Household Income" 
)
```

![](Lab03__files/figure-markdown_github/unnamed-chunk-2-1.png)

``` r
#Map of aggregate family income--red is low, blue is red

color.function.3 <- colorRampPalette( c("firebrick4","light gray", "steel blue" ) )
col.ramp.3 <- color.function.3( 4 )

entries <- length(na.omit(syr.shape.census$B19025_001E))
color.vector.3 <- cut( syr.shape.census$B19025_001E, breaks=c(0, sort(syr.shape.census$B19025_001E)[entries/4], sort(syr.shape.census$B19025_001E)[entries/2], sort(syr.shape.census$B19025_001E)[(3*entries)/4], sort(syr.shape.census$B19025_001E)[entries]), labels = col.ramp.3) #map of aggregate family income
color.vector.3 <- as.character( color.vector.3 )

plot(syr.shape.census, border=FALSE, col=color.vector.3)

title( main="Aggregate Household Income")

#map.scale( metric=F, ratio=F, relwidth = 0.15, cex=0.5 )

legend.text=c("lowest 25th percentile", "25th - 50th percentile", "50th - 75th percentile", "75th percentile and above")

legend( "bottomright", bg="white",
        pch=19, pt.cex=1.5, cex=0.7,
        legend=legend.text, 
        col=col.ramp, 
        box.col="white",
        title="Aggregate Household Income" 
)
```

![](Lab03__files/figure-markdown_github/unnamed-chunk-3-1.png)

``` r
#Map of age--red is high, blue is low 

color.function.2 <- colorRampPalette( c("steel blue","light gray","firebrick4") )
col.ramp.2 <- color.function.2( 5 )

color.vector.2 <- cut( syr.shape.census$B01002A_001E, breaks=c(0, 29.9, 39.9, 49.9, 59.9, 100), labels=col.ramp.2 ) #age
color.vector.2 <- as.character( color.vector.2 )

plot(syr.shape.census, border=FALSE, col=color.vector.2)



title( main="Median Age")

#map.scale( metric=F, ratio=F, relwidth = 0.15, cex=0.5 )

legend.text=c("over 60", "50 to 60", "40 to 50", "30 to 40", "under 30")

legend( "bottomright", bg="white",
        pch=19, pt.cex=1.5, cex=0.7,
        legend=legend.text, 
        col=col.ramp, 
        box.col="white",
        title="Median Age" 
)
```

![](Lab03__files/figure-markdown_github/unnamed-chunk-4-1.png)
