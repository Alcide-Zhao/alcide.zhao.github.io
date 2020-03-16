---
title: 'Commonly used meterology data format'
date: 2016-08-21
permalink: /posts/2016/08/post-MetDataFmt/
tags:
  - meterology
  - File
  - data
  - GRIB
  - netcdf
  - BUFR
  - GeoTIFF
---

Some commonly used filr format for oceanic and atmospheric sciences: definition, manipulation, and analysis

## GRIB

GRIB (GRIdded Binary or General Regularly-distributed Information in Binary form) is a concise data format commonly used in meteorology.  `pygrb` is a Python package to read and write GRIB
<pre>
import pygrib

# open GRIB file, create a grib message literator

>>> grbs = pygrib.open('sampledata/flux.grb');

# print inventory

>>> for g in grbs:
...   print g
1:Precipitation rate:kg m**-2 s**-1 (avg):regular_gg:surface:level 0:fcst time 108-120 hrs (avg):from 200402291200
2:Surface pressure:Pa (instant):regular_gg:surface:level 0:fcst time 120 hrs:from 200402291200
3:Maximum temperature:K (instant):regular_gg:heightAboveGround:level 2 m:fcst time 108-120 hrs:from 200402291200
4:Minimum temperature:K (instant):regular_gg:heightAboveGround:level 2 m:fcst time 108-120 hrs:from 200402291200

# ind the first grib message with a matching name: 

>>> grb = grbs.select(name='Maximum temperature')[0]

# extract the data values using the 'values' key (grb.keys() will return a list of the available keys). The data is returned as a numpy array

>>> maxt = grb.values
>>> maxt.shape, maxt.min(), maxt.max()
(94, 192) 223.7 319.9

# get the latitudes and longitudes of the grid

>>> lats, lons = grb.latlons()
>>> lats.shape, lats.min(), 
lats.max(), lons.shape, lons.min(), lons.max()
(94, 192) -88.5419501373 88.5419501373  0.0 358.125

# get the second grib message

>>> grb = grbs.message(2) # same as grbs.seek(1); grb=grbs.readline()
>>> grb
2:Surface pressure:Pa (instant):regular_gg:surface:level 0:fcst time 120 hrs:from 200402291200

# extract data and get lat/lon values for a subset over North America:

>>> data, lats, lons = grb.data(lat1=20,lat2=70,lon1=220,lon2=320)
>>> data.shape, lats.min(), lats.max(), lons.min(), lons.max()
(26, 53) 21.904439458 69.5216630593 221.25 318.75

# modify the values associated with existing keys (either via attribute or dictionary access):

>>> grb['forecastTime'] = 240
>>> grb.dataDate = 20100101

# get the binary string associated with the coded message:

>>> msg = grb.tostring()
>>> grbs.close() # close the grib file.

# write the modified message to a new GRIB file:

>>> grbout = open('test.grb','wb')
>>> grbout.write(msg)
>>> grbout.close()
>>> pygrib.open('test.grb').readline() 
1:Surface pressure:Pa (instant):regular_gg:surface:level 0:fcst time 240 hrs:from 201001011200
</pre>


## BUFR

The Binary Universal Form for the Representation of meteorological data (BUFR) is a binary data format maintained by the World Meteorological Organization (WMO). 


## GeoTIFF

## NetCDF
