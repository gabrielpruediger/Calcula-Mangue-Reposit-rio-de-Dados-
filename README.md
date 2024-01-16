// Definir a Área de Estudo
var geometry = ee.Geometry({
  'type': 'Polygon',
  'coordinates':
    [[[-43.550, -23.000],
      [-43.650, -23.000],
      [-43.650, -23.050],
      [-43.550, -23.050]]]
});
var geometryCoordinates = geometry.coordinates();

print('geometry.coordinates(...) =', geometryCoordinates);

Map.setCenter(-43.650, -23.000, 15);
Map.addLayer(geometry,
             {'color': 'black'},
             'Geometry [black]: geometry');
             

//Definir o Satélite em uso
var image = ee.ImageCollection("COPERNICUS/S2_SR")
.filter(ee.Filter.lt('CLOUDY_PIXEL_PERCENTAGE', 30))
.filterDate("2022-01-01", "2023-07-29") 
.select('B.*')
//Encontrar a média nesse ano com .median()
.filterBounds(geometry).median().clip(geometry);


var image_map = {
  "bands":["B8","B4", "B3"],
  "min": 600,
  "max": 3000,
  "gamma": 1,
};

Map.addLayer(image,image_map,'Image');

var controlPoints = agua.merge(vegetacaotenue).merge(vegetacaodensa).merge(areasurbanas);

var training = image.sampleRegions({
  collection: controlPoints,
  properties: ['landcover'],
  scale: 10,
  tileScale: 16
  });
  

var classifier = ee.Classifier.smileRandomForest(50).train({
    features: training,
    classProperty: 'landcover'
  });
  
var classified = image.classify(classifier);
  
  Map.addLayer(classified, {min:0, max:3, palette:['blue','yellow','green','red',]}, 'MANGUE');

var varagua = classified.eq(0);

  Map.addLayer(varagua,{min:0, max:1, palette:['grey', 'blue']}, 'Corpos dágua');
  
var varvegetacaotenue = classified.eq(1);

  Map.addLayer(varvegetacaotenue,{min:0, max:1, palette:['grey', 'yellow']}, 'Vegetação Tênue');

var varvegetacaodensa = classified.eq(2);

  Map.addLayer(varvegetacaodensa,{min:0, max:1, palette:['grey', 'green']}, 'Vegetação Densa');

var varareasurbanas = classified.eq(3);

  Map.addLayer(varareasurbanas,{min:0, max:1, palette:['grey', 'red']}, 'Áreas Urbanas');

//Alternativa para calcular a área sem separar as classes
var areaImage = varvegetacaodensa.multiply(ee.Image.pixelArea());
var area = areaImage.reduceRegion({
  reducer: ee.Reducer.sum(),
  geometry: classified.geometry(),
  scale: 500,
  maxPixels: 1e10
  });
print(densaAreaSqKm); 

//Cálculo da área referente a classe Corpos d'água
var corposdagua = classified.select('classification').eq(0);
var area_corposdagua = corposdagua.multiply(ee.Image.pixelArea()).divide(1000*1000);
var stat_corposdagua = area_corposdagua.reduceRegion ({
  reducer: ee.Reducer.sum(),
  geometry: classified.geometry(),
  scale: 30,
  maxPixels: 1e9
});

print(stat_corposdagua, 'Área ocupada pela água');

//Cálculo da área referente a classe Vegetação Tênue
var tenue = classified.select('classification').eq(1);
var area_tenue = tenue.multiply(ee.Image.pixelArea()).divide(1000*1000);
var stat_tenue = area_tenue.reduceRegion ({
  reducer: ee.Reducer.sum(),
  geometry: classified.geometry(),
  scale: 30,
  maxPixels: 1e9
});

print(stat_tenue, 'Área da Vegetação Tênue');

//Cálculo da área referente a classe Vegetação Densa
var densa = classified.select('classification').eq(2);
var area_densa = densa.multiply(ee.Image.pixelArea()).divide(1000*1000);
var stat_densa = area_densa.reduceRegion ({
  reducer: ee.Reducer.sum(),
  geometry: classified.geometry(),
  scale: 30,
  maxPixels: 1e9
});

print(stat_densa, 'Área da Vegetação Densa');

var ubana = classified.select('classification').eq(3);
var area_ubana = ubana.multiply(ee.Image.pixelArea()).divide(1000*1000);
var stat_ubana = area_ubana.reduceRegion ({
  reducer: ee.Reducer.sum(),
  geometry: classified.geometry(),
  scale: 30,
  maxPixels: 1e9
});

print(stat_ubana, 'Área Urbana');
