---
title: 'Cf-python and cf-plot quick notes'
date: 2020-07-15
permalink: /posts/2020/07/post-cf-python/
tags:
  - python
  - plot
  - climate
  - netcdf
---

CF is a netCDF convention which is in wide and growing use for the storage of model-generated and observational data relating to the atmosphere, ocean and Earth system. it was developed by the UK NCAS. Cf-plot is a set of Python routines for making the common contour, vector and line plots that climate researchers use. It is currently also avaliable for Python 3, although there are compitable isses. [installation](https://cfpython.bitbucket.io/docs/latest/installation.html) [mannual](https://cfpython.bitbucket.io/docs/latest/index.html) [plotting](http://ajheaps.github.io/cf-plot/user_guide.html)

The following is a summary of their functions and features for quick reference

* <span style="font-size:1.6em;">[Cf-python](#block1)</span>
    * <span style="font-size:1.2em;">[Displaying contents](#chapter1)</span>
    * <span style="font-size:1.2em;">[Creating cf fields](#chapter2)</span>
    * <span style="font-size:1.2em;">[Subspacing](#chapter3)</span>
        * [Subspacing by axis indices](#chapter3.1)
        * [Subspacing by values of domain items (coordinates or cell measures)](#chapter3.2)
    * <span style="font-size:1.2em;">[Assignment](#chapter4)</span>
    * <span style="font-size:1.2em;">[Selection](#chapter5)</span>
    * <span style="font-size:1.2em;">[Collapse](#chapter6)</span>
    * <span style="font-size:1.2em;">[Read and Write](#chapter7) </span>
        * [Read](#chapter7.1) [write](#chapter7.2) 
    * <span style="font-size:1.2em;">[Aggregate](#chapter8)</span>
    * <span style="font-size:1.2em;">[logical operations](#chapter9)</span>
    * <span style="font-size:1.2em;">[Time and calendar](#chapter10)</span>
    * <span style="font-size:1.2em;">[Regrid](#chapter11)</span>
* <span style="font-size:1.6em;">[Cf-plot](#block2)</span>
    * <span style="font-size:1.2em;">[Opening graph, subplots and positioning](#chapter12)</span>
        * [Opening graph](#chapter12.1) [Positioning](#chapter12.2) [Closeing](#chapter12.3) 
    * <span style="font-size:1.2em;">[Set plotting variables and their defaults](#chapter13)</span>
        * [Axes](#chapter13.1) [Colorbar](#chapter13.2) [Display](#chapter13.3) [Title](#chapter13.4) [Text](#chapter13.5) [Land-ocean-lake](#chapter13.6) [Rotation](#chapter13.7) [Legend](#chapter13.8) [Grid](#chapter13.9) 
    * <span style="font-size:1.2em;">[Colormap and colorbar](#chapter14)</span>
        * [Colormap](#chapter14.1) [Color levels](#chapter14.2)     
    * <span style="font-size:1.2em;">[Axes limits and labels](#chapter15)</span>
        * [Set plot limits for all non longitude-latitide plots](#chapter15.1)
        * [Axes labelling](#chapter15.2)     
    * <span style="font-size:1.2em;">[Plot typew with cf-lpot](#chapter16)</span>
    * <span style="font-size:1.2em;">[How to ?](#chapter17) </span>  
        * [Have the same time units](#chapter17.1)   
        * [Reset time bounds](#chapter17.2) 
        * [Check what the data is in a dimenssion](#chapter17.3)         
        * [Check time dimenssion](#chapter17.4) 
        * [Area weighted mean](#chapter17.5) 
        * [Passing data via arrays](#chapter17.6) 
        * [Convert pressure to height in kilometers and vice-vers](#chapter17.7)     


# Cf-python<a class="anchor" id="block1"></a>
## 1. Displaying contents<a class="anchor" id="chapter1"></a>
1. Dump metadata and field attributes
>f.dump()
2. Display field's data (including units and data array)
>f.data
3. Data attributes
> f.shape
> f.array
4. CF properties
>f.standar_name'air_temperature'<br/>
f.setprop('standard_name', 'air_temperature')<br/>
f.getprop('standard_name')<br/>
'air_temperature'
f.delprop('standard_name')

## 2. Creating cf fields<a class="anchor" id="chapter2"></a>

<pre>
import numpy
import cf

#---------------------------------------------------------------------
# 1. CREATE the field's domain items
#---------------------------------------------------------------------
# Create a grid_latitude dimension coordinate
Y = cf.DimensionCoordinate(properties={'standard_name': 'grid_latitude'},
                           data=cf.Data(numpy.arange(10.), 'degrees'))

# Create a grid_longitude dimension coordinate
X = cf.DimensionCoordinate(data=cf.Data(numpy.arange(10.), 'degrees'))
X.standard_name = 'grid_longitude'

# Create a time dimension coordinate (with bounds)
bounds = cf.Bounds(data=cf.Data([0.5, 1.5],
                                cf.Units('days since 2000-1-1', calendar='noleap')))
T = cf.DimensionCoordinate(properties=dict(standard_name='time'),
                           data=cf.Data(1, cf.Units('days since 2000-1-1',
                                                    calendar='noleap')),
                           bounds=bounds)

# Create a longitude auxiliary coordinate
lat = cf.AuxiliaryCoordinate(data=cf.Data(numpy.arange(100).reshape(10, 10),
                                          'degrees_north'))
lat.standard_name = 'latitude'

# Create a latitude auxiliary coordinate
lon = cf.AuxiliaryCoordinate(properties=dict(standard_name='longitude'),
                             data=cf.Data(numpy.arange(1, 101).reshape(10, 10),
                                          'degrees_east'))

# Create a rotated_latitude_longitude grid mapping coordinate reference
grid_mapping = cf.CoordinateReference('rotated_latitude_longitude',
                                      parameters={
                                          'grid_north_pole_latitude': 38.0,
                                          'grid_north_pole_longitude': 190.0})

#---------------------------------------------------------------------
# 3. Create the field
#---------------------------------------------------------------------
# Create CF properties
properties = {'standard_name': 'eastward_wind',
              'long_name'    : 'Eastward Wind'}

# Create the field's data array
data = cf.Data(numpy.arange(100.).reshape(10, 10), 'm s-1')

# Finally, create the field
f = cf.Field(properties=properties)

f.insert_cell_methods('latitude: point')

f.insert_dim(T)
f.insert_dim(X)
f.insert_dim(Y)

f.insert_aux(lat, axes=['Y', 'X'])
f.insert_aux(lon, axes=['X', 'Y'])

f.insert_ref(grid_mapping)

f.insert_data(data, axes=['Y', 'X'])
</pre>


## 3. Subspacing<a class="anchor" id="chapter3"></a>
- Subspacing a field means subspacing its data array and its domain in a consistent manner.
- In index-space, a subspace is defined by specifying indices of the data array,
- In domain-space a subspace is defined as the part of the data array corresponds to given domain item values (e.g. particular coordinate values)

### 3.1 Subspacing by axis indices<a class="anchor" id="chapter3.1"></a>

> f[slice(0, 12), :, 10:0:-2]
> f[..., f.coord('longitude')<180]
> f[:, [0, 72], [5, 4, 3]]
    
### 3.2 Subspacing by values of domain items (coordinates or cell measures) <a class="anchor" id="chapter3.2"></a>
 - The axes to be subspaced may identified by name.<br/>
 - The position in the data array of each axis need not be known and the axes to be subspaced may be given in any order.<br/>
 - Axes for which no subspacing is required need not be specified.<br/>
 - Size 1 axes of the domain which are not spanned by the data array may be specified.
	 <pre>
	 >>> f.subspace().shape
	(12, 73, 96)
	>>> f.subspace(latitude=0).shape
	(12, 1, 96)
	>>> f.subspace(latitude=cf.wi(-30, 30)).shape
	(12, 25, 96)
	>>> f.subspace(long=cf.ge(270, 'degrees_east'), lat=cf.set([0, 2.5, 10])).shape
	(12, 3, 24)
	>>> f.subspace(latitude=cf.lt(0, 'degrees_north'))
	(12, 36, 96)
	>>> f.subspace(latitude=[cf.lt(0, 'degrees_north'), 90])
	(12, 37, 96)
	>>> import math
	>>> f.subspace('exact', longitude=cf.lt(math.pi, 'radian'), height=2)
	(12, 73, 48)
	>>> f.subspace(height=cf.gt(3))
	IndexError: No indices found for 'height' values gt 3
	>>> f.subspace(dim2=3.75).shape
	(12, 1, 96)
	>>> f.subspace(time=cf.le(cf.dt('1860-06-16 12:00:00')).shape
	(6, 73, 96)
	>>> f.subspace(time=cf.gt(cf.dt(1860, 7)),shape
	(5, 73, 96)
	 </pre>

## 4. Assignment<a class="anchor" id="chapter4"></a>

- Set all negative data array values to zero and leave all other elements unchanged:
> g = f.where(f<0, 0)
- Double the values in the northern hemisphere:
> index = f.indices(longitude=cf.ge(0))<br/>
f[index] *= 2

## 5. Selection<a class="anchor" id="chapter5"></a>
- Field selection 
> f.select('air_temperature', items={'latitude': cf.gt(0)}, rank=cf.ge(3))<br/>
f = cf.read('file*.nc', select='air_temperature', select_options={'rank': cf.gt(2)})
- For in-place changes, a for loop may be used to process each field element. For example to reverse the data array axis order of each field in-place:
> for if in f:   f.transpose(i=True)

## 6. Collapse<a class="anchor" id="chapter6"></a>
- Axes of a field may be collapsed by statistical methods with the cf.Field.collapse method.
- For example, to find the minumum of the data array: 
> g = f.collapse('min')
- By default the calculations of means, standard deviations and variances use a combination of volume, area and linear weights based on the field’s metadata. For example to find the mean of the data array, weighted where possible: 
> g = f.collapse('mean')
- Specific weights may be forced with the weights parameter. For example to find the variance of the data array, weighting the X and Y axes by cell area, the T axis linearly and leaving all other axes unweighted: 
> g = f.collapse('variance', weights=['area', 'T'])
-  Subset of the axes may be collapsed. For example, to find the mean over the time axis: 
> g = f.collapse('T: mean')
-  To find the maximum over the time and height axes:  
> g = f.collapse('max', axes=['T', 'Z'])
- An ordered sequence of collapses over different (or the same) subsets of the axes may be specified.  For example, to first find the mean over the time axis and subequently the standard deviation over the latitude and longitude axes:
> g = f.collapse('mean', axes='T').collapse('sd', axes='area')
- Climatological statistics (a type of grouped collapse) as defined by the CF conventions may be specified. For example, to collapse a time axis into multiannual means of calendar monthly minima:
> g = f.collapse('time: minimum within years T: mean over years',within_years=cf.M())

## 7. Read and Write<a class="anchor" id="chapter7"></a>

### 7.1 `cf.read` : Read fields from netCDF, PP or UM fields files.<a class="anchor" id="chapter7.1"></a>
 f = cf.read('file.nc')
 a. f field can be indexed
> cf.read('file*.nc')[0:2]  output the first two f fields
b. Use select to constrain utput. E.g, otputs where standard_name contains pmsl and then that units is either K or Pa
 > cf.read('file*.nc', select={'standard_name': '.*pmsl*', 'units':['K', 'Pa']})

### 7.2 `cf.write`: Write fields to a netCDF file.<a class="anchor" id="chapter7.1"></a>
> cf.write(fields=[f,g], file_name='new_file.nc',verwrite=False,compress=5)

    a. compress  0-9 balance between speed and size 1 is the fastest, but has the lowest compression ratio; 9 is the slowest but best compression ratio. <br/>
    b. datatype: By default, input data types are preserve (numpy.dtype('float64'),numpy.dtype('float32'))

## 8. Aggregate<a class="anchor" id="chapter8"></a>
- `cf.aggregate`: Aggregate fields into as few fields as possible.
- The aggregation of fields may be thought of as the combination fields into each other to create a new field that occupies a larger domain.
- [See here](https://cfpython.bitbucket.io/docs/latest/generated/cf.aggregate.html#cf.aggregate)


## 9. logical operations<a class="anchor" id="chapter9"></a>
- `cf.contain`  Return a cf.Query object for cells of a variable for containing a value.
- `cf.eq` 	Return a cf.Query object for a variable for being equal to a value.
- `cf.ge` 	Return a cf.Query object for a variable for being greater than or equal to a value.
- `cf.gt` 	Return a cf.Query object for a variable for being strictly greater than a value.
- `cf.le` 	Return a cf.Query object for a variable for being less than or equal to a value.
- `cf.lt` 	Return a cf.Query object for a variable for being strictly less than a value.
- `cf.ne` 	Return a cf.Query object for a variable for being not equal to a value.
- `cf.set` 	Return a cf.Query object for a variable for being equal to any member of a collection.
- `cf.wi` 	Return a cf.Query object for a variable being within a range.
- `cf.wo` 	Return a cf.Query object for a variable for being without a range.

## 10. Time and calendar related<a class="anchor" id="chapter10"></a>

- `cf.dt` 	Return a date-time variable for a given date and time.
> d = cf.dt(2003, 4, 5, 12, 30, 15)<br/>
d = cf.dt(year=2003, month=4, day=5, hour=12, minute=30, second=15)<br/>
d = cf.dt('2003-04-05 12:30:15')<br/>
print (d.year, d.month, d.day, d.hour, d.minute, d.second)<br/>
(2003, 4, 5, 12, 30, 15)

- `cf.TimeDuration(duration, units)`  A duration of time.  units can be {'calendar_years','calendar_months', 'months','days','hours','minutes','econds'}
> t = cf.TimeDuration(6, 'calendar_months')<br/>
print (t)<br/>
CF TimeDuration: 6 calendar_months (from Y-M-01 00:00:00)<br/>

- `interval` create time interval
> t.interval(1999, 12); print (t)<br/>
    (<CF Datetime: 1999-12-01 00:00:00>, <CF Datetime: 2000-06-01 00:00:00>)
> t = cf.TimeDuration(5, 'days', hour=6)<br/>
t.interval(2004, 3, 2, end=True)<br/>
     (datetime.datetime(2004, 2, 26, 6, 0), <CF Datetime: 2004-03-02 06:00:00>)<br/>
t.interval(2004, 3, 2, end=True, calendar='noleap')<br/>
    (<CF Datetime: 2004-02-25 06:00:00>, <CF Datetime: 2004-03-02 06:00:00>)<br/>
t.interval(2004, 3, 2, end=True, calendar='360_day')<br/>
    (<CF Datetime: 2004-02-27 06:00:00>, <CF Datetime: 2004-03-02 06:00:00>)<br/>
t.interval(2004, 3, 2, end=True, calendar='360_day', iso='start and duration')<br/>
    '2004-02-27 06:00:00/P5D'

- `cf.dteq` 	Return a cf.Query object for a variable being equal to a date-time.
> d = cf.dt(2002, 6, 16, 18, 30, 0)<br/>
d == cf.dteq(1990, 1, 1)<br/>
False<br/>
d == cf.wi(cf.dt(2001, 1, 1), cf.dt('2010-12-31'))<br/>
True

- `cf.dtge` Return a cf.Query object for a variable being not earlier than a date-time.
> d == cf.dtge(1990, 1, 1)<br/>
True

- similarly for `cf.dtle, cf.dtlt, cf.dtne`

- `cf.year, cf.month, cf.day, cd.hour,cf,minute, cf.second` Return a cf.Query object for date-time years.
> d == cf.year(cf.wi(2003, 2006))<br/>
False<br/>
d == cf.month(cf.le(7))<br/>
True<br/>
d == cf.day(cf.wi(1, 21))<br/>
True<br/>
d == cf.second(cf.wi(0, 30))<br/>
True

## 11. Regrid<a class="anchor" id="chapter11"></a>
- `cf.Field.regrids` Return the field regridded onto a new latitude-longitude grid.
> h = f.regrids(g, 'conservative/bilinear/nearest_dtos/nearest_stod')

- Regrid f to the grid of g using bilinear regridding and forcing the source field f to be treated as cyclic.
> h = f.regrids(g, src_cyclic=True, method='bilinear')
- Regrid f to the grid of g using the mask of g.
> h = f.regrids(g, 'conservative_1st', use_dst_mask=True)

- Regrid f to 2D auxiliary coordinates lat and lon, which have their dimensions ordered ‘Y’ first then ‘X’.
> h = f.regrids({'longitude': lon, 'latitude': lat, 'axes': ('Y', 'X')}, 'conservative')

- Regrid f to the grid of g iterating over the ‘Z’ axis last and the ‘T’ axis next to last to minimise the number of times the mask is changed.
> h = f.regrids(g, 'nearest_dtos', axis_order='ZT')





# Cf-plot<a class="anchor" id="block2"></a>
## 12. Opening graph, subplots and positioning<a class="anchor" id="chapter12"></a>
### cfp.gopen(rows=1, columns=1, user_plot=1, file='cfplot.png', orientation='landscape', figsize=[11.7, 8.3], left=None, right=None, top=None, bottom=None, wspace=None, hspace=None, dpi=None, user_position=False):<a class="anchor" id="chapter12.1"></a>
>file='cfplot.png',<br/>
orientation - landscape or portrait<br/>
figsize=[11.7, 8.3] - figure size in inches<br/>
left=None - left margin in normalised coordinates - default=0.12<br/>
right=None - right margin in normalised coordinates - default=0.92<br/>
top=None - top margin in normalised coordinates - default=0.08<br/>
bottom=None - bottom margin in normalised coordinates - default=0.08<br/>
wspace=None - width reserved for blank space between subplots - default=0.2<br/>
hspace=None - height reserved for white space between subplots - default=0.2<br/>
user_position=False - set to True to supply plot position via gpos xmin, xmax, ymin, ymax values

### cfp.gpos(pos=1, xmin=None, xmax=None, ymin=None, ymax=None): <a class="anchor" id="chapter12.2"></a>
>pos=pos - plot position<br/>
when user_position=True in cfp.gopen - set to True to supply plot position via gpos (xmin, xmax, ymin, ymax values)

### cfp.gclose(view=True): <a class="anchor" id="chapter12.3"></a>
> saves a graphics file. The default is to view the file as well

## 13.Set plotting variables and their defaults: Use setvars() to reset to the defaults<a class="anchor" id="chapter12"></a>
### cfp.setvars(file=None, title_fontsize=None, text_fontsize=None, colorbar_fontsize=None, colorbar_fontweight=None, axis_label_fontsize=None, title_fontweight=None, text_fontweight=None, axis_label_fontweight=None, fontweight=None, continent_thickness=None, continent_color=None, continent_linestyle=None, viewer=None, tspace_year=None, tspace_month=None, tspace_day=None, tspace_hour=None, xtick_label_rotation=None, xtick_label_align=None, ytick_label_rotation=None, ytick_label_align=None, legend_text_weight=None, legend_text_size=None, cs_uniform=None, master_title=None, master_title_location=None, master_title_fontsize=None, master_title_fontweight=None, dpi=None, land_color=None, ocean_color=None, lake_color=None, rotated_grid_spacing=None, rotated_deg_spacing=None, rotated_continents=None, rotated_grid=None, rotated_labels=None, rotated_grid_thickness=None, legend_frame=None, legend_frame_edge_color=None, legend_frame_face_color=None, degsym=None, axis_width=None, grid=None, grid_spacing=None, grid_colour=None, grid_linestyle=None, grid_thickness=None):
#### Axes <a class="anchor" id="chapter13.1"></a>
>axis_width=None - width of line for the axes<br/>
axis_label_fontsize=None - axis label fontsize, default=11<br/>
axis_label_fontweight='normal' - axis font weight<br/>
xtick_label_rotation=0 - rotation of xtick labels<br/>
xtick_label_align='center' - alignment of xtick labels<br/>
ytick_label_rotation=0 - rotation of ytick labels<br/>
ytick_label_align='right' - alignment of ytick labels  <br/>
tspace_year=None - time axis spacing in years<br/>
tspace_month=None - time axis spacing in months<br/>
tspace_day=None - time axis spacing in days<br/>
tspace_hour=None - time axis spacing in hours<br/>
degsym=True - add degree symbol to longitude and latitude axis labels

#### Colorbar <a class="anchor" id="chapter13.2"></a>
>colorbar_fontsize='11' - colorbar text size<br/>
colorbar_fontweight='normal' - colorbar font weight<br/>
cs_uniform=True - make a uniform differential colour scale <br/>

#### Display  <a class="anchor" id="chapter13.3"></a>
>viewer='display' - use ImageMagick display program<br/>
'matplotlib' to use image widget to view the picture

#### Title <a class="anchor" id="chapter13.4"></a>
>master_title=None - master title text<br/>
master_title_location=[0.5,0.95] - master title location<br/>
master_title_fontsize=30 - master title font size<br/>
master_title_fontweight='normal' - master title font weight<br/>
title_fontsize=None - title fontsize, default=15<br/>
title_fontweight='normal' - title fontweight

#### Text <a class="anchor" id="chapter13.5"></a>
>text_fontsize='normal' - text font size, default=11<br/>
text_fontweight='normal' - text font weight

#### Land, ocean and lake <a class="anchor" id="chapter13.6"></a>
>continent_thickness=1.5 - default=1.5<br/>
continent_color='k' - default='k' (black)<br/>
continent_linestyle='solid' - default='k' (black)<br/>
dpi=None - dots per inch setting<br/>
land_color=None - land colour<br/>
ocean_color=None - ocean colour<br/>
lake_color=None - lake colour

#### Rotation <a class="anchor" id="chapter13.7"></a>
>rotated_grid_spacing=10 - rotated grid spacing in degrees<br/>
rotated_deg_spacing=0.75 - rotated grid spacing between graticule dots<br/>
rotated_deg_tkickness=1.0 - rotated grid thickness for longitude and latitude lines<br/>
rotated_continents=True - draw rotated continents<br/>
rotated_grid=True - draw rotated grid<br/>
rotated_labels=True - draw rotated grid labels<br/>

#### Legend <a class="anchor" id="chapter13.8"></a>
>legend_text_size='11' - legend text size<br/>
legend_text_weight='normal' - legend text weight<br/>
legend_frame=True - draw a frame around a lineplot legend<br/>
legend_frame_edge_color='k' - color for the legend frame<br/>
legend_frame_face_color=None - color for the legend background <br/>   
legend_text_weight='normal' - legend text weight

#### Grid <a class="anchor" id="chapter13.9"></a>
>grid=True - draw grid<br/>
grid_spacing=1 - grid spacing in degrees<br/>
grid_colour='k' - grid colour<br/>
grid_linestyle='--' - grid line style<br/>
grid_thickness=1.0 - grid thickness
    
### cfp.reset(): reset all plotting variables

## 14. Colormap and colorbar<a class="anchor" id="chapter14"></a>
### customise colormap: a list of cf-plot [colormaps](http://ajheaps.github.io/cf-plot/colour_scales.html) <a class="anchor" id="chapter14.1"></a>
### cfp.cscale(cmap=None, ncols=None, white=None, below=None, above=None, reverse=False, uniform=False):
>cmap=None - name of colour map<br/>
ncols=None - number of colours for colour map<br/>
white=None - change these colours to be white: white=6<br/>
below=None - change the number of colours below the mid point of the colour scale to be this<br/>
above=None - change the number of colours above the mid point of the colour scale to be this<br/>
reverse=False - reverse the colour scale<br/>
uniform=False - produce a uniform colour scale.<br/>

###  Control # color levels: <a class="anchor" id="chapter14.2"></a>
### cfp.levs(min=None, max=None, step=None, manual=None, extend='both')
> min=min - minimum level<br/>
max=max - maximum level<br/>
step=step - step between levels<br/>
manual= manual - set levels manually: manual=[-10, -5, -4, -3, -2, -1, 1, 2, 3, 4, 5, 10]<br/>
extend='neither', 'both', 'min', or 'max'
- The levs command manually sets the contour levels.
- Once a user call is made to levs the levels are persistent.i.e. the next plot will use the same set of levels. Use levs() to reset to undefined levels.

### Customise colorbar <a class="anchor" id="chapter14.3"></a>
### cfp.cbar(labels=None, orientation=None, position=None, shrink=None, fraction=None, title=None, text_fontsize=None, text_fontweight=None, text_up_down=None, text_down_up=None, drawedges=None, levs=None, thick=None, anchor=None, extend=None, mid=None, verbose=None)
>labels - colorbar labels<br/>
orientation - orientation 'horizontal' or 'vertical'<br/>
position - colorbar position in normalised plot coordinates [left, bottom, width, height]<br/>
shrink - default=1.0 - scale colorbar along length<br/>
fraction - default = 0.21 - space for the colorbar in normalised plot coordinates<br/>
title - title for the colorbar<br/>
text_fontsize - font size for the colorbar text<br/>
text_fontweight - font weight for the colorbar text<br/>
text_up_down - label division text up and down starting with up<br/>
text_down_up - label division text down and up starting with down<br/>
drawedges - Draw internal delimeter lines in colorbar<br/>
levs - colorbar levels<br/>
thick - set height of colorbar - default = 0.015 in normalised plot coordinates<br/>
anchor - default=0.3 - anchor point of colorbar within the fraction space: 0.0 = close to plot, 1.0 = further away<br/>
extend = None - extensions for colorbar. The default is for extensions at both ends.<br/>
mid = False - label mid points of colours rather than the boundaries<br/>
verbose = None   

## 15. Axes: Controlling axes limit and labels<a class="anchor" id="chapter15"></a>
### Set plot limits for all non longitude-latitide plots:<a class="anchor" id="chapter15.1"></a>
### cfp.gset(xmin=None, xmax=None, ymin=None, ymax=None, xlog=False, ylog=False, user_gset=1, twinx=None, twiny=None):

>xmin=None - x minimum<br/>
xmax=None - x maximum<br/>
ymin=None - y minimum<br/>
ymax=None - y maximum<br/>
xlog=False - set to True to have log x<br/>
ylog=False - set to True to have log y<br/>
twinx=None - set to True to make a twin y axis plot<br/>
twiny=None - set to True to make a twin x axis plot<br/>
To set date axes use date strings i.e. cfp.gset(xmin = '1970-1-1', xmax = '1999-12-31', ymin = 285, ymax = 295);correct date format is 'YYYY-MM-DD' or 'YYYY-MM-DD HH:MM:SS'

### Customise axes labelling: <a class="anchor" id="chapter15.2"></a>
### cfp.axes(xticks=None, xticklabels=None, yticks=None, yticklabels=None, xstep=None, ystep=None, xlabel=None, ylabel=None, title=None)
> xstep=xstep - x axis step<br/>
    ystep=ystep - y axis step<br/>
    xlabel=xlabel - label for the x-axis<br/>
    ylabel=ylabel - label for the y-axis<br/>
    xticks=xticks - values for x ticks<br/>
    xticklabels=xticklabels - labels for x tick marks<br/>
    yticks=yticks - values for y ticks<br/>
    yticklabels=yticklabels - labels for y tick marks<br/>
    title=None - set title

## 16. Plot typew with cf-lpot<a class="anchor" id="chapter16"></a>
>#### [Contours](http://ajheaps.github.io/cf-plot/con.html): [demo](http://ajheaps.github.io/cf-plot/cylindrical.html), [latitude - pressure croess](http://ajheaps.github.io/cf-plot/pressure.html), [latitude-time](http://ajheaps.github.io/cf-plot/hovmuller.html)
>#### [Lines](http://ajheaps.github.io/cf-plot/lineplot.html): [demo](http://ajheaps.github.io/cf-plot/graphs.html) 
>#### [vectors](http://ajheaps.github.io/cf-plot/vect.html): [demo](http://ajheaps.github.io/cf-plot/vectors.html) 
>#### [trajectory](http://ajheaps.github.io/cf-plot/traj.html): [demo](http://ajheaps.github.io/cf-plot/trajectories.html+) 
>#### [Stipple](http://ajheaps.github.io/cf-plot/stipple.html): [demo](http://ajheaps.github.io/cf-plot//stipple_plots.html)
>#### [colormap](http://ajheaps.github.io/cf-plot/colour_scales.html)


## 17. How to ? <a class="anchor" id="chapter17"></a>
### Have a common time units <a class="anchor" id="chapter17.1"></a>
When plotting data with different time units users need to move their data to using a common set of units as below.this is because when making a contour or line plot the axes are defined in terms of a linear scale of numbers. Having two different linear scales breaks the connection between the data.
>data1.construct('T').Units<br/>
<CF Units: hours since 1900-01-01 00:00:00 standard><br/>
data2.construct('T').Units<br/>
<CF Units: days since 2008-09-01 00:00:00 standard><br/>
`data1.construct('T').Units=data2.construct('T').Units` <br/>
data1.construct('T').Units<br/>
<CF Units: days since 2008-09-01 00:00:00 standard><br/>
### Reset time bounds <a class="anchor" id="chapter17.2"></a>
>T=f.coord('T')<br/>
T.del_bounds()<br/>
new_bounds=T.create_bounds()<br/>
T.set_bounds(new_bounds)

### Check what the data is in a dimenssion <a class="anchor" id="chapter17"></a>
>g = cf.read('cfplot_data/tas_A1.nc')[0]<br/>
g.construct('longitude').array or<br/>
g.construct('X').array or<br/>
g.construct('long_name=longitude').array

### Check time dimenssion <a class="anchor" id="chapter17.3"></a>
- All data
> g.construct('T').dtarray<br/>
g.item('T').dtarray
- Bounds
>g.item('T').bounds.dtarray<br/>

 
### Select dimenssions <a class="anchor" id="chapter17.4"></a>
- One dimenssion
>g.subspace(longitude=0.0, latitude=0.0)<br/>
g.subspace(time=cf.dt('1860-1-16'))

-  range of dimension values
> g.subspace(longitude=cf.wi(0, 60))<br/>
g.subspace(X=cf.wi(0, 60))<br/>
g.subspace(T=cf.wi(cf.dt('1860-1-16'), cf.dt('1960-1-16')))

### Area weighted mean <a class="anchor" id="chapter17.5"></a>
> rea_mean = g.collapse('area: mean', weights='area')<br/>
area_mean
<CF Field: air_temperature(time(1680), latitude(1), longitude(1)) K>

### Passing data via arrays <a class="anchor" id="chapter17.6"></a>
>import cfplot as cfp<br/>
from netCDF4 import Dataset as ncfile<br/>
nc = ncfile('cfplot_data/tas_A1.nc')<br/>
lons=nc.variables['lon'][:]<br/>
lats=nc.variables['lat'][:]<br/>
temp=nc.variables['tas'][0,:,:]<br/>
cfp.con(f=temp, x=lons, y=lats,ptype=1) ! ptype -> 1: lon-lat, 2:lat-height cross, 3:lon-height cross, 4:lon-time cross, 5:lon-time cross, 7:time-height cross

### Change field metadata <a class="anchor" id="chapter17.7"></a>
> f.set_property('standard_name', 'new_name')

### Convert pressure to height in kilometers and vice-versa using the equation P=P0exp(-z/H) <a class="anchor" id="chapter17.8"></a>
>cfp.pcon(mb=None, km=None, h=7.0, p0=1000)<br/>
mb=None - input pressure<br/>
km=None - input height<br/>
h=7.0 - default value for h The value of H can vary from 6.0 in the polar regions to 8.5 in the tropics as well as seasonally. <br/>
p0=1000 - default value for p0
