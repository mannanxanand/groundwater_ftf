// Define a region of interest as a point - adjust the coordinates to your location of interest in the US.
var point = ee.Geometry.Point(-98.484246, 39.011902); // Example coordinates for Kansas
var bufferedRegion = point.buffer(10000); // Buffer by 10 km to capture surrounding fields

// Define the time range for your analysis
var startYear = 2018;
var endYear = 2021;

// Load the CDL dataset
var cdl = ee.ImageCollection("USDA/NASS/CDL")
             .filter(ee.Filter.calendarRange(startYear, endYear, 'year'))
             .select(['cropland']);

// Function to calculate the area for a given crop class (band value)
function calculateCropArea(cropClass, colorHex, label) {
  var cropArea = cdl.map(function(image) {
    return image.eq(cropClass).selfMask();
  }).sum().multiply(ee.Image.pixelArea()).divide(1e4); // Area in hectares
  
  var totalCropArea = cropArea.reduceRegion({
    reducer: ee.Reducer.sum(),
    geometry: bufferedRegion,
    scale: 30,
    maxPixels: 1e9
  }).get('cropland');
  
  totalCropArea.evaluate(function(result) {
    print(label + ' Area (hectares) - ' + colorHex + ':', result);
  });
}

// Calculate areas for Spring Wheat (class 23), Winter Wheat (class 24), and Durum Wheat
calculateCropArea(23, '#d8b56b', 'Spring Wheat');
calculateCropArea(24, '#a57000', 'Winter Wheat');
calculateCropArea(22, '#896054', 'Durum Wheat'); // Assuming the class value for Durum Wheat is 22


// Load a temperature dataset such as MODIS Land Surface Temperature
var temperature = ee.ImageCollection('MODIS/061/MOD11A2')
                   .filter(ee.Filter.date(ee.Date.fromYMD(startYear, 1, 1), ee.Date.fromYMD(endYear, 12, 31)))
                   .select('LST_Day_1km');

/// Convert temperature from Kelvin to Celsius, and calculate average temperature during the spring and winter seasons
var tempToCelsius = function(image) {
  // MODIS LST data are in Kelvin and scaled by 0.02; thus, we divide by 50, then subtract 273.15 to convert to Celsius.
  return image.multiply(0.02).subtract(273.15).set('system:time_start', image.get('system:time_start'));
};

// Modify the band name for solar radiation
var radiation = ee.ImageCollection("NASA/GLDAS/V021/NOAH/G025/T3H")
                 .filter(ee.Filter.date(ee.Date.fromYMD(startYear, 1, 1), ee.Date.fromYMD(endYear, 12, 31)))
                 .select('SWdown_f_tavg'); // Corrected band name
                 //Shortwave radiation downwards

var tempToCelsius = function(image) {
  // MODIS LST data are in Kelvin and scaled by 0.02; thus, we divide by 50, then subtract 273.15 to convert to Celsius.
  return image.multiply(0.02).subtract(273.15).set('system:time_start', image.get('system:time_start'));
};

// Calculate average daily solar radiation during the spring and winter seasons
var avgSpringRadiation = radiation.filter(ee.Filter.dayOfYear(91, 182)).mean();
var avgWinterRadiation = radiation.filter(ee.Filter.dayOfYear(274, 365)).mean();

// Reduce the region to get average radiation values
var getRadiationStats = function(image, label) {
  var stats = image.reduceRegion({
    reducer: ee.Reducer.mean(),
    geometry: point,
    scale: 30,
    maxPixels: 1e9
  });
  print(label, stats.get('SWdown_f_tavg'));
};
// ... (other code)

// Convert the temperature collection to Celsius
var temperatureCelsius = temperature.map(tempToCelsius);

// Function to create a time series chart of temperature
function createTempChart(tempCollection, seasonName) {
  var tempChart = ui.Chart.image.series({
    imageCollection: tempCollection,
    region: bufferedRegion,
    reducer: ee.Reducer.mean(),
    scale: 1000
  }).setOptions({
    title: seasonName + ' Temperature Over Time',
    vAxis: {title: 'Temperature (Celsius)'},
    hAxis: {title: 'Time'},
    lineWidth: 1,
    pointSize: 3,
    series: {0: {color: 'FF0000'}} // Color the line red for visibility
  });
  
  print(tempChart);
}

// Create temperature charts for the spring and winter seasons
var tempDuringSpring = temperatureCelsius.filter(ee.Filter.dayOfYear(91, 182)); // Spring: April to June
createTempChart(tempDuringSpring, 'Spring');

var tempDuringWinter = temperatureCelsius.filter(ee.Filter.dayOfYear(274, 365)); // Winter: October to December
createTempChart(tempDuringWinter, 'Winter');

// ... (other code)


// Print average solar radiation for spring and winter
getRadiationStats(avgSpringRadiation, 'Average Intensity of Sunlight (b/w 200-600) (W/m^2)');
getRadiationStats(avgWinterRadiation, 'Average Intensity of Sunlight (b/w 200-600) (W/m^2)');


// Define your point of interest with longitude and latitude
var pointOfInterest = ee.Geometry.Point(-98.484246, 39.011902);

// Import the SMAP soil moisture data
var dataset = ee.ImageCollection('NASA/SMAP/SPL4SMGP/007')
                .filter(ee.Filter.date('2020-01-01', '2020-01-10'));

// Select the surface and root zone soil moisture data
var surfaceSoilMoisture = dataset.select('sm_surface');
var rootZoneSoilMoisture = dataset.select('sm_rootzone');

// Get the mean soil moisture over the specified period for both layers
var meanSurfaceSoilMoisture = surfaceSoilMoisture.mean();
var meanRootZoneSoilMoisture = rootZoneSoilMoisture.mean();

// Add the soil moisture layers to the map with different color palettes for visualization
Map.addLayer(meanSurfaceSoilMoisture, {min: 0, max: 0.5, palette: ['blue', 'green', 'yellow', 'red']}, 'Mean Surface Soil Moisture');
Map.addLayer(meanRootZoneSoilMoisture, {min: 0, max: 0.5, palette: ['lightblue', 'lightgreen', 'lightyellow', 'pink']}, 'Mean Root Zone Soil Moisture');
Map.centerObject(pointOfInterest, 10);

// Print the value of soil moisture at the point of interest for both layers
var surfaceSoilMoistureValue = meanSurfaceSoilMoisture.reduceRegion({
  reducer: ee.Reducer.mean(),
  geometry: pointOfInterest,
  scale: 10000
});
var rootZoneSoilMoistureValue = meanRootZoneSoilMoisture.reduceRegion({
  reducer: ee.Reducer.mean(),
  geometry: pointOfInterest,
  scale: 10000
});

print('Surface Soil Moisture Value (between 0.2-0.3):', surfaceSoilMoistureValue);
print('Root Zone Soil Moisture Value (between 0.2-0.3):', rootZoneSoilMoistureValue);
