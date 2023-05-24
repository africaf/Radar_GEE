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
First we will visualize optical data over the area of interest and the time period selected. We will use Sentinel-2 Optical data and will visualize on the Map viewer the first image available:
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

Prepare and visualize animation for the Sentinel-2 Image Collection 
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
print(s2colVis2.getVideoThumbURL(gifParams)); //URL 
  ```

The following animation should appear in the Console. In addition, a URL will be printed and you can download the .gif animation

![fig](/Figures/Sentinel2_PapNewGu1.gif)

<sub>Figure 2. Sentinel-2 Animation . </sub>

___
> ####  Question 2: 
> *Can you identify the date on which the major deforestation took place?*
___
Now, we will create similar animations using Sentinel-1 and Alos Palsar data.
Get image collections for both SAR datasets

```java
//Get Functions that will be used to process SAR datasets
var functions = require('users/africa_uah/SAREdX:00.Functions')
var todBall = functions.todBall

//Get ALOS Palsar data 
var collectionALOS =  ee.ImageCollection('JAXA/ALOS/PALSAR-2/Level2_2/ScanSAR')
  .filterBounds(defPaG)
  .filterDate(firstdate, seconddate)
  .filter(ee.Filter.eq('Polarizations', ["HH", "HV"]))

var ALOSdB = collectionALOS.map(todBall) //add dB bands for HH and HV 

print(ALOSdB, 'ALOSdB Image Collection')
var hhdB =   ee.Image( ALOSdB.select('dBHH').mean()); //calculate average value at pixel level for HH 
var hvdB =   ee.Image( ALOSdB.select('dBHV').mean()); //calculate average value at pixel level for HV

//Visualize average image for HH and HV 
Map.addLayer( hhdB, {min:-30, max:0}, "1.Mean HH ")
Map.addLayer(hvdB, {min: -30, max:0}, "2.Mean HV ")

```
___
> ####  Question 3: 
> *How many Alos Palsar images are available for the time period of interest?*
___

Get Sentinel-1 data
```java
var sentinel1 = ee.ImageCollection('COPERNICUS/S1_GRD');

//Get descending mode
var ddata = sentinel1
  // Filter to get images with VV and VH dual polarization.
  .filter(ee.Filter.listContains('transmitterReceiverPolarisation', 'VV'))
  .filter(ee.Filter.listContains('transmitterReceiverPolarisation', 'VH'))
  // Filter to get images collected in interferometric wide swath mode.
  .filter(ee.Filter.eq('instrumentMode', 'IW'))
  .filter(ee.Filter.eq('orbitProperties_pass', 'DESCENDING'))
  .filterDate(firstdate, seconddate)
  .filterBounds(defaultStudyArea);

print(ddata, 'Sentinel-1 Descending')

//Calculate average image and visualize it for VV and VH polarizations
var vv=ee.Image(ddata.select('VV').mean());
Map.addLayer(vv, {min: -30, max:0}, "3. Mean VV ")
print('vv',vv);

var vh = ee.Image(ddata.select('VH').mean());
print('vh',vh)
Map.addLayer(vh, {min: -30, max:0}, "4. Mean VH ")
```
