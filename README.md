# ricepaddymapping with multi-temporal data and CNN-RF Hybrid Method
Read Me:
Rice Paddy Mapping with CNN-RF Hybrid
1. pre-processing your satellite image data through GEE Platform

  example code for pre-processing satellite image via GEE
  
//select one date image sentinel-2A
var single = ee.Image('COPERNICUS/S2_SR/20220630T024529_20220630T030927_T48MZU')
print('isi sentinel 2: ',single)
function maskS2clouds(single) {
  var qa = single.select('QA60');
  var cloudBitMask = 1 << 10;
  var cirrusBitMask = 1 << 11;
  var mask = qa.bitwiseAnd(cloudBitMask).eq(0)
      .and(qa.bitwiseAnd(cirrusBitMask).eq(0));
  return single.updateMask(mask).divide(1);
}  

var S2A = ee.ImageCollection('COPERNICUS/S2_SR')
                   .filterDate('2022-01-01', '2022-12-31')
                .filter(ee.Filter.lt('CLOUDY_PIXEL_PERCENTAGE', 5))
                .map(maskS2clouds)
                .median()
                .clip(indramayu);
//print(S2A);
//variabel visualisasi 
var RGBTrue = single.select(['B4', 'B3', 'B2']);
var RGBparam = {min: 0, max: 3000,};
var greyscale = {min: 0, max: 30000, palette: ['black', 'white']};
var visRGB = {bands: ['B4', 'B3', 'B2'], max: 3000, gamma: 0.5};

//raw spectral band
var blue = {bands: ['B2'], max: 3000, gamma: 0.5};
var green = {bands: ['B3'], max: 3000, gamma: 0.5};
var red = {bands: ['B4'], max: 3000, gamma: 0.5};
var vre1 = {bands: ['B5'], max: 3000, gamma: 0.5};
var vre2 = {bands: ['B6'], max: 3000, gamma: 0.5};
var vre3 = {bands: ['B7'], max: 3000, gamma: 0.5};
var nir = {bands: ['B8'], max: 3000, gamma: 0.5};
var vre4 = {bands: ['B8A'], max: 3000, gamma: 0.5};
var swir1 = {bands: ['B11'], max: 3000, gamma: 0.5};
var swir2 = {bands: ['B12'], max: 3000, gamma: 0.5};

//spectral index features

var visParams_ndvi = {min: -0.2, max: 0.8, palette: 'FFFFFF, CE7E45, DF923D, F1B555, FCD163, 99B718, 74A901, 66A000, 529400,' +
    '3E8601, 207401, 056201, 004C00, 023B01, 012E01, 011D01, 011301'};

// Calculate NDVI
var image_ndvi = single.normalizedDifference(['B8','B4']);


var colorizedVis = {min: -0.2, max: 0.8, palette: [
    'FFFFFF', 'CE7E45', 'DF923D', 'F1B555', 'FCD163', '99B718', '74A901',
    '66A000', '529400', '3E8601', '207401', '056201', '004C00', '023B01',
  '012E01', '011D01', '011301']};
Map.addLayer(image_ndvi, colorizedVis, 'NDVI Sentinel2')

// evi
var evi = single.expression(
    '2.5 * ((NIR - RED) / (NIR + 6 * RED - 7.5 * BLUE + 1))', {
      'NIR': single.select('B8'),
      'RED': single.select('B4'),
      'BLUE': single.select('B2')
}).rename('EVI');

Map.setCenter(6.746, 46.529, 6);
Map.addLayer(evi, colorizedVis, 'EVI Sentinel-2');

//LSWI
var lswi = single.expression(
  '(NIR-SWIR1)/(NIR+SWIR1)',{
  'NIR':single.select('B8'),
  'SWIR1':single.select('B11')
  }).rename('LSWI');
Map.setCenter(6.746, 46.529, 6);
Map.addLayer(lswi, colorizedVis, 'LSWI Sentinel-2');

//RGVI2
var rgvi2 = single.expression(
  '1.05*((BLUE+RED)/(NIR+SWIR1+0.5))',{
    'NIR': single.select('B8'),
    'RED': single.select('B4'),
    'SWIR1': single.select('B11'),
    'BLUE': single.select('B2')
  }).rename('RGVI2');
Map.setCenter(6.746, 46.529, 6);
Map.addLayer(rgvi2, colorizedVis, 'RGVI2 Sentinel-2');

//menampilkan citra single date sentinel-2A RGB

Map.addLayer(single, red, 'red sentinel2A');
Map.addLayer(single, nir, 'nir sentinel2A');
Map.addLayer(image_ndvi,visParams_ndvi,'NDVI sentinel2A');
Map.addLayer(single, visRGB, 'RGB sentinel2A');
Map.addLayer(evi, colorizedVis, 'EVI sentinel2A');
Map.addLayer(lswi, colorizedVis, 'LSWI sentinel2A');
Map.addLayer(rgvi2, colorizedVis, 'RGVI2 sentinel2A');
//resampling band
//var resampled = single.resample();
//var ndvi_resampled = ndvi.resample();

var VRE1 = single.select('B5');
var VRE2 = single.select('B6');
var VRE3 = single.select('B7');
var VRE4 = single.select('B8A');
var SWIR1 = single.select('B11');
var SWIR2 = single.select('B12');

var band_20m = single.select('B6','B7', 'B8A', 'B11', 'B12');
var resampled_band = band_20m.resample();
var sentinel_allbands = single.select('B2', 'B3', 'B4', 'B5', 'B6', 'B7', 'B8', 'B8A', 'B11', 'B12');
var sentinel_resampled = sentinel_allbands.resample();

var vre1_resampled = VRE1.resample();
var vre2_resampled = VRE2.resample();
var vre3_resampled = VRE3.resample();
var vre4_resampled = VRE4.resample();
var swir1_resampled = SWIR1.resample();
var swir2_resampled = SWIR2.resample();
var lswi_resampled = lswi.resample();
var rgvi2_resampled = rgvi2.resample();


//menyimpan citra ke gdrive
var exportar_single = single.visualize(visRGB);

Export.image.toDrive({
image: rgvi2_resampled,
description: 'sentinel2A_06_05_2022_rgvi',
folder: 'Sentinel',
scale: 10,
region: region u selected,
crs: 'EPSG:4326',
});

3. import all the dataset, including vegetation inices that were calculated
4. import dataset and ground_truth=pol2_class
5. build the CNN-RF hybrid model
6. train model and save
