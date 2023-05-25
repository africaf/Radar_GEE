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
Transform Alos Palsar data to dB 
```java
//Create function to tranform Alos Palsar bands HH and HV to dB
function todBall (image) {
  var dBHH =  (ee.Image((image.select('HH').pow(2)).log10()).multiply(10).subtract(83)).rename('dBHH')
  var dBHV =  (ee.Image((image.select('HV').pow(2)).log10()).multiply(10).subtract(83)).rename('dBHV')
  return image.addBands([dBHH, dBHV]).copyProperties(image, ["system:time_start"]);
}

//Apply to relevant IC
var AlosdB = collectionALOS.map(todBall)

```
Plot data in one graph
```java
//Integrated graph
// Define the chart and print it to the console.
var chart = ui.Chart.image.series({
  imageCollection: AlosdB.select('dBHH', 'dBHV'),
  region: aoi,
  reducer: ee.Reducer.mean(),
  scale: 30,
  xProperty: 'system:time_start'
})
.setSeriesNames(['HH', 'HV'])
.setOptions({
  title: 'Average dB by Date',
  hAxis: {title: 'Date', titleTextStyle: {italic: false, bold: true}},
  vAxis: {
    title: 'dB ',
    titleTextStyle: {italic: false, bold: true}
  },
  lineWidth: 4,
  colors: ['red', 'green'],
  curveType: 'function'
});
print(chart);
```
![fig](Figures/chartHHHV.png)

<sub>Figure 1. Chart of HH and HV polarization over time for aoi </sub>

Now, we will calculate the [Radar Forest Degradation Index](https://servirglobal.net/Portals/0/Documents/Articles/2019_SAR_Handbook/SAR_VegIndices_1page_new.pdf) as described in the [SAR Handbook](https://servirglobal.net/Global/Articles/Article/2674/sar-handbook-comprehensive-methodologies-for-forest-monitoring-and-biomass-estimation). 

![fig](/Figures/RFDI_0.png)
![fig](/Figures/RFDI_1.png)

<sub>Figure 2. RFDI calculation and interpretation. Source: [SAR Vegetation Indices](https://servirglobal.net/Portals/0/Documents/Articles/2019_SAR_Handbook/SAR_VegIndices_1page_new.pdf) . </sub>

```java
//Create function to transform data from dB to power
function dbToPowerALL(img){
  var powerHH = ee.Image(10).pow(img.select('dBHH').divide(10)).rename('powerHH')
  var powerHV = ee.Image(10).pow(img.select('dBHV').divide(10)).rename('powerHV')
  return img.addBands([powerHH, powerHV]);
}
//Apply to the dB Image Collection
var AlosPw = AlosdB.map(dbToPowerALL)

//Create function to calculate RFDI 
function RFDIall (image){
  var rfdi = image.normalizedDifference(['powerHH', 'powerHV']).rename('RFDI')
  return image.addBands(rfdi)
}

//Apply RFDI function to data. This will add a band 'RFDI' in each image in the Image Collection
var withRFDIall =AlosPw.map(RFDIall)

//Chart RFDI 
var chartRFDIall = ui.Chart.image.series({
  imageCollection: withRFDIall.select('RFDI'),
  region:  aoi,
  reducer: ee.Reducer.mean(),
  scale: 30
}).setOptions({
  title: 'RFDI over time',
  hAxis: {title: 'Date', titleTextStyle: {italic: false, bold: true}},
  vAxis: {
    title: 'RFDI ',
    titleTextStyle: {italic: false, bold: true}
  },
  lineWidth: 4,
  colors: ['gray'],
  curveType: 'function'
});

```
![fig](Figures/chartRFDI.png)

<sub>Figure 3. Expected RFDI graph </sub>
