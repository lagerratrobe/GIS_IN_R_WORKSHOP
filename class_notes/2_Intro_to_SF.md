# Working with Vector Data

### Introducing the `sf` package
There are several R packages that provide support for working with spatial features, but in this class, we're going to use `sf` to work with vector data.  `sf` is short for "Simple Features", which are described on Wikipedia as being...

> ...a set of standards that specify a common storage and access model of geographic features made of mostly two-dimensional geometries (point, line, polygon, multi-point, multi-line, etc.) used by geographic databases and geographic information systems. It is formalized by both the Open Geospatial Consortium (OGC) and the International Organization for Standardization (ISO).

`sf` provides a standardized way to encode spatial vector data. It binds to ‘GDAL’ for reading and writing data, to ‘GEOS’ for geometrical operations, and to ‘PROJ’ for projection conversions and datum transformations.  

### Why use `sf`?
From a practical standpoint, `sf` is convenient because it represents features as a `data.frame` (or `tibble`) with an added geometry column.  This is similar to how most databases implement spatial support, and in fact, the spatial operators in `sf` mostly match those found in systems like PostGIS and Snowflake (for example).  This means that people who are already familiar with spatial operators like `st_buffer`, `st_intersect` etc. from other platforms will find `sf` fairly easy to understand.  It also means that R users can treat spatial data like any other tabular data set and use tools from things like the Tidyverse.

_NOTE: One extremely convenient feature of having features stored in dataframes is that they can be stored as a single RDS file._

#### 1. Open a spatial data set obtained from a public agency as a shapefile.

```
#| echo: true
library(sf)

states <- st_read("../Data/cb_2018_us_state_20m.shp")

Show in New Window
Reading layer `cb_2018_us_state_20m' from data source 
  `/home/randre/Documents/Code/GIS_IN_R_WORKSHOP/Data/cb_2018_us_state_20m.shp' 
  using driver `ESRI Shapefile'
Simple feature collection with 52 features and 9 fields
Geometry type: MULTIPOLYGON
Dimension:     XY
Bounding box:  xmin: -179.1743 ymin: 17.91377 xmax: 179.7739 ymax: 71.35256
Geodetic CRS:  NAD83
```

#### 2. Reproject the data into someting more suitable for spatial analysis

```
#| echo: true

albers_states <- st_transform(states, crs = 5070)
head(albers_states, n=1)

Simple feature collection with 1 feature and 9 fields
Geometry type: MULTIPOLYGON
Dimension:     XY
Bounding box:  xmin: 1396621 ymin: 1838048 xmax: 1796585 ymax: 2037741
Projected CRS: NAD83 / Conus Albers
  STATEFP  STATENS    AFFGEOID GEOID STUSPS     NAME LSAD       ALAND     AWATER
1      24 01714934 0400000US24    24     MD Maryland   00 25151100280 6979966958
                        geometry
1 MULTIPOLYGON (((1722285 184...
```

#### 3. Obtain some spatial information about the data set
Print the area of Washington State in sq km

```
#| echo: true
#| warning: false
library(dplyr)
library(units)

albers_states |> 
  filter(STUSPS == "WA") |>
  st_area() |>
  set_units(km^2)
  
 178816.4 [km^2]
```

Wikipedia says that the Area of Washington state is between 172,587 - 184,827 km2, depending on whether we measure just land area, or total area which includes water.  Why the discrepancy?

#### 4. View the data
Do a simple plot with the geometry of Washington state.

```
#| echo: true
#| fig-height: 6
#| fig-cap: "Washington State, Albers projection"
albers_wa <- albers_states |> 
  filter(STUSPS == "WA")

plot(albers_wa$geometry, graticule = TRUE)
```

Clearly, the scale of this data is pretty coarse, so it's simplified out a fair amount of land area.  (Something that's good to remember when working with spatial data - spatial resolution matters. )
