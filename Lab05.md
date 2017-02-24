Lab05
================

``` r
#Geocode house adresses
houses <- zill.dat[ , c("address","zip") ]


houses$address <- gsub( ",", "", houses$address )
houses$address <- gsub( "\\.", "", houses$address )

addresses <- paste( houses$address, "Syracuse, NY", houses$zip, sep=", " )

lat.long.test <- geocode(addresses[1:6])

lat.long <- read.csv( "https://raw.githubusercontent.com/lecy/hedonic-prices/master/Data/lat.long.csv")

#Add in lat and long, reorder lat and lon
zill.dat.2 <- cbind( zill.dat, lat.long )

order <- c("lon" , "lat")
zill.ordered <- zill.dat.2[ order ]

#Spatial Join Zillow data (geocoded) with shapefiles
zill.spatial <- SpatialPoints(zill.ordered,
                     proj4string=CRS("+proj=longlat +datum=WGS84"))

zill.over <- over(zill.spatial , onondoga)
zill.shape <- cbind(zill.dat.2, zill.over)
```

``` r
#Add census data

acs5_2015 <- getCensus(name = "acs5", vintage = 2015, key = cenesuskey, vars = c("NAME"), region = "tract:*", regionin = "state:36")
syr.census <- acs5_2015[ acs5_2015$county == "067" , ]
zill.shape.census <- merge( zill.shape, syr.census, by.x="TRACTCE10" , by.y= "tract", all.x=TRUE)
```

``` r
#add crime data
crime <- read.csv("https://raw.githubusercontent.com/lecy/hedonic-prices/master/Data/crime.lat.lon.csv")
crime.spatial <- SpatialPoints(crime,
                     proj4string=CRS("+proj=longlat +datum=WGS84"))
crime.over <- over(crime.spatial, onondoga)
crime.shape <- cbind(crime, crime.over)

#count crimes
crime.grpuped <- group_by(crime.shape , TRACTCE10)
crime.count <- as.data.frame(summarise(crime.grpuped , crimes.in.tract = n() ))

#merge crime count to data
zill.shape.census.crime <- merge(zill.shape.census , crime.count , by="TRACTCE10")

head(zill.shape.census.crime) %>% pander
```

<table>
<caption>Table continues below</caption>
<colgroup>
<col width="15%" />
<col width="23%" />
<col width="10%" />
<col width="6%" />
<col width="6%" />
<col width="8%" />
<col width="17%" />
<col width="12%" />
</colgroup>
<thead>
<tr class="header">
<th align="center">TRACTCE10</th>
<th align="center">timestamp</th>
<th align="center">price</th>
<th align="center">X1</th>
<th align="center">X2</th>
<th align="center">sqft</th>
<th align="center">your.name</th>
<th align="center">lot.size</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td align="center">000100</td>
<td align="center">1/20/2015 17:46:09</td>
<td align="center">87000</td>
<td align="center">NA</td>
<td align="center">NA</td>
<td align="center">2760</td>
<td align="center">Fonda Chronis</td>
<td align="center">4800</td>
</tr>
<tr class="even">
<td align="center">000100</td>
<td align="center">1/20/2015 23:46:47</td>
<td align="center">133000</td>
<td align="center">NA</td>
<td align="center">NA</td>
<td align="center">2232</td>
<td align="center">Fonda Chronis</td>
<td align="center">13068</td>
</tr>
<tr class="odd">
<td align="center">000100</td>
<td align="center">1/21/2015 0:01:20</td>
<td align="center">46500</td>
<td align="center">NA</td>
<td align="center">NA</td>
<td align="center">720</td>
<td align="center">Fonda Chronis</td>
<td align="center">6120</td>
</tr>
<tr class="even">
<td align="center">000200</td>
<td align="center">1/20/2015 23:18:12</td>
<td align="center">82900</td>
<td align="center">NA</td>
<td align="center">NA</td>
<td align="center">1480</td>
<td align="center">Ryan Reeves</td>
<td align="center">NA</td>
</tr>
<tr class="odd">
<td align="center">000200</td>
<td align="center">1/20/2015 23:14:21</td>
<td align="center">125000</td>
<td align="center">NA</td>
<td align="center">NA</td>
<td align="center">2794</td>
<td align="center">Ryan Reeves</td>
<td align="center">8276</td>
</tr>
<tr class="even">
<td align="center">000200</td>
<td align="center">1/20/2015 22:54:28</td>
<td align="center">34900</td>
<td align="center">NA</td>
<td align="center">NA</td>
<td align="center">1080</td>
<td align="center">Ryan Reeves</td>
<td align="center">2613</td>
</tr>
</tbody>
</table>

<table style="width:100%;">
<caption>Table continues below</caption>
<colgroup>
<col width="9%" />
<col width="9%" />
<col width="12%" />
<col width="9%" />
<col width="17%" />
<col width="12%" />
<col width="9%" />
<col width="9%" />
<col width="9%" />
</colgroup>
<thead>
<tr class="header">
<th align="center">beds</th>
<th align="center">bath</th>
<th align="center">garage</th>
<th align="center">year</th>
<th align="center">elementary</th>
<th align="center">middle</th>
<th align="center">high</th>
<th align="center">walk</th>
<th align="center">tax</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td align="center">6</td>
<td align="center">2</td>
<td align="center">No</td>
<td align="center">1910</td>
<td align="center">2</td>
<td align="center">1</td>
<td align="center">2</td>
<td align="center">45</td>
<td align="center">1489</td>
</tr>
<tr class="even">
<td align="center">5</td>
<td align="center">2</td>
<td align="center">No</td>
<td align="center">1930</td>
<td align="center">2</td>
<td align="center">1</td>
<td align="center">1</td>
<td align="center">52</td>
<td align="center">1451</td>
</tr>
<tr class="odd">
<td align="center">2</td>
<td align="center">1</td>
<td align="center">No</td>
<td align="center">1964</td>
<td align="center">2</td>
<td align="center">1</td>
<td align="center">2</td>
<td align="center">48</td>
<td align="center">650</td>
</tr>
<tr class="even">
<td align="center">2</td>
<td align="center">2</td>
<td align="center">Yes</td>
<td align="center">1986</td>
<td align="center">2</td>
<td align="center">1</td>
<td align="center">2</td>
<td align="center">61</td>
<td align="center">1799</td>
</tr>
<tr class="odd">
<td align="center">6</td>
<td align="center">2</td>
<td align="center">Yes</td>
<td align="center">1909</td>
<td align="center">2</td>
<td align="center">1</td>
<td align="center">1</td>
<td align="center">59</td>
<td align="center">1500</td>
</tr>
<tr class="even">
<td align="center">3</td>
<td align="center">1.5</td>
<td align="center">No</td>
<td align="center">1880</td>
<td align="center">2</td>
<td align="center">1</td>
<td align="center">1</td>
<td align="center">64</td>
<td align="center">1087</td>
</tr>
</tbody>
</table>

<table>
<caption>Table continues below</caption>
<colgroup>
<col width="13%" />
<col width="17%" />
<col width="16%" />
<col width="9%" />
<col width="9%" />
<col width="26%" />
<col width="6%" />
</colgroup>
<thead>
<tr class="header">
<th align="center">highway</th>
<th align="center">restaurant</th>
<th align="center">starbucks</th>
<th align="center">park</th>
<th align="center">mall</th>
<th align="center">address</th>
<th align="center">zip</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td align="center">Yes</td>
<td align="center">5</td>
<td align="center">5</td>
<td align="center">30</td>
<td align="center">2</td>
<td align="center">139 Pulaski St</td>
<td align="center">13204</td>
</tr>
<tr class="even">
<td align="center">Yes</td>
<td align="center">2</td>
<td align="center">1.5</td>
<td align="center">8</td>
<td align="center">1.3</td>
<td align="center">106 Giminski Drive</td>
<td align="center">13204</td>
</tr>
<tr class="odd">
<td align="center">Yes</td>
<td align="center">4</td>
<td align="center">1.4</td>
<td align="center">7</td>
<td align="center">1.3</td>
<td align="center">103 Pulaski St</td>
<td align="center">13204</td>
</tr>
<tr class="even">
<td align="center">No</td>
<td align="center">27</td>
<td align="center">1.2</td>
<td align="center">10</td>
<td align="center">1.2</td>
<td align="center">103 Arnts Place</td>
<td align="center">13208</td>
</tr>
<tr class="odd">
<td align="center">No</td>
<td align="center">25</td>
<td align="center">1.2</td>
<td align="center">10</td>
<td align="center">1.2</td>
<td align="center">805 Turtle Street</td>
<td align="center">13208</td>
</tr>
<tr class="even">
<td align="center">No</td>
<td align="center">30</td>
<td align="center">1.2</td>
<td align="center">8</td>
<td align="center">1.2</td>
<td align="center">619 2nd N Street</td>
<td align="center">13208</td>
</tr>
</tbody>
</table>

<table style="width:92%;">
<caption>Table continues below</caption>
<colgroup>
<col width="11%" />
<col width="9%" />
<col width="8%" />
<col width="16%" />
<col width="18%" />
<col width="16%" />
<col width="11%" />
</colgroup>
<thead>
<tr class="header">
<th align="center">tract</th>
<th align="center">lon</th>
<th align="center">lat</th>
<th align="center">STATEFP10</th>
<th align="center">COUNTYFP10</th>
<th align="center">GEOID10</th>
<th align="center">NAME10</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td align="center">73000</td>
<td align="center">-76.17</td>
<td align="center">43.06</td>
<td align="center">36</td>
<td align="center">067</td>
<td align="center">36067000100</td>
<td align="center">1</td>
</tr>
<tr class="even">
<td align="center">1</td>
<td align="center">-76.17</td>
<td align="center">43.06</td>
<td align="center">36</td>
<td align="center">067</td>
<td align="center">36067000100</td>
<td align="center">1</td>
</tr>
<tr class="odd">
<td align="center">1</td>
<td align="center">-76.17</td>
<td align="center">43.06</td>
<td align="center">36</td>
<td align="center">067</td>
<td align="center">36067000100</td>
<td align="center">1</td>
</tr>
<tr class="even">
<td align="center">2</td>
<td align="center">-76.16</td>
<td align="center">43.08</td>
<td align="center">36</td>
<td align="center">067</td>
<td align="center">36067000200</td>
<td align="center">2</td>
</tr>
<tr class="odd">
<td align="center">2</td>
<td align="center">-76.16</td>
<td align="center">43.07</td>
<td align="center">36</td>
<td align="center">067</td>
<td align="center">36067000200</td>
<td align="center">2</td>
</tr>
<tr class="even">
<td align="center">2</td>
<td align="center">-76.16</td>
<td align="center">43.08</td>
<td align="center">36</td>
<td align="center">067</td>
<td align="center">36067000200</td>
<td align="center">2</td>
</tr>
</tbody>
</table>

<table style="width:99%;">
<caption>Table continues below</caption>
<colgroup>
<col width="20%" />
<col width="13%" />
<col width="18%" />
<col width="13%" />
<col width="15%" />
<col width="16%" />
</colgroup>
<thead>
<tr class="header">
<th align="center">NAMELSAD10</th>
<th align="center">MTFCC10</th>
<th align="center">FUNCSTAT10</th>
<th align="center">ALAND10</th>
<th align="center">AWATER10</th>
<th align="center">INTPTLAT10</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td align="center">Census Tract 1</td>
<td align="center">G5020</td>
<td align="center">S</td>
<td align="center">4842958</td>
<td align="center">1284980</td>
<td align="center">+43.0691355</td>
</tr>
<tr class="even">
<td align="center">Census Tract 1</td>
<td align="center">G5020</td>
<td align="center">S</td>
<td align="center">4842958</td>
<td align="center">1284980</td>
<td align="center">+43.0691355</td>
</tr>
<tr class="odd">
<td align="center">Census Tract 1</td>
<td align="center">G5020</td>
<td align="center">S</td>
<td align="center">4842958</td>
<td align="center">1284980</td>
<td align="center">+43.0691355</td>
</tr>
<tr class="even">
<td align="center">Census Tract 2</td>
<td align="center">G5020</td>
<td align="center">S</td>
<td align="center">1095299</td>
<td align="center">0</td>
<td align="center">+43.0747759</td>
</tr>
<tr class="odd">
<td align="center">Census Tract 2</td>
<td align="center">G5020</td>
<td align="center">S</td>
<td align="center">1095299</td>
<td align="center">0</td>
<td align="center">+43.0747759</td>
</tr>
<tr class="even">
<td align="center">Census Tract 2</td>
<td align="center">G5020</td>
<td align="center">S</td>
<td align="center">1095299</td>
<td align="center">0</td>
<td align="center">+43.0747759</td>
</tr>
</tbody>
</table>

<table>
<colgroup>
<col width="18%" />
<col width="34%" />
<col width="11%" />
<col width="12%" />
<col width="23%" />
</colgroup>
<thead>
<tr class="header">
<th align="center">INTPTLON10</th>
<th align="center">NAME</th>
<th align="center">state</th>
<th align="center">county</th>
<th align="center">crimes.in.tract</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td align="center">-076.1730170</td>
<td align="center">Census Tract 1, Onondaga County, New York</td>
<td align="center">36</td>
<td align="center">067</td>
<td align="center">45</td>
</tr>
<tr class="even">
<td align="center">-076.1730170</td>
<td align="center">Census Tract 1, Onondaga County, New York</td>
<td align="center">36</td>
<td align="center">067</td>
<td align="center">45</td>
</tr>
<tr class="odd">
<td align="center">-076.1730170</td>
<td align="center">Census Tract 1, Onondaga County, New York</td>
<td align="center">36</td>
<td align="center">067</td>
<td align="center">45</td>
</tr>
<tr class="even">
<td align="center">-076.1583997</td>
<td align="center">Census Tract 2, Onondaga County, New York</td>
<td align="center">36</td>
<td align="center">067</td>
<td align="center">11</td>
</tr>
<tr class="odd">
<td align="center">-076.1583997</td>
<td align="center">Census Tract 2, Onondaga County, New York</td>
<td align="center">36</td>
<td align="center">067</td>
<td align="center">11</td>
</tr>
<tr class="even">
<td align="center">-076.1583997</td>
<td align="center">Census Tract 2, Onondaga County, New York</td>
<td align="center">36</td>
<td align="center">067</td>
<td align="center">11</td>
</tr>
</tbody>
</table>
