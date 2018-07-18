+++
title = "Command Line Reprojection"
date = 2018-06-23T18:15:53+01:00
categories = [ "Geospatial" ]
tags = [ "OSGeo4W" , "gdalwarp" , "gdal_translate" , "gdalmanage" ]
+++

A lot of the data manipulation I do is FME-based these days for ease of use, but for reprojecting rasters the gdal toolset is just superior to any other tool because it requires no foreknowledge of the original projection of the dataset. 

In many cases I'll have a set of rasters which are all in slightly different projections (with different scaling latitudes, for example), so being able to reproject each raster on-the-fly is really useful. 

In this case I was able to write a simple batch script that can be run via the OSGEO4W shell. 

The script reprojects all GeoTIFF (.tif) files in a source folder with undefined EPSG to a destination EPSG code, applying LZW compression and saving resultant GeTIFFs to a destination folder. 

[See the finalised code on my github account](https://github.com/ostens/gdal-scripts).


```bash
@echo off
setlocal enabledelayedexpansion

set oldpath=C:\
set newpath=C:\

set numFiles=0
for %%x in (%oldpath%*.tif) do (
  set file[!numFiles!]=%%~nfx
  set /a numFiles+=1
)

echo Number of files to reproject: %numFiles%

```
This first section sets the source and destination folders, then counts the number of GeoTIFF files in the source folder and reports this to the user.

```bash
set count=1

for /f %%i in ('dir /B %oldpath%*.tif') do (
	echo Starting !count! of %numFiles%	
	gdalwarp -t_srs EPSG:3857 -q "%oldpath%%%i" "%newpath%temp%%i"	
	if errorlevel 1 (
		echo The file %oldpath%%%i is not georeferenced and therefore cannot be reprojected
	) else (
		gdal_translate -q -co "COMPRESS=LZW" "%newpath%temp%%i" "%newpath%%%i"
		gdalmanage delete "%newpath%temp%%i"	
		echo Reprojected !count! of %numFiles%
	) 
	set /a count+=1
)
echo Finished.

```
This second section runs a for loop on each GeoTIFF which:

* Reports the file number to the user
* Uses gdalwarp to reproject from a source EPSG to EPSG:3857 (pseudo-mercator) with -q switch to suppress output. Saves the temporary reprojected file in the destination folder temp added to its filename. 
* Checks to see if this delivered an error; if it has, reports this to the user and skips the rest of the commands
* If it didn't deliver an error, uses gdal_translate to apply the creation option (-co) LZW compression to the GeoTIFF and saves it in the destination folder
* Clears up by deleting the intermediary temp file
