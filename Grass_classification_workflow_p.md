## Exploratory analysis of classification strategies for the realization of a land use and land cover map oriented to the determination of built-up areas 

### Authors: 
Celina Vionnet (1), Daniel Castellano (2),  Lucía Magnin (3), Mónica Pascual (4).

(1):Est. Ciencias Geológicas, Facultad de Ciencias Exactas, Físicas y Naturales - UNC. (2):CEPROCOR Centro de Excelencia en Productos y Procesos Córdoba. MinCyT Cba. (3):División Arqueología, Facultad de Cs. Naturales y Museo, UNLP - CONICET. (4):Departamento Planes Hidrológicos dentro de la Dirección de Planificación, Autoridad del Agua de la Provincia de Buenos Aires.

### Key words: 
building/construction; Landsat; GRASS GIS; pixel based classification; OBIA.

### Introduction 
This article explores two strategies for the identification of pixels corresponding to built-up areas in multispectral images of medium spatial resolution. Pixel-based classifiers and object-oriented satellite image analysis have been widely used and compared for the determination of urban features in medium-resolution multispectral images. As discussed in several investigations (i.e. Tadesse et al. 2012; Waqar et al. 2012), the classification of urban features presents difficulties due to the variability of the materials used, the diversity of spectral responses encountered, and the similarity of these responses to natural features such as rock outcrops and bare soil. Although there are numerous proposals on how to approach this issue / problem, the procedures cannot be extrapolated to other areas without a prior evaluation of the performance obtained. This tutorial presents a workflow created to compare, with GRASS GIS, the performance of two different classification strategies of medium resolution multispectral imagery, for the determination of built-up area within the case study (Figure 1). The classification techniques used were unsupervised pixel-based classification by classifying with four classes and subsequently reclassifying to two classes corresponding to "Built-up", "Non built-up"; and supervised object-based image analysis (OBIA) through Machine Learning.

![Figure 1. Steps followed to achieve the goals proposed in GRASS GIS.](https://github.com/dcstlln/Alfa/blob/RGrass/WorkFlow.jpg)

### 1. Input data (ingredients)
The data used for this analysis were:

|description | filename|preview|
|---------------------------------------------------|----------------------------|-------------------------|
|Basin or mask corresponding to the area (0-1 raster data) |cuenca-lago.tif|  ![see it](https://github.com/dcstlln/Alfa/blob/RGrass/Cuenca-Lago.jpg)|
|Digital elevation model (DEM-raster data) of the area |DemCcaFillSink.tif| ![see it](https://github.com/dcstlln/Alfa/blob/RGrass/DemCCa.jpg)|
|Landsat image from 02_15_2020 |LC08_L1TP_229082_20200215_20200225_01_T1.tar.gz | ![see it](https://github.com/dcstlln/Alfa/blob/RGrass/L8.jpg)|
|Ground truth (Raster data) containing classified area |VerdCampoEdificados2020.tif|![see it](https://github.com/dcstlln/Alfa/blob/RGrass/VerdDeCampo.jpg)|
|Ground truth (Points shape type data) containing 150 training points randomly sampled and exported as a vector |VCEdPoli.shp|![see it](https://github.com/dcstlln/Alfa/blob/RGrass/VCEdPoli.jpg) |

### 2. Configuring working directories and starting Grass
```
grass78 -c CuencaLago.tif $HOME/grassdata/mylocation
g.mapset -c mapset=CcaSanRoque
grass78 $HOME/grassdata/CCaSanRoque/PERMANENT 
```

### 3. Loading libraries
```
g.extension extension=i.landsat
g.extension i.superpixels.slic
g.extension r.neighborhoodmatrix
g.extension i.segment.uspo
g.extension i.segment.stats
g.extension r.object.geometry
g.extension r.sample.category
g.extension v.class.mlR
g.extension d.colors
```

### 4. Loading data
```
r.import input=$HOME/grassgis/cuenca-lago.tif output=cuenca
r.import input=$HOME/grassgis/DemCcaIGNFillSink.tif output=dem
r.import input=$HOME/grassgis/VerdCampoEdificados2020.tif output=VCEd
v.import input=$HOME/grassgis/VCEdPoli.shp output=VCEdPoli
i.landsat.import -p input=$HOME/grassgis/landsat_data pattern=’B(1|2|3|4|5|6|7|9)’
```

### 5. Initial preprocessing 
Set working area and projection to the watershed.
```
g.proj -c georef=$HOME/grassgis/
g.region raster=cuenca
```
Masking: remove clouds, invalid pixels and restricting output to the watershed by combining the three elements in a mask.
```
i.landsat.qa collection=1 cloud_shadow_confidence="Medium,High" cloud_confidence="Medium,High" output=ReglasNubosas.txt
r.reclass input=LC08_L1TP_229082_20200215_20200225_01_T1_BQA output=MascaraNubes rules=ReglasNubosas.txt
r.mapcalc "Mascara = MascaraNubes * cuenca" 
r.null map = Mascara setnull=0,0.0 
```
->The output maps for this code block are: 

|Initial cloud mask                                           | Final work mask                                            | 
|-------------------------------------------------------------|------------------------------------------------------------|
|![initial mask=_MascaraNubes_](https://github.com/dcstlln/Alfa/blob/RGrass/masknubes.jpg)| ![final mask=_Mascara_](https://github.com/dcstlln/Alfa/blob/RGrass/mask.jpg)|

### 6. Conditioning the Landsat image
Moving from digital numbers to surface reflectance and brightness temperature
```
i.landsat.toar input=LC08_L1TP_229082_20200215_20200225_01_T1_B output=LC08_L1TP_229082_20200215_20200225_01_T1_c_B \
sensor=oli8 metfile=$HOME/grassgis/LC08_L1TP_229082_20200215_20200225_01_T1_MTL.txt  method=dos1
```
-->it generate a number of useful maps named: _LC08_L1TP_229082_20200215_20200225_01_T1_c_B[N], where [N] is 1|2|3|4|5|6|7|9.

Correcting topographic shading (only 30m optical, excluding cirrus)
Obtain position solar variables from the metadata file ([MTL file](https://github.com/dcstlln/Alfa/blob/RGrass/LC08_L1TP_229082_20200215_20200225_01_T1_MTL.txt), zenith and azimuth values "zenith angle = 90° - elevation").
Z=37.68999506 #Z=90.0-52.31000494
```
Lista=`g.list rast pattern='*_B*', exclude='*_B11,*_B10,*_B8,*_B9' sep=comma`
i.group group=L8 subgroup=L8 input=$Lista
Z=37.68999506 
A=67.86241430
i.topo.corr -i base=dem zenith=$Z azimuth=$A output=L8.ilu
i.topo.corr base=L8.ilu input=$Lista output=t_ zenith=$Z method=percent #correct topography
```
This block generate an iluminatin model (shading) and uses it to correct shadows in Landsat bands. See _l8.ilu_ [here](https://github.com/dcstlln/Alfa/blob/RGrass/L8ilu.jpg)

-->it generate a number of useful maps named: t_LC08_L1TP_229082_20200215_20200225_01_T1_c_B[N], where [N] is 1|2|3|4|5|6|7|9.

#### Here you can see a comparison between the RGB composition of the original bands in digital numbers and the reflectance of the surface with topographic correction: 
![Landsat 8: comparison before and after corrections calibrations](https://github.com/dcstlln/Alfa/blob/RGrass/CompNDsRad.jpg)


### 7. Generation of features and additional characteristics derived from Landsat spectral information
Fill in null values in all bands _(cycle for C syntax only works on Linux investigte sintax for Windows)_.
```
for (( i=1; i<=7; i++ )); do \
    r.fillnulls input=t_.LC08_L1TP_229082_20200215_20200225_01_T1_c_B$i output=L8_ctf$i;\
    done
```
-->it generate a number of useful maps named: L8_ctf[N], where [N] is 1|2|3|4|5|6|7|9

Generating panchromatic band and compute textures by IDM (1/enthropy) and ASM (1/contrast) methods.
```
r.mapcalc "Pancro=(L8_ctf1+L8_ctf2+L8_ctf3+L8_ctf4)/4" 
r.colors map=Pancro -e color=grey #Color table application for panchromatic band visualization.
r.texture input=Pancro output=Texturas size=5 distance=1 method=idm,asm  
```
-->it generate three useful maps:

|_Pancro_| _Texturas_ASM_| _Texturas_ID|
|--------------------------------------|--------------------------------------|--------------------------------------|
|![_Pancro_](https://github.com/dcstlln/Alfa/blob/RGrass/Pancro.jpg)|![_Texturas_ASM_](https://github.com/dcstlln/Alfa/blob/RGrass/TextIDM.jpg)| ![_Texturas_IDM_](https://github.com/dcstlln/Alfa/blob/RGrass/TextASM.jpg)|

Generating spectral indexes: NDBI (normalized difference built up index) and SAVI (soil adjusted vegetation index) with  r.mapcalc y i.vi.
Reclassify odd values above 1 and below -1.
```
r.mapcalc "L8_NDBI=(L8_ctf5-L8_ctf4)/(L8_ctf5+L8_ctf4)" #NDBI  = (OLI6 – OLI5) / (OLI6 + OLI5)
i.vi red=L8_ctf4 nir=L8_ctf5 viname=savi output=L8_SAVI 
r.mapcalc "L8_NDBIc=if(L8_NDBI>1,null(),L8_NDBI)" --overwrite
r.mapcalc "L8_NDBIc=if(L8_NDBI<-1,null(),L8_NDBIc)" --overwrite
r.mapcalc "L8_SAVIc=if(L8_SAVI>1,null(),L8_SAVI)" --overwrite
r.mapcalc "L8_SAVIc=if(L8_SAVI<-1,null(),L8_SAVIc)" --overwrite
```
-->it generate useful two maps:

|_L8\_NDBIc_|_L8\_SAVIc_|
|-----------------------------|--------------------------------|
|![_L8\_NDBIc_](https://github.com/dcstlln/Alfa/blob/RGrass/NDBI.jpg)| ![_L8\_SAVIc_](https://github.com/dcstlln/Alfa/blob/RGrass/SAVI.jpg)|

### 8. Classification and visualization of results
#### Conducting an unsupervised pixel-based classification using 11 initial classes 

The number of initial thematic classes were arbitrarily selected to broadly include the four classes distinguishable to the naked eye: bare soil, sparsely vegetated soil, dense vegetated soil, built-up areas. The classification obtained is reclassified to give two classes with value 0 and 1 corresponding to built, not built.

Grouping the working data.
```
ListaClasif=`g.list rast pattern='L8_ctf*,*SAVIc*,*NDBIc*' sep=comma`
i.group group=L8o subgroup=L8o input=$ListaClasif 
```
Generating spectral signatures for classifier, classifying and reclassifying classes *3 to 11->0* and *1,2->1* (0=no eddificated, 1=eddificated), based on knowledge of the study area 
```
i.cluster group=L8o subgroup=L8o sig=L8_11clusters classes=11 separation=0.7 --overwrite
i.maxlik group=L8o subgroup=L8o sig=L8_11clusters output=Class_L8_11clusters rej=Rechazo_L8_11Clusters --overwrite
r.reclass input=Class_L8_11clusters output=Class_L8_11clusters_recl rules=$HOME/grassgis/reclass11clases --overwrite
r.category Class_L8_11clusters_recl
r.colors map=Class_L8_11clusters_recl rules=$HOME/grassgis/PaletaUrbanaRASTER
```
_You can Get file/classification information typing:_ *r.info map=Class\_L8\_11clusters* and *r.category Class_L8_11clusters_recl*

The reclassification rules can be seen here: [Rules](https://github.com/dcstlln/Alfa/blob/RGrass/reclass11clases)

The color rules applied can be seen here: [PaletaUrbanaRASTER](https://github.com/dcstlln/Alfa/blob/RGrass/PaletaUrbanaRASTER)

--> The output: 

|original classification | reclassified one|
|----------------------------------------------------|----------------------------------------------------|
|![_L8\_11clusters_](https://github.com/dcstlln/Alfa/blob/RGrass/class11clu.jpg)|![_L8\_11clusters\_recl_](https://github.com/dcstlln/Alfa/blob/RGrass/class11clurecla.jpg)|

#### Perform supervised object-based classification (using 2 classes with training points)
Generating region for algorithms and seeds for speed up classification process. Determining optimal classification parameters with USPO.
```
g.region -p save=regionOBIA
i.superpixels.slic input=L8o output=L8sp step=2 compactness=0.7 memory=2000
i.segment.uspo group=L8o output=L8_uspo.csv seeds=L8sp segment_map=segs region=regionOBIA \
threshold_start=0.005 threshold_stop=0.05 threshold_step=0.005 minsizes=3 number_best=5 memory=2000 processes=4 
```
--> The output contains a set of information represented in data structures and maps.

The data structure containing ranked optimized params looks like:

|Region |Thresh |Minsize |Optimization|
|---------|------|---|----------------|
|regionOBIA |0.015 |3 |1.1359822904731627|
|...|...|...|...|
|regionOBIA |0.025 |3 |1.0872001515196168|

The output maps are of type raster containing grouped pixels, as in the example below:

| _segs\_regionOBIA\_rank1_|_segs\_regionOBIA\_rank2_|
|-----------------------------|----------------------------|
|![_segs\_regionOBIA\_rank1_](https://github.com/dcstlln/Alfa/blob/RGrass/So1.png)|![_segs\_regionOBIA\_rank2_](https://github.com/dcstlln/Alfa/blob/RGrass/So2.png)|

Converting rank1 to vector for visualization and making stistics for segments
```
r.to.vect -tv input=segs_regionOBIA_rank1 output=segs_rank1 type=area
i.segment.stats map=segs_regionOBIA_rank1 rasters=$ListaOBIA raster_statistics=mean,stddev \
area_measures=area,perimeter,compact_circle,compact_square vectormap=segs_stats processes=4
```
Selection of segment points for training from segmentation and field truth. Making spatial database.
```
v.select ainput=segs_stats binput=VCEdPoli output=train_segments operator=overlap
v.info train_segments
v.db.addcolumn train_segments column="class"
v.distance from=train_segments to=VCEdPoli upload=to_attr column=class to_column=Edificado
db.select sql="SELECT class,COUNT(cat) as count_class FROM train_segments GROUP BY class"
```
--> The output vector training map its called _train_segments_

_The color pallette can by set for the training vector using_ *v.colors map=train_segments rules=$HOME/grassgis/PaletaUrbana column=class*, for better representation on Grass map viewer

Classification with machine learning (a long single line statement to call the classifier algorithm)

| Clasiffication code |   Output map|
|---------------------------------------------------------|----------------------------------------|
|![code](https://github.com/dcstlln/Alfa/blob/RGrass/classobiacode.png)|!["classification_rf"](https://github.com/dcstlln/Alfa/blob/RGrass/clasification_rf.jpg)|

### 9. VALIDATION / ASSESSMENT OF RESULTS
kappa and overall accuracy for pixel-oriented classification with 11 initial classes reclassified to 2, and OBIA validation (Machine Learning).

```
r.kappa classification=Class_L8_4clusters_recl reference=VCEd
r.kappa classification=classification_rf reference=VCEd
```
The results of both tests are shown in the following table

|Performance pixel based  |Performance OBIA based|
|-------------------------|----------------------------|
|Kappa: 0.272 |Kappa: 0.308|
|Obs Correct: 81.949|Obs Correct: 81.027|


### 10. Observations

- The first observation is that the OBIA classification procedure involves a greater number of steps and is generally more complicated compared to the unsupervised classification.
- Second: the results of the evaluation of the accuracy of the classification obtained through the kappa index (% hit rate penalized by random hits) and the value of the general hit rate (PG = observed correct) indicate that both strategies have a similar performance, not being able to determine which of them is clearly superior to the other in the conditions of the test.
The performance of the OBIA classification Obia efficiency can be influenced by spatial resolution of Landsat images (30 m), which prevents making use of the greater advantages of OBIA (greater precision in the definition of buildings, in this case) (Blaschke 2010).
As a conclusion, a greater number/wider range of configurations in the classification strategies should be explored, such as using higher spatial resolution data, using spectral indexes that improve the discrimination of classes with lower spectral separability such as rock and urban areas, e. g., incorporating the indexes proposed by Waqar et al. (2012), which in this first approximation were not used, and using a greater number of initial classes in the unsupervised classification.
 

### 11. Bibliography
- Blaschke 2010. Object Based Image Analysis for Remote Sensing. ISPRS Journal of Photogrammetry and Remote Sensing 65:2-16.
- Castellano, D. 2021. Análisis del desarrollo urbano y cambios del uso de suelo en la cuenca del embalse San Roque mediante técnicas de teledetección y su relación con aspectos geológicos-geomorfológicos. Presentado ante la Facultad de Matemática, Astronomía, Física y Computación  y el Instituto de Altos Estudios Espaciales Mario Gulich  como parte de los requerimientos para la obtención del grado de Magister en aplicaciones de Información Espacial. universidad Nacional de Córdoba, Ms.
- Chavez, Jr, P. 1988. An improved dark-object subtraction technique for atmospheric scattering correction of multispectral data, Remote Sensing of Environment, vol. 24, pp. 459–479, 04.
- Felicisimo, A. 1994.  Modelos Digitales del Terreno. Introducción y Aplicaciones en las Ciencias Ambientales. Pentalfa.
- Henrich, V. 2012. Index DataBase. https://www.indexdatabase.de.
- Chander, G. y B. Markham, 2009. Summary of current radiometric calibration coefficients for landsat mss, tm, etm+, and eo-1 ali sensors, Remote Sensing of Environment, vol. 113, pp. 893– 903, 05.
- Mas, J. Dı́az-Gallegos, J. y A. Vega. 2003. Evaluación de la confiabilidad temática de mapas o de imágenes clasificadas: una revisión, Investigaciones geográficas, pp. 53–72, 08.
- Riaño, D., Chuvieco, E., Salas,  J. e I. Aguado. 2003. Assessment of different topographic corrections in landsat-tm data for mapping vegetation types (2003), Geoscience and Remote Sensing, IEEE Transactions on, vol. 41, pp. 1056 – 1061, 06.
- Tadesse, W., Coleman, T. y T. Tsegaye. 2011. Improvement of land use and land cover classification of an urban area using image segmentation from landsat etm + data, 10.
- Waqar, Mirza & Mirza, J.F. & Mumtaz, R. & Hussain, Ejaz. (2012). Development of new indices for extraction of built-up area and bare soil from landsat. Data. 1. http://dx.doi.org/10.4172/scientificreports.136.

### 12. Grass reference pages
- https://grass.osgeo.org/grass76/manuals/d.mon.html
- https://grass.osgeo.org/grass76/manuals/g.list.html
- https://grass.osgeo.org/grass76/manuals/g.remove.html
- https://grass.osgeo.org/grass76/manuals/v.import.html
- https://grass.osgeo.org/grass78/manuals/addons/i.landsat.import.html
- https://grass.osgeo.org/grass78/manuals/addons/i.landsat.qa.html
- https://grass.osgeo.org/grass78/manuals/addons/i.segment.uspo.html
- https://grass.osgeo.org/grass78/manuals/addons/i.superpixels.slic.html
- https://grass.osgeo.org/grass78/manuals/addons/r.sample.category.html 
- https://grass.osgeo.org/grass78/manuals/d.rast.html
- https://grass.osgeo.org/grass78/manuals/g.extension.html
- https://grass.osgeo.org/grass78/manuals/g.region.html
- https://grass.osgeo.org/grass78/manuals/i.cluster.html 
- https://grass.osgeo.org/grass78/manuals/i.group.html
- https://grass.osgeo.org/grass78/manuals/i.topo.corr.html
- https://grass.osgeo.org/grass78/manuals/i.vi.html 
- https://grass.osgeo.org/grass78/manuals/r.buffer.html 
- https://grass.osgeo.org/grass78/manuals/r.colors.html
- https://grass.osgeo.org/grass78/manuals/r.import.html
- https://grass.osgeo.org/grass78/manuals/r.null.html
- https://grass.osgeo.org/grass78/manuals/r.texture.html 
- https://grass.osgeo.org/grass78/manuals/v.colors.html
- https://grass.osgeo.org/grass79/manuals/d.legend.html
- https://grass.osgeo.org/grass79/manuals/g.list.html
- https://grass.osgeo.org/grass79/manuals/g.proj.html
- https://grass.osgeo.org/grass79/manuals/i.colors.enhance.html
- https://grass.osgeo.org/grass79/manuals/i.landsat.toar.html
- https://grass.osgeo.org/grass79/manuals/r.fillnulls.html 
- https://grass.osgeo.org/grass79/manuals/r.info.html
- https://grass.osgeo.org/grass79/manuals/r.kappa.html 
- https://grass.osgeo.org/grass79/manuals/r.mapcalc.html
- https://grass.osgeo.org/grass79/manuals/r.mask.html
- https://grass.osgeo.org/grass79/manuals/r.reclass.html
- https://grass.osgeo.org/grass79/manuals/r.report.html
- https://grass.osgeo.org/grass79/manuals/r.univar.html
- https://grass.osgeo.org/grass79/manuals/v.distance.html 


```
```
