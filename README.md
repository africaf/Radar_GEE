# Interpreting SAR data for forest monitoring

## Radar remote sensing for forest monitoring in Google Earth Engine

Africa I. Flores-Anderson 19-05-2023

## Learning objectives:
* Visualize and interpret deforestation events using Sentinel-1 and Alos Palsar data in GEE
* Plot time series data from Sentinel-1 and Alos Palsar and explain changes in forest cover 
* Distinguish how SAR C-band  and L-band see deforestation events 

## Exercise 1: Visualize Time series of Sentinel-1 and Alos Palsar data over a deforestation event
First let's check how a deforestation event looks with Optical data. We will explore a deforestation event in Papau New Guinea as detected by [RADD Forest Disturbance Alert](https://nrtwur.users.earthengine.app/view/raddalert). 

There is availability of [ALOS Palsar ScanSAR](https://developers.google.com/earth-engine/datasets/catalog/JAXA_ALOS_PALSAR-2_Level2_2_ScanSAR#description) data in GEE over Oceania and South East Asia that we can use together with [Sentinel-1 data](https://developers.google.com/earth-engine/datasets/catalog/COPERNICUS_S1_GRD). 

The location of interest is Lat: -4.011 &deg; , Lon: 141.303 &deg;  

1. Open the code editor of GEE: https://code.earthengine.google.com/ 
2. Copy and paste the following code blocks
3. 
```java
//Create variable with the point location of interest
var defPaG = ee.Geometry.Point([141.303, -4.011])

// Define a rectangular area of interest, by listing coordinates
var aoi = ee.Geometry.Polygon(
        [[[141.2500740578722,-4.04103621945203],
          [141.3599373391222,-4.04103621945203],
          [141.3599373391222,-3.9691145182502945],
          [141.2500740578722,-3.9691145182502945]]], null, false);

Map.addLayer(aoi, {}, 'aoi');

// Zoom to area of of interest
Map.centerObject(aoi,10);
```

