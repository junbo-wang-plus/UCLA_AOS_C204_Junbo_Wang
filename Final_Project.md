```python
import matplotlib.pyplot as plt
import pandas as pd
import numpy as np
from scipy.interpolate import griddata
import cartopy
import cartopy.crs as ccrs
import cartopy.feature as cfeature
from matplotlib.colors import LogNorm
```


```python
# Read PM2.5 and Ozone data
filepath = 'Datasets/daily_44201_2022.csv'
data = pd.read_csv(filepath)
pd.set_option('display.max_columns', None)

df = data[['Date Local', 'Latitude', 'Longitude', 'Arithmetic Mean', 'AQI']]
df = df.rename(columns={'Arithmetic Mean': 'Ozone'})
df.head(5)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Date Local</th>
      <th>Latitude</th>
      <th>Longitude</th>
      <th>Ozone</th>
      <th>AQI</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2022-02-28</td>
      <td>30.497478</td>
      <td>-87.880258</td>
      <td>0.038000</td>
      <td>35</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2022-03-01</td>
      <td>30.497478</td>
      <td>-87.880258</td>
      <td>0.037235</td>
      <td>50</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2022-03-02</td>
      <td>30.497478</td>
      <td>-87.880258</td>
      <td>0.038235</td>
      <td>51</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2022-03-03</td>
      <td>30.497478</td>
      <td>-87.880258</td>
      <td>0.024333</td>
      <td>40</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2022-03-04</td>
      <td>30.497478</td>
      <td>-87.880258</td>
      <td>0.049647</td>
      <td>77</td>
    </tr>
  </tbody>
</table>
</div>




```python
# Read weather data
file2 = 'Datasets/3524590.csv'
data2 = pd.read_csv(file2)
df2 = data2[['DATE', 'LATITUDE', 'LONGITUDE', 'ELEVATION','PRCP','SNOW','SNWD','TMAX','TMIN']]
df2.head(5)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>DATE</th>
      <th>LATITUDE</th>
      <th>LONGITUDE</th>
      <th>ELEVATION</th>
      <th>PRCP</th>
      <th>SNOW</th>
      <th>SNWD</th>
      <th>TMAX</th>
      <th>TMIN</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2022-03-01</td>
      <td>44.4044</td>
      <td>-123.7533</td>
      <td>70.1</td>
      <td>2.27</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>56.0</td>
      <td>51.0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2022-03-02</td>
      <td>44.4044</td>
      <td>-123.7533</td>
      <td>70.1</td>
      <td>0.74</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>52.0</td>
      <td>49.0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2022-03-03</td>
      <td>44.4044</td>
      <td>-123.7533</td>
      <td>70.1</td>
      <td>0.62</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>50.0</td>
      <td>45.0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2022-03-04</td>
      <td>44.4044</td>
      <td>-123.7533</td>
      <td>70.1</td>
      <td>0.02</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>49.0</td>
      <td>38.0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2022-03-05</td>
      <td>44.4044</td>
      <td>-123.7533</td>
      <td>70.1</td>
      <td>0.15</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>44.0</td>
      <td>36.0</td>
    </tr>
  </tbody>
</table>
</div>




```python
# Read Aeronet data
file3 = 'Datasets/aeronet2.csv'
df3 = pd.read_csv(file3)
df3.head(5)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Date</th>
      <th>Latitude</th>
      <th>Longitude</th>
      <th>Site_Elevation(m)</th>
      <th>AOD_1640nm</th>
      <th>AOD_1020nm</th>
      <th>AOD_870nm</th>
      <th>AOD_865nm</th>
      <th>AOD_779nm</th>
      <th>AOD_675nm</th>
      <th>AOD_667nm</th>
      <th>AOD_620nm</th>
      <th>AOD_560nm</th>
      <th>AOD_555nm</th>
      <th>AOD_551nm</th>
      <th>AOD_532nm</th>
      <th>AOD_531nm</th>
      <th>AOD_510nm</th>
      <th>AOD_500nm</th>
      <th>AOD_490nm</th>
      <th>AOD_443nm</th>
      <th>AOD_440nm</th>
      <th>AOD_412nm</th>
      <th>AOD_400nm</th>
      <th>AOD_380nm</th>
      <th>AOD_340nm</th>
      <th>AOD_681nm</th>
      <th>AOD_709nm</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2022-12-30</td>
      <td>-62.663056</td>
      <td>-60.389444</td>
      <td>5.0</td>
      <td>0.049188</td>
      <td>0.059211</td>
      <td>0.062151</td>
      <td>-999.0</td>
      <td>-999.0</td>
      <td>0.066013</td>
      <td>-999.0</td>
      <td>-999.0</td>
      <td>-999.0</td>
      <td>-999.0</td>
      <td>-999.0</td>
      <td>-999.0</td>
      <td>-999.0</td>
      <td>-999.0</td>
      <td>0.073626</td>
      <td>-999.0</td>
      <td>-999.0</td>
      <td>0.083497</td>
      <td>-999.0</td>
      <td>-999.0</td>
      <td>0.079924</td>
      <td>0.082425</td>
      <td>-999.0</td>
      <td>-999.0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2022-03-01</td>
      <td>40.451900</td>
      <td>-3.723950</td>
      <td>680.0</td>
      <td>-999.000000</td>
      <td>0.050193</td>
      <td>0.056500</td>
      <td>-999.0</td>
      <td>-999.0</td>
      <td>0.067353</td>
      <td>-999.0</td>
      <td>-999.0</td>
      <td>-999.0</td>
      <td>-999.0</td>
      <td>-999.0</td>
      <td>-999.0</td>
      <td>-999.0</td>
      <td>-999.0</td>
      <td>0.095629</td>
      <td>-999.0</td>
      <td>-999.0</td>
      <td>0.115899</td>
      <td>-999.0</td>
      <td>-999.0</td>
      <td>0.140979</td>
      <td>0.151545</td>
      <td>-999.0</td>
      <td>-999.0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2022-03-02</td>
      <td>40.451900</td>
      <td>-3.723950</td>
      <td>680.0</td>
      <td>-999.000000</td>
      <td>0.020488</td>
      <td>0.023579</td>
      <td>-999.0</td>
      <td>-999.0</td>
      <td>0.028519</td>
      <td>-999.0</td>
      <td>-999.0</td>
      <td>-999.0</td>
      <td>-999.0</td>
      <td>-999.0</td>
      <td>-999.0</td>
      <td>-999.0</td>
      <td>-999.0</td>
      <td>0.040038</td>
      <td>-999.0</td>
      <td>-999.0</td>
      <td>0.047169</td>
      <td>-999.0</td>
      <td>-999.0</td>
      <td>0.058094</td>
      <td>0.058613</td>
      <td>-999.0</td>
      <td>-999.0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2022-03-05</td>
      <td>40.451900</td>
      <td>-3.723950</td>
      <td>680.0</td>
      <td>-999.000000</td>
      <td>0.033906</td>
      <td>0.037883</td>
      <td>-999.0</td>
      <td>-999.0</td>
      <td>0.044933</td>
      <td>-999.0</td>
      <td>-999.0</td>
      <td>-999.0</td>
      <td>-999.0</td>
      <td>-999.0</td>
      <td>-999.0</td>
      <td>-999.0</td>
      <td>-999.0</td>
      <td>0.064754</td>
      <td>-999.0</td>
      <td>-999.0</td>
      <td>0.079289</td>
      <td>-999.0</td>
      <td>-999.0</td>
      <td>0.097636</td>
      <td>0.102812</td>
      <td>-999.0</td>
      <td>-999.0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2022-03-06</td>
      <td>40.451900</td>
      <td>-3.723950</td>
      <td>680.0</td>
      <td>-999.000000</td>
      <td>0.019403</td>
      <td>0.023290</td>
      <td>-999.0</td>
      <td>-999.0</td>
      <td>0.030635</td>
      <td>-999.0</td>
      <td>-999.0</td>
      <td>-999.0</td>
      <td>-999.0</td>
      <td>-999.0</td>
      <td>-999.0</td>
      <td>-999.0</td>
      <td>-999.0</td>
      <td>0.045460</td>
      <td>-999.0</td>
      <td>-999.0</td>
      <td>0.056093</td>
      <td>-999.0</td>
      <td>-999.0</td>
      <td>0.069005</td>
      <td>0.071608</td>
      <td>-999.0</td>
      <td>-999.0</td>
    </tr>
  </tbody>
</table>
</div>




```python
# GRID PARAMETERS
degree = 0.5
# Step 1: Define grid boundaries and resolution
#on_min, lon_max = -125, -115 #CA
#lat_min, lat_max = 32, 42 #CA

lon_min, lon_max = -125, -70 #US
lat_min, lat_max = 32, 52 #US

#lon_min, lon_max = -180, 180 #US
#lat_min, lat_max = -90, 90 #US

#lon_min, lon_max = -25, 45 #EU
#lat_min, lat_max = 34, 72 #EU

#lon_min, lon_max = -5, 20 #EU land
#lat_min, lat_max = 40, 55 #EU land

nlon = int((lon_max-lon_min)/degree)
nlat = int((lat_max-lat_min)/degree)

lon_grid = np.linspace(lon_min, lon_max, nlon)
lat_grid = np.linspace(lat_min, lat_max, nlat)
lon_grid, lat_grid = np.meshgrid(lon_grid, lat_grid)
```


```python
def assemble(combined_df,df,df2,df3,date,lon_min,lon_max,lat_min,lat_max,lon_grid,lat_grid):
    temp_df = pd.DataFrame()
    # Fit date for PM and Ozone data
    new_df = df[df['Date Local'] == date].copy()
    # Clean up PM2.5 and Ozone data
    new_df.reset_index(drop=True, inplace=True)
    new_df.drop(labels=['Date Local'], axis=1, inplace=True)
    new_df = new_df[new_df['AQI'] != 0]
    new_df = new_df[new_df['Ozone'] != 0]
    
    new_df['AQI'] = new_df['AQI'].apply(lambda x: np.log(x))
    new_df['Ozone'] = new_df['Ozone'].apply(lambda x: np.log(x*100))
    
    # Interpolate PM2.5 and Ozone data onto the grid
    points = new_df[['Longitude', 'Latitude']].values
    lon = points[:, 0].astype(float)
    lat = points[:, 1].astype(float)
    pm25_values = new_df['AQI'].values
    ozone_values = new_df['Ozone'].values
    pm25_grid = griddata(points, pm25_values, (lon_grid, lat_grid), method='nearest')
    ozone_grid = griddata(points, ozone_values, (lon_grid, lat_grid), method='nearest')
    
    # flatten pm and ozone data
    pm25_flat = pm25_grid.flatten()
    ozone_flat = ozone_grid.flatten()
    temp_df['AQI'] = pm25_flat
    temp_df['Ozone'] = ozone_flat
    
    # Fit date for weather data
    new_df2 = df2[df2['DATE'] == date].copy()
    
    # Clean up weather data
    new_df2 = new_df2.dropna(subset=['TMAX','TMIN','PRCP','SNOW','SNWD'])
    
    # weather interpolation 
    def MET_clean_int(df,col,lon_grid,lat_grid,met):
        points = df[['LONGITUDE', 'LATITUDE']].values
        lon = points[:, 0].astype(float)
        lat = points[:, 1].astype(float)
        values = df[col].values
        grid = griddata(points, values, (lon_grid, lat_grid), method=met)
        return grid
    
    # flatten weather data
    met_var = ['ELEVATION','PRCP','SNOW','SNWD','TMAX','TMIN']
    for i in range(len(met_var)):
        grid = MET_clean_int(new_df2,met_var[i],lon_grid,lat_grid,'nearest')
        flat = grid.flatten()
        temp_df[met_var[i]] = flat
    
    # Fit date for aeronet data
    new_df3 = df3[df3['Date'] == date].copy()
    
    # Aeronet interpolation
    def AOD_clean_int(df,col,lon_grid,lat_grid,met):
        new_df = df[df[col] != -999.000]
        points = new_df[['Longitude', 'Latitude']].values
        lon = points[:, 0].astype(float)
        lat = points[:, 1].astype(float)
        values = new_df[col].values
        grid = griddata(points, values, (lon_grid, lat_grid), method=met)
        return grid
    
    # flatten aeronet data
    aeronet_var = [
        'Site_Elevation(m)', 'AOD_1640nm','AOD_1020nm', 'AOD_870nm',  'AOD_675nm', 'AOD_500nm', 
        'AOD_440nm', 'AOD_380nm','AOD_340nm'
    ]
    for i in range(len(aeronet_var)):
        grid = AOD_clean_int(new_df3,aeronet_var[i],lon_grid,lat_grid,'nearest')
        flat = grid.flatten()
        temp_df[aeronet_var[i]] = flat

    frames = [combined_df,temp_df]
    combined_df = pd.concat(frames)

    if date == '2022-03-01':
        fig, ax = plt.subplots(figsize=(20, 8), subplot_kw={'projection': ccrs.PlateCarree()})
        ax.set_extent([lon_min, lon_max, lat_min, lat_max], crs=ccrs.PlateCarree())
        cax = ax.pcolormesh(lon_grid, lat_grid, pm25_grid, cmap='viridis', shading='auto', transform=ccrs.PlateCarree())
        #plt.scatter(lon, lat)
        ax.add_feature(cfeature.COASTLINE)
        cbar = fig.colorbar(cax)
        ax.set_xlabel('Longitude')
        ax.set_ylabel('Latitude')
        ax.set_title('Interpolated Daily Average log(PM2.5) using Nearest Method')
        #plt.savefig('nearest_CA.jpg', dpi=300)
        plt.show()

        tmax_grid = MET_clean_int(new_df2,'TMAX',lon_grid,lat_grid,'nearest')
        fig, ax = plt.subplots(figsize=(20, 8), subplot_kw={'projection': ccrs.PlateCarree()})
        ax.set_extent([lon_min, lon_max, lat_min, lat_max], crs=ccrs.PlateCarree())
        cax = ax.pcolormesh(lon_grid, lat_grid, tmax_grid, cmap='viridis', shading='auto', transform=ccrs.PlateCarree())
        #plt.scatter(lon, lat)
        ax.add_feature(cfeature.COASTLINE)
        cbar = fig.colorbar(cax)
        ax.set_xlabel('Longitude')
        ax.set_ylabel('Latitude')
        ax.set_title('Interpolated Daily Maximum Temperature using Nearest Method')
        #plt.savefig('nearest_CA.jpg', dpi=300)
        plt.show()

        aod_grid = AOD_clean_int(new_df3,'AOD_870nm',lon_grid,lat_grid,'nearest')
        fig, ax = plt.subplots(figsize=(20, 8), subplot_kw={'projection': ccrs.PlateCarree()})
        ax.set_extent([lon_min, lon_max, lat_min, lat_max], crs=ccrs.PlateCarree())
        cax = ax.pcolormesh(lon_grid, lat_grid, aod_grid, cmap='viridis', shading='auto', transform=ccrs.PlateCarree())
        #plt.scatter(lon, lat)
        ax.add_feature(cfeature.COASTLINE)
        cbar = fig.colorbar(cax)
        ax.set_xlabel('Longitude')
        ax.set_ylabel('Latitude')
        ax.set_title('Interpolated AOD at 870 nm using Nearest Method')
        #plt.savefig('nearest_CA.jpg', dpi=300)
        plt.show()
        
    return combined_df
```


```python
start_date = '2022-03-01'
end_date = '2022-05-01'
dates = pd.date_range(start=start_date, end=end_date)
dates = [d.date() for d in dates]
date_list = [d.strftime('%Y-%m-%d') for d in dates]
combined_df = pd.DataFrame()

for date in date_list:
    combined_df = assemble(combined_df,df,df2,df3,date,lon_min,lon_max,lat_min,lat_max,lon_grid,lat_grid)
```


    
![png](output_6_0.png)
    



    
![png](output_6_1.png)
    



    
![png](output_6_2.png)
    



```python
combined_df.describe()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>AQI</th>
      <th>Ozone</th>
      <th>ELEVATION</th>
      <th>PRCP</th>
      <th>SNOW</th>
      <th>SNWD</th>
      <th>TMAX</th>
      <th>TMIN</th>
      <th>Site_Elevation(m)</th>
      <th>AOD_1640nm</th>
      <th>AOD_1020nm</th>
      <th>AOD_870nm</th>
      <th>AOD_675nm</th>
      <th>AOD_500nm</th>
      <th>AOD_440nm</th>
      <th>AOD_380nm</th>
      <th>AOD_340nm</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>count</th>
      <td>272800.000000</td>
      <td>272800.000000</td>
      <td>272800.000000</td>
      <td>272800.000000</td>
      <td>272800.000000</td>
      <td>272800.000000</td>
      <td>272800.000000</td>
      <td>272800.000000</td>
      <td>272800.000000</td>
      <td>272800.000000</td>
      <td>272800.000000</td>
      <td>272800.000000</td>
      <td>272800.000000</td>
      <td>272800.000000</td>
      <td>272800.000000</td>
      <td>272800.000000</td>
      <td>272800.000000</td>
    </tr>
    <tr>
      <th>mean</th>
      <td>3.671220</td>
      <td>1.287508</td>
      <td>634.911407</td>
      <td>0.084096</td>
      <td>0.124856</td>
      <td>1.239775</td>
      <td>55.508941</td>
      <td>31.687848</td>
      <td>630.146989</td>
      <td>0.035964</td>
      <td>0.051996</td>
      <td>0.058227</td>
      <td>0.072020</td>
      <td>0.098856</td>
      <td>0.113137</td>
      <td>0.129516</td>
      <td>0.140808</td>
    </tr>
    <tr>
      <th>std</th>
      <td>0.203017</td>
      <td>0.226666</td>
      <td>629.163475</td>
      <td>0.239985</td>
      <td>0.639336</td>
      <td>4.540302</td>
      <td>16.191569</td>
      <td>13.564992</td>
      <td>660.764261</td>
      <td>0.029118</td>
      <td>0.034647</td>
      <td>0.037557</td>
      <td>0.045898</td>
      <td>0.062943</td>
      <td>0.072533</td>
      <td>0.086426</td>
      <td>0.096469</td>
    </tr>
    <tr>
      <th>min</th>
      <td>0.693147</td>
      <td>-3.189317</td>
      <td>1.200000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>8.000000</td>
      <td>-34.000000</td>
      <td>-32.000000</td>
      <td>-0.002763</td>
      <td>0.004974</td>
      <td>0.005876</td>
      <td>0.005940</td>
      <td>0.009877</td>
      <td>0.013738</td>
      <td>0.013624</td>
      <td>0.011249</td>
    </tr>
    <tr>
      <th>25%</th>
      <td>3.583519</td>
      <td>1.179670</td>
      <td>183.800000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>44.000000</td>
      <td>24.000000</td>
      <td>186.000000</td>
      <td>0.015336</td>
      <td>0.027362</td>
      <td>0.031255</td>
      <td>0.040188</td>
      <td>0.056383</td>
      <td>0.064412</td>
      <td>0.071326</td>
      <td>0.076389</td>
    </tr>
    <tr>
      <th>50%</th>
      <td>3.688879</td>
      <td>1.319406</td>
      <td>396.500000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>55.000000</td>
      <td>31.000000</td>
      <td>373.440000</td>
      <td>0.027958</td>
      <td>0.042280</td>
      <td>0.047775</td>
      <td>0.059227</td>
      <td>0.081134</td>
      <td>0.092278</td>
      <td>0.104479</td>
      <td>0.112596</td>
    </tr>
    <tr>
      <th>75%</th>
      <td>3.784190</td>
      <td>1.435085</td>
      <td>932.700000</td>
      <td>0.030000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>67.000000</td>
      <td>40.000000</td>
      <td>808.000000</td>
      <td>0.047737</td>
      <td>0.068086</td>
      <td>0.075762</td>
      <td>0.091700</td>
      <td>0.124078</td>
      <td>0.140639</td>
      <td>0.160992</td>
      <td>0.174667</td>
    </tr>
    <tr>
      <th>max</th>
      <td>5.010635</td>
      <td>2.017247</td>
      <td>2724.600000</td>
      <td>3.500000</td>
      <td>20.000000</td>
      <td>80.000000</td>
      <td>100.000000</td>
      <td>71.000000</td>
      <td>3376.000000</td>
      <td>0.306340</td>
      <td>0.324715</td>
      <td>0.374396</td>
      <td>0.500045</td>
      <td>0.714082</td>
      <td>0.811168</td>
      <td>0.929410</td>
      <td>1.008788</td>
    </tr>
  </tbody>
</table>
</div>




```python
combined_df.head(5)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>AQI</th>
      <th>Ozone</th>
      <th>ELEVATION</th>
      <th>PRCP</th>
      <th>SNOW</th>
      <th>SNWD</th>
      <th>TMAX</th>
      <th>TMIN</th>
      <th>Site_Elevation(m)</th>
      <th>AOD_1640nm</th>
      <th>AOD_1020nm</th>
      <th>AOD_870nm</th>
      <th>AOD_675nm</th>
      <th>AOD_500nm</th>
      <th>AOD_440nm</th>
      <th>AOD_380nm</th>
      <th>AOD_340nm</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>3.78419</td>
      <td>1.106415</td>
      <td>527.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>71.0</td>
      <td>37.0</td>
      <td>50.0</td>
      <td>0.027825</td>
      <td>0.033055</td>
      <td>0.034693</td>
      <td>0.035636</td>
      <td>0.042980</td>
      <td>0.049442</td>
      <td>0.053372</td>
      <td>0.048359</td>
    </tr>
    <tr>
      <th>1</th>
      <td>3.78419</td>
      <td>1.106415</td>
      <td>527.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>71.0</td>
      <td>37.0</td>
      <td>33.0</td>
      <td>0.027825</td>
      <td>0.010138</td>
      <td>0.012280</td>
      <td>0.013081</td>
      <td>0.020905</td>
      <td>0.027062</td>
      <td>0.029704</td>
      <td>0.031415</td>
    </tr>
    <tr>
      <th>2</th>
      <td>3.78419</td>
      <td>1.106415</td>
      <td>527.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>71.0</td>
      <td>37.0</td>
      <td>33.0</td>
      <td>0.027825</td>
      <td>0.010138</td>
      <td>0.012280</td>
      <td>0.013081</td>
      <td>0.020905</td>
      <td>0.027062</td>
      <td>0.029704</td>
      <td>0.031415</td>
    </tr>
    <tr>
      <th>3</th>
      <td>3.78419</td>
      <td>1.106415</td>
      <td>393.2</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>81.0</td>
      <td>43.0</td>
      <td>33.0</td>
      <td>0.027825</td>
      <td>0.010138</td>
      <td>0.012280</td>
      <td>0.013081</td>
      <td>0.020905</td>
      <td>0.027062</td>
      <td>0.029704</td>
      <td>0.031415</td>
    </tr>
    <tr>
      <th>4</th>
      <td>3.78419</td>
      <td>1.106415</td>
      <td>393.2</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>81.0</td>
      <td>43.0</td>
      <td>33.0</td>
      <td>0.027825</td>
      <td>0.010138</td>
      <td>0.012280</td>
      <td>0.013081</td>
      <td>0.020905</td>
      <td>0.027062</td>
      <td>0.029704</td>
      <td>0.031415</td>
    </tr>
  </tbody>
</table>
</div>




```python
from sklearn.model_selection import train_test_split

target = 'AQI'

X_data=combined_df.drop([target],axis=1).values
y_data=combined_df[target].values
y_data = y_data.reshape(-1,1)

test_size = 0.3
X_train, X_test, y_train, y_test = train_test_split(X_data, y_data, test_size=test_size)

print("shape of X_train:")
print("", X_train.shape)
print("shape of y_train:")
print("", y_train.shape)
print()
print("shape of X_test:")
print("", X_test.shape)
print("shape of y_test:")
print("", y_test.shape)
print()
```

    shape of X_train:
     (190960, 16)
    shape of y_train:
     (190960, 1)
    
    shape of X_test:
     (81840, 16)
    shape of y_test:
     (81840, 1)
    



```python
# REC curve
def rec(m, n, tol):
    if not type(m) == 'numpy.ndarray':
        m = np.array(m) #change m to a np array
    if not type(n) == 'numpy.ndarray':
        n = np.array(n) #change n to a np array

    l = m.size
    percent = 0
    for i in range(l):
        if np.abs(10**m[i]-10**n[i])<=tol:
            percent+=1
    return 100*(percent/l)
```


```python
# Linear
from sklearn.pipeline import Pipeline
from sklearn.linear_model import Ridge
from sklearn.preprocessing import StandardScaler

#repeating the whole thing using a pipeline:
#with a scaler:
pipe = Pipeline([('scaler', StandardScaler()), ('lr', Ridge(alpha=1.0))])
pipe.fit(X_train, y_train)
y_pred_lr = pipe.predict(X_test)

print("RMSE for linear regression (with scaling):", np.sqrt(np.mean((y_test-y_pred_lr)**2)))
print()

tol_max = 40
rec_linear=[]

for i in range(tol_max):
    rec_linear.append(rec(y_pred_lr, y_test, i))

plt.figure(figsize=(5,5))
plt.title("REC curve for Ridge regression\n")
plt.xlabel("Absolute error (tolerance) in prediction")
plt.ylabel("Percentage of correct prediction")
plt.xticks([i*5 for i in range(tol_max+1)])
plt.ylim(-10,100)
plt.yticks([i*20 for i in range(6)])
plt.grid(True)
plt.plot(range(tol_max),rec_linear, label='linear')
plt.legend()
plt.show()
```

    RMSE for linear regression (with scaling): 0.10588709993935899
    



    
![png](output_11_1.png)
    



```python
from sklearn.tree import DecisionTreeRegressor

pipe2 = Pipeline([('scaler', StandardScaler()), ('regressor', DecisionTreeRegressor())])
pipe2.fit(X_train, y_train)
y_pred_tree = pipe2.predict(X_test)

print("RMSE for decision tree (with scaling):", np.sqrt(np.mean((y_test-y_pred_tree)**2)))
print()

# Decision Tree
rec_tree=[]

for i in range(tol_max):
    rec_tree.append(rec(y_pred_tree, y_test, i))

plt.figure(figsize=(5,5))
plt.title("REC curve for Ridge regression\n")
plt.xlabel("Absolute error (tolerance) in prediction")
plt.ylabel("Percentage of correct prediction")
plt.xticks([i*5 for i in range(tol_max+1)])
plt.ylim(-10,100)
plt.yticks([i*20 for i in range(6)])
plt.grid(True)
plt.plot(range(tol_max),rec_linear, label='linear')
plt.plot(range(tol_max),rec_tree, label='tree')
plt.legend()
plt.show()
```

    RMSE for decision tree (with scaling): 0.2869838134836316
    



    
![png](output_12_1.png)
    



```python
# ANN
from sklearn.neural_network import MLPRegressor

ann = MLPRegressor(hidden_layer_sizes=(15,10,8,5))
pipe3 = Pipeline([('scaler', StandardScaler()), ('mlp', ann)])
pipe3.fit(X_train, y_train)
y_pred_ann = pipe3.predict(X_test)

print("RMSE for neural network (with scaling):", np.sqrt(np.mean((y_test-y_pred_ann)**2)))


rec_ann=[]

for i in range(tol_max):
    rec_ann.append(rec(y_pred_ann, y_test, i))

plt.figure(figsize=(5,5))
plt.title("REC curve \n")
plt.xlabel("Absolute error (tolerance) in prediction")
plt.ylabel("Percentage of correct prediction")
plt.xticks([i*5 for i in range(tol_max+1)])
plt.ylim(-10,100)
plt.yticks([i*20 for i in range(6)])
plt.grid(True)
plt.plot(range(tol_max),rec_linear, label='linear')
plt.plot(range(tol_max),rec_tree, label='tree')
plt.plot(range(tol_max),rec_ann, label='neural network')
plt.legend()
plt.show()
```

    /home/junbo-wang/miniconda3/lib/python3.11/site-packages/sklearn/neural_network/_multilayer_perceptron.py:1625: DataConversionWarning: A column-vector y was passed when a 1d array was expected. Please change the shape of y to (n_samples, ), for example using ravel().
      y = column_or_1d(y, warn=True)


    RMSE for neural network (with scaling): 0.27512704311543584



    
![png](output_13_2.png)
    



```python
# DNN
import tensorflow as tf
from tensorflow.keras.models import Sequential
from tensorflow.keras.layers import Dense, Flatten
from sklearn.model_selection import train_test_split
from sklearn.preprocessing import StandardScaler
import numpy as np

dnn = Sequential([
    Flatten(input_shape=(X_train.shape[1],)),
    Dense(128, activation='relu'),
    Dense(128, activation='relu'),
    Dense(1, activation='sigmoid')
])

# Compile the model
dnn.compile(optimizer='adam', loss='binary_crossentropy', metrics=['accuracy'])

# Train the model
dnn.fit(X_train, y_train, epochs=10, batch_size=5000)

# Evaluate the model
y_pred_dnn = dnn.predict(X_test)

print("RMSE for deep neural network (with scaling):", np.sqrt(np.mean((y_test-y_pred_dnn)**2)))

rec_dnn=[]

for i in range(tol_max):
    rec_dnn.append(rec(y_pred_dnn, y_test, i))

plt.figure(figsize=(5,5))
plt.title("REC curve \n")
plt.xlabel("Absolute error (tolerance) in prediction")
plt.ylabel("Percentage of correct prediction")
plt.xticks([i*5 for i in range(tol_max+1)])
plt.ylim(-10,100)
plt.yticks([i*20 for i in range(6)])
plt.grid(True)
plt.plot(range(tol_max),rec_linear, label='linear')
plt.plot(range(tol_max),rec_tree, label='tree')
plt.plot(range(tol_max),rec_ann, label='neural network')
plt.plot(range(tol_max),rec_dnn, label='deep neural network')
plt.legend()
plt.show()
```

    2023-12-03 16:19:42.089792: I tensorflow/tsl/cuda/cudart_stub.cc:28] Could not find cuda drivers on your machine, GPU will not be used.
    2023-12-03 16:19:42.119323: I tensorflow/tsl/cuda/cudart_stub.cc:28] Could not find cuda drivers on your machine, GPU will not be used.
    2023-12-03 16:19:42.119930: I tensorflow/core/platform/cpu_feature_guard.cc:182] This TensorFlow binary is optimized to use available CPU instructions in performance-critical operations.
    To enable the following instructions: AVX2 FMA, in other operations, rebuild TensorFlow with the appropriate compiler flags.
    2023-12-03 16:19:42.663542: W tensorflow/compiler/tf2tensorrt/utils/py_utils.cc:38] TF-TRT Warning: Could not find TensorRT


    Epoch 1/10


    2023-12-03 16:19:43.142196: I tensorflow/compiler/xla/stream_executor/cuda/cuda_gpu_executor.cc:995] successful NUMA node read from SysFS had negative value (-1), but there must be at least one NUMA node, so returning NUMA node zero. See more at https://github.com/torvalds/linux/blob/v6.0/Documentation/ABI/testing/sysfs-bus-pci#L344-L355
    2023-12-03 16:19:43.174542: W tensorflow/core/common_runtime/gpu/gpu_device.cc:1960] Cannot dlopen some GPU libraries. Please make sure the missing libraries mentioned above are installed properly if you would like to use GPU. Follow the guide at https://www.tensorflow.org/install/gpu for how to download and setup the required libraries for your platform.
    Skipping registering GPU devices...


    39/39 [==============================] - 1s 4ms/step - loss: -3019.1299 - accuracy: 0.0000e+00
    Epoch 2/10
    39/39 [==============================] - 0s 4ms/step - loss: -20447.5469 - accuracy: 0.0000e+00
    Epoch 3/10
    39/39 [==============================] - 0s 4ms/step - loss: -83720.4141 - accuracy: 0.0000e+00
    Epoch 4/10
    39/39 [==============================] - 0s 3ms/step - loss: -244866.7812 - accuracy: 0.0000e+00
    Epoch 5/10
    39/39 [==============================] - 0s 3ms/step - loss: -559147.7500 - accuracy: 0.0000e+00
    Epoch 6/10
    39/39 [==============================] - 0s 3ms/step - loss: -1080093.1250 - accuracy: 0.0000e+00
    Epoch 7/10
    39/39 [==============================] - 0s 3ms/step - loss: -1860415.7500 - accuracy: 0.0000e+00
    Epoch 8/10
    39/39 [==============================] - 0s 3ms/step - loss: -2952749.0000 - accuracy: 0.0000e+00
    Epoch 9/10
    39/39 [==============================] - 0s 3ms/step - loss: -4402827.5000 - accuracy: 0.0000e+00
    Epoch 10/10
    39/39 [==============================] - 0s 3ms/step - loss: -6259467.5000 - accuracy: 0.0000e+00
    2558/2558 [==============================] - 1s 375us/step
    RMSE for deep neural network (with scaling): 2.6788514809524733



    
![png](output_14_4.png)
    



```python
# Ensemble
from sklearn.ensemble import RandomForestRegressor, GradientBoostingRegressor
from sklearn.linear_model import LinearRegression
from sklearn.ensemble import VotingRegressor
from sklearn.model_selection import train_test_split
from sklearn.datasets import make_regression
from sklearn.metrics import mean_squared_error


# Create the individual models
model1 = LinearRegression()
model2 = RandomForestRegressor(n_estimators=50, random_state=42)
model3 = GradientBoostingRegressor(random_state=42)
model4 = DecisionTreeRegressor()

# Create the ensemble model
voting_regressor = VotingRegressor(estimators=[
    ('lr', model1),
    ('rf', model2),
    ('gbr', model3),                              
    ('dt', model4),   
])

voting_regressor.fit(X_train, y_train)
# Make predictions
y_pred_ens = voting_regressor.predict(X_test)

print("RMSE for ensemble (with scaling):", np.sqrt(np.mean((y_test-y_pred_ens)**2)))

rec_ens=[]

for i in range(tol_max):
    rec_ens.append(rec(y_pred_ens, y_test, i))

plt.figure(figsize=(5,5))
plt.title("REC curve \n")
plt.xlabel("Absolute error (tolerance) in prediction")
plt.ylabel("Percentage of correct prediction")
plt.xticks([i*5 for i in range(tol_max+1)])
plt.ylim(-10,100)
plt.yticks([i*20 for i in range(6)])
plt.grid(True)
plt.plot(range(tol_max),rec_linear, label='linear')
plt.plot(range(tol_max),rec_tree, label='decision tree')
plt.plot(range(tol_max),rec_ann, label='neural network')
plt.plot(range(tol_max),rec_dnn, label='deep neural network')
plt.plot(range(tol_max),rec_ens, label='ensemble')
plt.legend()
plt.show()
```

    /home/junbo-wang/miniconda3/lib/python3.11/site-packages/sklearn/ensemble/_voting.py:604: DataConversionWarning: A column-vector y was passed when a 1d array was expected. Please change the shape of y to (n_samples, ), for example using ravel().
      y = column_or_1d(y, warn=True)


    RMSE for ensemble (with scaling): 0.27236187287972274



    
![png](output_15_2.png)
    



```python

```
