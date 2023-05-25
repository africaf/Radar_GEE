## Exercise 2: Plotting Sentinel-1 and Alos Palsar Time series data over a deforestation event

We will keep working in the same deforestation spot, Lat: -4.011&deg; , Lon: 141.303&deg;. 

We will create a small polygon around the point of interest.
```java
var defPaG = ee.Geometry.Point([141.303, -4.011])

//create buffer around point of interest - circle of about 10Ha
var aoi = defPaG.buffer(178.4146)

Map.addLayer(aoi, {}, 'aoi')
// Zoom to area of of interest
Map.centerObject(aoi, 13);
```
Define dates of interest
```java
var firstdate = '2022-01-01'
var seconddate = '2022-12-31'
 ```

Then, we will call and filter by date, location and available polarizations Image Collections of  [Alos Palsar ScanSAR](https://developers.google.com/earth-engine/datasets/catalog/JAXA_ALOS_PALSAR-2_Level2_2_ScanSAR) and [Sentinel-1 GRD](https://developers.google.com/earth-engine/datasets/catalog/COPERNICUS_S1_GRD) data. 
```java
//Alos Palsar data
var collectionALOS =  ee.ImageCollection('JAXA/ALOS/PALSAR-2/Level2_2/ScanSAR')
  .filterBounds(defPaG)
  .filterDate(firstdate, lastdate)
  .filter(ee.Filter.eq('Polarizations', ["HH", "HV"]))
  
//Create collections by polarizations
var collectionHH = collectionALOS.select('HH'); 
var collectionHV = collectionALOS.select('HV');

//Sentinel-1 data

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
  .filterBounds(defPaG);

print(ddata, 'Sentinel-1 Descending')

//Create collections by polarizations
var collectionVV = ddata.select('VV')
var collectionVH = ddata.select('VH')
```
