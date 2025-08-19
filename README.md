// Datos de entrada 
// Sectorización

var roi = ee.FeatureCollection("projects/learning-curse/assets/StudyArea_2024").geometry();

var roi_1 = ee.FeatureCollection("projects/learning-curse/assets/StudyArea_2024")  //Selección de zona de estudio
  .filterMetadata("Zona", "equals", "Huepetue") //Filtro de zona Huepetuhe
  .geometry();
  
var roi_2 = ee.FeatureCollection("projects/learning-curse/assets/StudyArea_2024")  //Selección de zona de estudio
  .filterMetadata("Zona", "equals", "Delta") //Filtro de Corredor
  .geometry();
  
var roi_3 = ee.FeatureCollection("projects/learning-curse/assets/StudyArea_2024")  //Selección de zona de estudio
  .filterMetadata("Zona", "equals", "Pampa") //Filtro de Delta
  .geometry();
  
var roi_4 = ee.FeatureCollection("projects/learning-curse/assets/StudyArea_2024")  //Selección de zona de estudio
  .filterMetadata("Zona", "equals", "Corredor") //Filtro de Delta
  .geometry();
  
Map.addLayer(roi, {}, "Corredor")


//-----------------------------------------------------------------------------------------------------------------------
//-----------------------------------------------------------------------------------------------------------------------
//Parametros de visualización 

var vis_paramClass = {
  min:1,
  max:3,
  palette:["#112ed6","#fffe46","#4f8400"]
}

var vis_param = {
    min: 0.13447800000000001,
    max: 0.531822,
    bands: ["B11", "B8", "B4"]
}

var visParamClass = {
  
}

var visParamIndexNdvi = {
  max:0.5,
  min:0,
  palette:["#11b74f"]
}

var visParamIndexNdwi = {
  max:0.5,
  min:0,
  palette:["#87fffc"]
}
 

//-----------------------------------------------------------------------------------------------------------------------
//-----------------------------------------------------------------------------------------------------------------------

//Para este ejemplo las imágenes satelitales ya estan identificadas, para los posteriores años
//se tendrá que hacer el filtro correspondientea la temporada seca de cada año de estudio.
//Solo para este caso se desbloquea esta linea de código


var img_sentinel2 = ee.ImageCollection("COPERNICUS/S2_SR")
.filterDate("2024-05-10", "2024-05-19")
.filterMetadata("CLOUDY_PIXEL_PERCENTAGE", "less_than", 10)
.filterBounds(roi_4)

print(img_sentinel2,"Colección de imagenes satelitales")


var img_sentinel2_ident = ee.Image("COPERNICUS/S2_SR/20240607T145731_20240607T150237_T19LCG")

Map.addLayer(img_sentinel2_ident, {}, "Imagen de identificación")



//-----------------------------------------------------------------------------------------------------------------------
//-----------------------------------------------------------------------------------------------------------------------

//Cuepros de agua perteneciente a la primera versión

var pozas_v1 = ee.FeatureCollection("projects/learning-curse/assets/MiningPonds_2022");

//-----------------------------------------------------------------------------------------------------------------------
//-----------------------------------------------------------------------------------------------------------------------
//Mosaico para el corredor
 
var img1_corredor = ee.Image("COPERNICUS/S2_SR/20240717T145731_20240717T150257_T19LCG")
  .clip(roi)
  .multiply(0.0001);

var img2_corredor = ee.Image("COPERNICUS/S2_SR/20240717T145731_20240717T150257_T19LCF") //La Pampa, Delta y Huepetuhe
  .clip(roi)
  .multiply(0.0001);
  
var img3_corredor = ee.Image("COPERNICUS/S2_SR/20240719T144739_20240719T145612_T19LDF")   //Complemento de la pampa
  .clip(roi)
  .multiply(0.0001);

var img4_corredor = ee.Image("COPERNICUS/S2_SR/20240719T144739_20240719T145612_T19LDG")
  .clip(roi)
  .multiply(0.0001);

var img5_corredor = ee.Image("COPERNICUS/S2_SR/20240717T145731_20240717T150257_T19LBF")
  .clip(roi)
  .multiply(0.0001);

var img6_corredor = ee.Image("COPERNICUS/S2_SR/20230802T145731_20230802T150413_T19LCF")
  .clip(roi_3)
  .multiply(0.0001);
  
var img7_corredor = ee.Image("COPERNICUS/S2_SR/20240717T145731_20240717T150257_T19LCF")
  .clip(roi_3)
  .multiply(0.0001);
  
// Mosaico

var img_select_corredor = ee.ImageCollection.fromImages([img1_corredor, img2_corredor, img3_corredor, img4_corredor, img5_corredor]);
var s2_mosaic_corredor_1 = img_select_corredor.mosaic().clip(roi_4).select(["B2", "B3", "B4", "B5", "B6", "B7", "B8", "B9", "B11", "B12"]);

//var img_select_pampa = ee.ImageCollection.fromImages([img6_corredor, img7_corredor])
//var img_Pampa = img_select_pampa.mosaic().clip(roi_3).select(["B2", "B3", "B4", "B5", "B6", "B7", "B8", "B9", "B11", "B12"]).multiply(0.0001);

//-----------------------------------------------------------------------------------------------------------------------
//-----------------------------------------------------------------------------------------------------------------------

//Recorte de imagen satelital de sectores

var img_huepetue = ee.Image("COPERNICUS/S2_SR/20240717T145731_20240717T150257_T19LCF")  //Imagen perteneciente a la fecha
  .clip(roi_1) //Huepetue
  .multiply(0.0001);
  
var img_delta = ee.Image("COPERNICUS/S2_SR/20240717T145731_20240717T150257_T19LCF")  //Imagen perteneciente a la fecha
  .clip(roi_2) //Delta
  .multiply(0.0001);
  
var img_Pampa = ee.Image("COPERNICUS/S2_SR/20240717T145731_20240717T150257_T19LCF")  //Imagen perteneciente a la fecha
  .clip(roi_3) //Pampa
  .multiply(0.0001);
  
//-----------------------------------------------------------------------------------------------------------------------
//-----------------------------------------------------------------------------------------------------------------------

// Resampleo de las imágenes

var proj = ee.Projection('EPSG:32719');

var img_huepetueh_resampled = img_huepetue.reproject({
  crs: proj,
  scale: 10
});

var img_delta_resampled = img_delta.reproject({
  crs: proj,
  scale: 10
});

var img_pampa_resampled = img_Pampa.reproject({
  crs: proj,
  scale: 10
});

var img_corredor_resampled = s2_mosaic_corredor_1.reproject({
  crs: proj,
  scale: 10
});

//-----------------------------------------------------------------------------------------------------------------------
//-----------------------------------------------------------------------------------------------------------------------

Map.addLayer(img_huepetueh_resampled, vis_param, "Resampleo imagen satelital zona de Huepetuhe")
Map.addLayer(img_delta_resampled, vis_param, "Resampleo imagen satelital zona de Delta")
Map.addLayer(img_pampa_resampled, vis_param, "Resampleo imagen satelital zona de Pampa")
Map.addLayer(img_corredor_resampled, vis_param, "Resampleo imagen satelital zona de Corredor")

//-----------------------------------------------------------------------------------------------------------------------
//------------------------------------------MACHINE LEARNING-------------------------------------------------------------
//-----------------------------------------------------------------------------------------------------------------------
//Clasificación de la zona huepetuhe
// Selección de bandas
var bands_huep = img_huepetueh_resampled.bandNames();

var training_Data_huep = rios_hue.merge(suelo_hue).merge(vegetacion_hue);   //Huepetuhe <------------------------------

// Muestreo
var prj = ee.Projection("EPSG:32719");
var svm_training_huep = img_huepetueh_resampled.select(bands_huep).sampleRegions({
  collection: training_Data_huep,
  properties: ["land_class"],
  scale: 10,
  projection: prj
});

// Clasicador 
var svm_huep = ee.Classifier.libsvm().train({
  features: svm_training_huep,
  classProperty: "land_class",
  inputProperties: bands_huep
});

// Imagen clasificada
var imgClass_Huep = img_huepetueh_resampled.select(bands_huep).classify(svm_huep);

//-----------------------------------------------------------------------------------------------------------------------

//Clasificación de la zona Delta
// Selección de bandas
var bands_del = img_delta_resampled.bandNames();

var training_Data_delta = rios_deltaa.merge(suelo_deltaa).merge(vegetacion_deltaa);  //Delta <------------------------------

// Muestreo
var prj = ee.Projection("EPSG:32719");
var svm_training_delta = img_delta_resampled.select(bands_del).sampleRegions({
  collection: training_Data_delta,
  properties: ["land_class"],
  scale: 10,
  projection: prj
});

// Clasicador 
var svm_delt = ee.Classifier.libsvm().train({
  features: svm_training_delta,
  classProperty: "land_class",
  inputProperties: bands_del
});

// Imagen clasificada
var imgClass_del = img_delta_resampled.select(bands_del).classify(svm_delt);

//-----------------------------------------------------------------------------------------------------------------------

//Clasificación de la zona La Pampa
// Selección de bandas
var bands_pam = img_pampa_resampled.bandNames();

var training_Data_pamp = rios_pampaa.merge(suelo_pampaa).merge(vegetacion_pampaa);  //La Pampa <------------------------------

// Muestreo
var prj = ee.Projection("EPSG:32719");
var svm_training_pam = img_pampa_resampled.select(bands_pam).sampleRegions({
  collection: training_Data_pamp,
  properties: ["land_class"],
  scale: 10,
  projection: prj
});

// Clasicador 
var svm_pam = ee.Classifier.libsvm().train({
  features: svm_training_pam,
  classProperty: "land_class",
  inputProperties: bands_pam
});

// Imagen clasificada
var imgClass_pam = img_pampa_resampled.select(bands_pam).classify(svm_pam);

//-----------------------------------------------------------------------------------------------------------------------

// Clasificación de la zona Corredor
// Selección de bandas
var bands_corredor = img_corredor_resampled.bandNames();

var training_Data_corredor = rios_corredor.merge(suelo_corredor).merge(vegetacion_corredor);  //Corredor minero <--------

// Muestreo
var prj = ee.Projection("EPSG:32719");
var svm_training_corredor = img_corredor_resampled.select(bands_corredor).sampleRegions({
  collection: training_Data_corredor,
  properties: ["land_class"],
  scale: 10,
  projection: prj
});

// Clasicador 
var svm_corredor = ee.Classifier.libsvm().train({
  features: svm_training_corredor,
  classProperty: "land_class",
  inputProperties: bands_corredor
});

// Imagen clasificada
var imgClass_corredor = img_corredor_resampled.select(bands_corredor).classify(svm_corredor);



//Visualización de datos
//Machine Learning
Map.addLayer(imgClass_Huep,vis_paramClass, "Clasificación supervisada Huepetuhe");
Map.addLayer(imgClass_del,vis_paramClass, "Clasificación supervisada Delta");
Map.addLayer(imgClass_pam,vis_paramClass, "Clasificación supervisada La Pampa");
Map.addLayer(imgClass_corredor,vis_paramClass, "Clasificación supervisada Corredor");



//-----------------------------------------------------------------------------------------------------------------------
//------------------------------------------EXPORTAR ARCHIVOS------------------------------------------------------------
//-----------------------------------------------------------------------------------------------------------------------
//Clasificación
Export.image.toDrive({
  image: imgClass_Huep,
  description: "Clasificación supervisada Huepetuhe",
  region: roi_1,
  scale: 10,
  fileFormat: "GeoTIFF",
  maxPixels: 10000e9,
  crs:"EPSG:32719"
})

Export.image.toDrive({
  image: imgClass_del,
  description: "Clasificación supervisada Delta",
  region: roi_2,
  scale: 10,
  fileFormat: "GeoTIFF",
  maxPixels: 10000e9,
  crs:"EPSG:32719"
})

Export.image.toDrive({
  image: imgClass_pam,
  description: "Clasificación supervisada Pampa",
  region: roi_3,
  scale: 10,
  fileFormat: "GeoTIFF",
  maxPixels: 10000e9,
  crs:"EPSG:32719"
})

Export.image.toDrive({
  image: imgClass_corredor,
  description: "Clasificación supervisada Corredor",
  region: roi_4,
  scale: 10,
  fileFormat: "GeoTIFF",
  maxPixels: 10000e9,
  crs:"EPSG:32719"
})

