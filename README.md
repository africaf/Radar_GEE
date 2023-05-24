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

## Exercise 1: Visualize Time series of Sentinel-1 and Alos Palsar data over a deforestation event
First let's check how a deforestation event looks with Optical data. We will explore a deforestation event in Papau New Guinea as detected by [RADD Forest Disturbance Alert](https://nrtwur.users.earthengine.app/view/raddalert). 

There is availability of [ALOS Palsar ScanSAR](https://developers.google.com/earth-engine/datasets/catalog/JAXA_ALOS_PALSAR-2_Level2_2_ScanSAR#description) data in GEE over Oceania and South East Asia that we can use in combination with [Sentinel-1 data](https://developers.google.com/earth-engine/datasets/catalog/COPERNICUS_S1_GRD) for this excercise. 

The location of interest is Lat: -4.011&deg; , Lon: 141.303&deg;. First we will locate the area of interest through the point location and create a rectangle around it.    


```java
//Create variable with the point location of interest
var defPaG = ee.Geometry.Point([141.303, -4.011])

// Define a rectangular area of interest, by listing coordinates
var defaultStudyArea = ee.Geometry.Polygon(
        [[[141.2500740578722,-4.04103621945203],
          [141.3599373391222,-4.04103621945203],
          [141.3599373391222,-3.9691145182502945],
          [141.2500740578722,-3.9691145182502945]]], null, false);

Map.addLayer(defaultStudyArea, {}, 'aoi');
Map.addLayer(defPaG, {}, 'point of deforestation')

// Zoom to area of of interest
Map.centerObject(defaultStudyArea, 13);
```
Then, we will define the dates of interest:
 ```java
var firstdate = '2022-01-01'
var seconddate = '2022-12-31'
 ```
Get Collection of Sentinel-2 Optical data for time period of interest and visualize first image of the collection:
```java
var sentinel2 = ee.ImageCollection('COPERNICUS/S2_SR_HARMONIZED')
                  .filterDate(firstdate, seconddate)
                  .filterBounds(defaultStudyArea)
                  .limit(60) //limit for animation purposes
                  
print(sentinel2, 'sentinel2 collection') //Explore properties of Sentinel-2 data 

// Define visualization arguments for Optical S2 data.
var visArgsSen2 = {bands: ['B4', 'B3', 'B2'], min: 300, max: 2500};

Map.addLayer(sentinel2.first(), visArgsSen2, 'First Sentinel2 image')  
 ```
___
> ####  Question 1: 
> *What is the date of the first image available for Sentinel-2?*
___

Prepare animation for the Sentinel-2 Image Collection 
 ```java
 // Define a function to convert an image to an RGB visualization image and copy
// properties from the original image to the RGB image.
var visFun = function(img) {
  return img.visualize(visArgsSen2).copyProperties(img, img.propertyNames());
};

// Map over the image collection to convert each image to an RGB visualization
// using the previously defined visualization function.
var s2colVis = sentinel2.map(visFun);   

//Adding date to animation
var text = require('users/gena/packages:text'); // Import gena's package which allows text overlay on image

var annotations = [
  {position: 'left', offset: '1%', margin: '1%', property: 'label', scale: 40} //large scale because image if of the whole world. Use smaller scale otherwise
  ]

//Function to add text (date) to animation
function addText(image){
  
  var timeStamp =  ee.Date(image.get('system:time_start')).format().slice(0,10) // get the time stamp of each frame. This can be any string. Date, Years, Hours, etc.
  var timeStamp = ee.String('Date: ').cat(ee.String(timeStamp)); //conveee.String('Date: ').cat(ee.String(timeStamp));rt time stamp to string 
  var image = image.visualize(visArgsSen2).set({'label': timeStamp}) // set a property called label for each image
  
  var annotated = text.annotateImage(image, {}, defaultStudyArea, annotations); // create a new image with the label overlayed using gena's package

  return annotated 
}

var s2colVis2 = sentinel2.map(addText) //add time stamp to all Sentinel 2 images

// Define GIF visualization parameters.
var gifParams = {
  'region': defaultStudyArea,
  'dimensions': 700,
  'crs': 'EPSG:3857',
  'framesPerSecond': 5,
  'maxPixels': 46046000
};

//Visualize animation in Console and get URL for .gif animation
print(ui.Thumbnail(s2colVis2, gifParams)); // Sentinel-2 Optical 
print(s2colVis2.getVideoThumbURL(gifParams));
  ```

The following animation should appear in the Console:
![fig](/Figures/Sentinel2_PapNewGu1.gif)
<sub>Figure 2. Sentinel-2 Animation . </sub>
