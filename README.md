# Interpreting SAR data for forest monitoring

## Radar remote sensing for forest monitoring in Google Earth Engine

Africa I. Flores-Anderson 19-05-2023

## Learning objectives:
* Visualize and interpret deforestation events using Sentinel-1 and Alos Palsar data in GEE
* Plot time series data from Sentinel-1 and Alos Palsar and explain changes in forest cover 
* Distinguish how SAR C-band  and L-band see deforestation events 


## Using Google Earth Engine
1. Open the [code editor of GEE](https://code.earthengine.google.com/) 
2. The following figure describes the GEE code editor interface

![fig](/Figures/GEE_Interface5.png)
<sub>Figure 1. GEE User Interface. </sub>

## Location of Interest
We will explore a deforestation event in Papau New Guinea as detected by [RADD Forest Disturbance Alert](https://nrtwur.users.earthengine.app/view/raddalert). The location of interest is Lat: -4.011° , Lon: 141.303°. Below we can see a detailed time series of this location using Planet data. A main deforestation event is first visible on April 2022. 

<img src="Figures/DeforestationPaNG2.png"  width="988" height="233">
<sub>Figure 2. Time series of location of interest in optical data . </sub>

We will explore in the following exercises how this event is seen by radar data, both Sentinel-1 and Alos Palsar. 


## Exercises:
1. [Time series visualization Sentinel-1 and Alos Palsar](/Exercise1_TimeSeries.md)
2. [Plotting Time series values and calculation of Vegetation Indices from Sentinel-1 and Alos Palsar](/Exercise2.Plotting_time_series.md)
