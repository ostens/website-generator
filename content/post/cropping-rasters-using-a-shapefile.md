+++
title = "Cropping Rasters Using a Shapefile"
date = 2018-04-23T20:13:40+01:00
categories = [ "Geospatial" ]
tags = [ "gdalUtils", "R", "Raster", "Shapefile", "rgdal", "gdalwarp" ]
+++

At work I'm regularly delivered a batch of third-party geotiffs. They arrive in a localised projection and need to be converted into EPSG3857. Most annoyingly, some of the third-party providers don't provide inset plans separately, so each time we're sent updates, the inset plans need to cropped from the main charts and georeferenced.


To attempt to automate the process, I set up shapefiles for each inset plan, with a "chart" attribute corresponding to the main chart they would be derived from, and a "plan" attribute which they would eventually be named. 

I was hoping to write a batch script that used a gdal utility tool to run through each feature of the shapefile, find the corresponding geoTiff, crop it using the shapefile limits, and finally save the finished plan with the correct name.

It proved to be too advanced to be a batch script and I wrote in in R instead, which I'm less familiar with. 

I started out with the basic batch script, which was only able to crop one feature. This was run through the OSGeo4W shell:
```bash
echo off
set "plan=BSH1120-1-2018-02.tif"
gdalwarp -cutline "cutlines.shp" -csql "select * from cutlines where location='%plan%'" -crop_to_cutline -of GTiff -srcnodata -9999 -dstnodata -9999 BSH1120-0-2018-02.tif %plan%

```
When adapting this to a script in R using the gdalUtils package, the first thing to note is how the arguments of gdalwarp change format. While the shell gdalwarp's arguments are implied by whitespace, the gdalwarp in R's gdalUtils package takes a bracketed argument. The arguments also need to be comma-separated and explicit rather than implied (I think the order matters less as a result).\\
OSGeo4W shell:
```bash
gdalwarp -cutline "cutlines.shp" -csql "select * from cutlines where location='%plan%'" -crop_to_cutline -of GTiff -srcnodata -9999 -dstnodata -9999 %chart% %plan%

```
gdalUtils package in R:
```R
gdalwarp(srcfile=chart, dstfile=plan, srcnodata = "-9999", dstnodata = "-9999", of = "GTiff", csql = paste0("select * from cutlines where plan='",plan,"'"), cutline = "cutlines.shp", crop_to_cutline = TRUE, overwrite=TRUE, verbose=TRUE)
```
Next I wrote a simple For loop to iterate through each feature in the shapefile and crop the plan from the corresponding geoTif. The difficulty with this was that I wanted to use attribute strings from the shapefiles as arguments in the For loop, and it proved quite difficult to extract strings from a shapefile. I ended up extracting the strings from the corresponding .dbf file (this is a database file which is created whenever you generate a shapefile). I could read in the table containing the attributes without having the associated geometry, and extract the attributes that I needed as arguments.
```R
shape <- st_read('cutlines.shp')
database <- read.dbf('cutlines.dbf')
```
The For loop itself was very simple:
```R
for (i in 1:nrow(database)){
  chart <- sapply(database[i,"chart"], as.character)
  plan <- sapply(database[i,"plan"], as.character)
  gdalwarp...
}
```
* In the first line, I've just set the bounds for the loop, from 1 to the number of features in the shapefile (corresponding rows in the database file)
* In the second line, I've set the variable "chart" to pull its value from the ith row in the table, from the column entitled "chart"
* In the third line, I've done the same but pulled out the "plan" attribute from the shape database

[See the finalised code on my github account](https://github.com/ostens/gdal-scripts)!