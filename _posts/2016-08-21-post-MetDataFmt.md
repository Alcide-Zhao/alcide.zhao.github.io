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
A GeoTIFF  is a public domain metadata standard which has the **georeferencing information** embedded within the image file so there is no accompany .tfw file needed.  The georeferencing information is included by way of tif tags that contains spatial information about the image file such as `map projection, coordinate systems, ellipsoids, datums`.  

Open GeoTIFF with the GDAL in rasterio
<pre>	
>>> import rasterio
###  Open file
>>> dataset = rasterio.open('example.tif')

# Data shape
>>> dataset.width
7731
>>> dataset.height
7871

# Some dataset attributes expose the properties of all dataset bands via a tuple of values, one per band. To get a mapping of band indexes to variable data types, apply a dictionary comprehension to the zip() product of a dataset’s indexes and dtypes attributes.
>>> {i: dtype for i, dtype in zip(dataset.indexes, dataset.dtypes)}
{1: 'uint16'}

# The spatial position of the upper left corner
>>> dataset.transform * (0, 0)

# The lower right corner:
dataset.transform * (dataset.width, dataset.height)

# These coordinate values are relative to the origin of the dataset’s coordinate reference system (CRS).
>>> dataset.rcs   # reference coordinate system
CRS.from_epsg(32612)  # The upper left corner of the example dataset, (358485.0, 4265115.0), is 141.5 kilometers west of zone 12’s central meridian (111 degrees west) and 4265 kilometers north of the equator.

# Data from a raster band can be accessed by the band’s index number. 
>>> dataset.indexes
(1,)
>>> band1 = dataset.read(1
>>> band1
array([[0, 0, 0, ..., 0, 0, 0],
       [0, 0, 0, ..., 0, 0, 0],
       [0, 0, 0, ..., 0, 0, 0],
       ...,
       [0, 0, 0, ..., 0, 0, 0],
       [0, 0, 0, ..., 0, 0, 0],
       [0, 0, 0, ..., 0, 0, 0]], dtype=uint16

# Datasets have an index() method for getting the array indices corresponding to points in georeferenced space.
>>> x, y = (dataset.bounds.left + 100000, dataset.bounds.top - 50000)
>>> row, col = dataset.index(x, y)
>>> row, col
(1666, 3333)
>>> band1[row, col]
7566

### To save array along with georeferencing information to a new raster data file, call rasterio.open() with a path to the new file to be created, 'w' to specify writing mode, and several keyword arguments.
>>> new_dataset = rasterio.open(
...     '/tmp/new.tif',   # file_name
...     'w',              
...     driver='GTiff',   # desired format driver
...     height=Z.shape[0], 
...     width=Z.shape[1],
...     count=1,          # a count of the dataset bands
...     dtype=Z.dtype,
...     crs='+proj=latlong',
...     transform=transform,
... )
# To copy the grid to the opened dataset, call the new dataset’s write() method with the grid and target band number as arguments.
>>> new_dataset.write(Z, 1)
# sync data to disk and finish
>>> new_dataset.close()

 </pre>

## Shapefile






