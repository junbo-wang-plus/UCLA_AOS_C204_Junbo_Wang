# Estimating Local Air Quality with Ground-based Aerosol Optical Depth Measurements and Meteorological Conditions

This report describes the approach of using several machine learning models to estimate local Air Quality Index (AQI) values using Aerosol Optical Depth (AOD) measurements from AeroNET and local meteorological conditions on an interpolated grid.

***

## Introduction 

This effort is driven by the desire to predict daily average air quality represented by AQI using the minimum amount of information, including the most basic local meteorological conditions and  AOD measurements from a sparsely located ground-based instrument network (AERONET), without relying on satellite measurements which are ineffective in cloudy conditions or complex global chemistry models. 

## Data

Three datasets are used in this project:  

1. [Daily AQI][1] summary in the year 2022 provided by the EPA includes the monitor site locations, ozone concentrations, and AQI values. Monitor locations are used for interpolation onto a uniform grid, ozone concentrations are used as an input feature, and log(AQI) values are the regression target due to the wide range of raw AQI values. A cleaned up snippet of this dataset is shown below:   

	`aqi table`

2. [Meteorological conditions][2] including monitor site locations, daily averaged precipitation, snow depth, snowfall, maximum temperature, and minimum temperature provided by NOAA are interpolated in the same manner as AQI and aggregated as input features. Similarly, below is an example of the raw dataset:

	`met table`
4. [Aerosol Optical Depths][3] measured by a network of ground-based instruments called the Aerosol Robotic Network (AERONET) covering a range of wavelengths are the final components of input features (example shown below).

	`AOD table`

After aggregating all input features and trimming off undesired features such as faulty measurements of AOD on some wavelengths, 16 input features were obtained and are shown below. Note that "ELEVATION" references monitor sites elevation of meteorological data while "Site_Elevation(m)" refers those of AOD instruments.

<table>
    <tr>
		<th>Ozone</th>
		<th>ELEVATION</th>
    </tr>
	<tr>
		<th>PRCP</th>
		<th>SNOW</th>
	</tr>
	<tr>
		<th>SNWD</th>
		<th>TMAX</th>
  	</tr>
	<tr>
		<th>TMIN</th>
		<th>Site_Elevation(m)</th>
	</tr>
	<tr>
		<th>AOD_1640nm</th>
		<th>AOD_1020nm</th>
	</tr>
	<tr>
		<th>AOD_870nm</th>
		<th>AOD_675nm</th>
	</tr>
	<tr>
		<th>AOD_500nm</th>
		<th>AOD_440nm</th>
	</tr>
	<tr>
		<th>AOD_380nm</th>
		<th>AOD_340nm</th>
	</tr>
</table>

All three datasets have a similar format with monitor site locations and measurement dates. The three datasets for the entirety of 2022 were downloaded but only data from March 1 to May 1 are used due to memory limitations.

Data preprocessing includes minor renaming, reformatting, and replacing invalid values before looping through the selected dates. In each loop, only the data on the specified date in each dataset are selected and separately interpolated onto a uniform grid covering the mainland U.S. with a resolution of 0.5 degrees. After each iteration, the interpolated values for each feature are flattened to a 1-D vector and appended with those from other dates. Hence the number of training instances without trimming off invalid data is the number of individual cells on the uniform grid multiplied by the number of days, which in this case is 272,800 instances. The preprocessing loop is shown below:

[1]: https://aqs.epa.gov/aqsweb/airdata/download_files.html#AQI
[2]: https://www.ncei.noaa.gov/maps/daily/
[3]: https://aeronet.gsfc.nasa.gov/

```python
preprocessing loop
```

Below are examples of interpolated values of one feature from each of the three datasets for March 1 using the "nearest" method plotted onto the uniform grid:

`PM graph`  
`TMAX graph`  
`AOD graph`

It is shown that AOD measurements are sparse even for a relatively coarse grid of 0.5 degree resolution. The original plan was to include satellite measurements at higher resolutions to increase the robustness. However, due to the reason mentioned before that satellite measurements are unusable at cloudy conditions and that satellite data products were given in a format difficult to implement in this project, only the three datasets introduced are used. The errors introduced by the sparseness of ground-based AOD measurements are unavoidably large and will be discussed in the discussion section.

## Model prediction
After the numerous testing, the only model capable of providing reliable results is the decision tree regressor from the sci-kit learn library. The model is incorporated into a pipeline with a standard scaler and the process is shown below:

```python
model code
```

The RMSE of scaled data for this model is `120` and its REC is shown below:
```
REC
```

## Discussions
Besides the successful decision tree regression, linear regression, artificial neural network, random forest, and deep neural network models are also tested and they all produced unacceptable results. RECs for all models tested are shown below:

It is unclear why decision tree model outperforms other models by such a significant margin besides its known advantage in nonlinear behavior. The poor performance of other models can be attributed to the interpolation method used, which introduces a significant amount of errors especially in large regions with sparsely located monitor sites. Adding this to the already sparse AOD measurements even in more populated areas, it is clear how simple models such as linear regression will not produce reliable results especially when there is not enough physically relevant features to PM2.5 except for AOD measurements. 
## References
[1] DALL-E 3

[back](./)

