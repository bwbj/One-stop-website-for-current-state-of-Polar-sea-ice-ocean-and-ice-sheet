# proj_carto_PS_EG
this suite of programs aim to:
1.	Projecting sea ice parameter into Polar Stereographic and EASE-Grid.2.0 from latitude and longitude array of same size, then export NetCDF and GeoTIFF(optional) files. Scale original data if range is too short.
2.	Creating a map in Polar Stereographic projection, and set fix color-ramp upper limit to make same parameter comparable.

![](https://github.com/bwbj/proj_carto_PS_EG/blob/master/webpage%20framework.png "main")  
Function statement:
proj_PS_EG_main
main_body function to recall other utilities.
1.	Construct necessary input and output file and parameter augment.
2.	Recall margin_nan(nc_path, parameter) then return three array of lat / lon / parameter, convert lat and /lon to PS and EG coordinates via polarstereo_fwd() and latlon2ease_isprs().
3.	Recall regular_grid() to make equally-spaced grid of coordinates in PS and EG projection, based on which resampling original parameter by griddata() and ‘nearest’ strategy .
4.	Write output file of NetCDF and GeoTIFF(useless finally), delete first if file with same name already exits.
5.	Recall carto_png() to map the parameter in PS projection, then write the figure into PNG file.
proj_PS_EG_test
same function as proj_PS_EG_main(), but without input augment, works as another branch for convenience of debugging.
readme_test
traverse all NetCDF files in specified input_path directory and feed them into proj_PS_EG_main as input.
Projection Calculation
polarstereo_fwd() convert lat/lon to XY coordinates in Polar Stereographic projection, such as:
 [~, radius_thresh] = polarstereo_fwd(out_bound, proj_lon, major_axis, eccentricity, true_lat, proj_lon);
polarstereo_inv() convert XY coordinates in Polar Stereographic projection to lat/lon, such as:
[lat_cor, lon_cor] = polarstereo_inv([X_pad(end, 1), X_pad(end, end), X_pad(1, 1)],...
    [Y_pad(end, 1), Y_pad(end, end), Y_pad(1, 1)], major_axis, eccentricity, true_lat, proj_lon);
For above two function, please refer to: 
https://cn.mathworks.com/matlabcentral/fileexchange/32907-polar-stereographic-coordinate-transformation--map-to-lat-lon-
and 
https://cn.mathworks.com/matlabcentral/fileexchange/32950-polar-stereographic-coordinate-transformation--lat-lon-to-map-
latlon2ease_isprs() convert lat/lon to XY coordinates in EASE-Grid2.0 projecion, such as:
For its specification, please refer to <2014-ISPRS-Mary J. Brodzik -EASE-Grid 2.0: Incremental but Significant Improvements for Earth-Gridded Data Sets> and <Map Projections-A Working Manual>(P187&333)
Margin_nan
Read NetCDF to extract three arrays of latitude, longitude and parameter. To void the resampling error, this function expand the three original arrays with peripheral nan. Supplementing scale_pool() to judge if this parameter need scale 100 before writing into NetCDF file to save storage space.
For parameter, it is the only goal to padding with NaN marginally. However for lat or lon arrays of same size, certain values should be assigned to padding pixel. Hopefully linear interpolation wound not induce such large geolocation error.
  regular_grid
make a coherent regular grid, which is equally-spaced in projection coordinates. Fixing the up-left corner and array size before repmat() matrix, be carefully to shift half of pixel size at last. 
carto_png
ATTENTION! This function is based on m_map library, please download from https://www.eoas.ubc.ca/~rich/map.html  and install the package before mapping, insight from https://www.eoas.ubc.ca/~rich/map.html#2._SSMI_Ice_cover helps as well. There are a few tricks need clarify:
1.	Polar and out-bound(So far 50°N/S is sufficient to represent existing datasets.) are needed to construct m_proj object.
2.	Colormap should keep consistent for same kinds of parameter, otherwise the maps in time-series are not comparable.
3.	Distance comparisons are made between 2*out-bound radius and dimension of datasets in certain projection, then the original parameter array is expanded to the larger one, or keep itself if dimension is larger than 2*out-bound radius. All above is to ensure area inside the out-bound circle, whether it has valid data or not, is filled with deep blue, area outside is filled with white at the same time. Besides, all pixel further than out-bound radius was set to non.
4.	To achieve the goals of 2 & 4, ‘CDataMapping’ property in image() function should be set as ‘scale’. At the same time, to suppress the maximum in dataset into deep-red instead of white, a scale factor slightly larger than 1.01 is necessary because initial colormap has jet(100) + white([1,1,2]).
5.	Out-bound circle’s longitudinal label’s property in m_grid() is different for Arctic and Antarctic.
6.	Set figure() property as ‘off’ temporarily, so the program export PNG file directly without showing figure explicitly.
M_grid_lt
ATTENTION! Teng Li 20170815 modified original m_grid script by commenting the graticule label of latitude to serve cartographic requirement of project manager(as well as other slight modification, such as font size, etc). Please save this file in the installing path of m_map library (same directory as original m_grid).
Write_netcdf
Construct netcdf schema with two dimensions and three variables, with their numerical type specified. Supplement projection information at last modifying.
Write_geotiff
Construct structure of projection information, whose format was specified by MATLAB, please use ‘geotiffread()’ to explore how this structure is organized if you want to know more. For ‘GeoKeyDirectoryTag’, please refer to GeoTIFF file foramt’s official websites for detail http://geotiff.maptools.org/spec/geotiff6.html .
