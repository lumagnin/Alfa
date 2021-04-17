
## Exploratory analysis of classification strategies for the realization of a land use and land cover map oriented to the determination of  built-up/constructed/developed area. 

### Authors: 
Celina Vionnet, Daniel Castellano,  Lucía Magnin, Mónica Pascual.
### Key words: 
building/construction; Landsat; GRASS GIS; pixel based classification; OBIA.

### Introduction 
This article explores two strategies for the identification of pixels corresponding to built-up area in multispectral images of medium spatial resolution. Pixel-based classifiers and object-oriented satellite image analysis have been widely used and compared for the determination of urban features in medium-resolution multispectral images. As discussed in several investigations (i.e. Tadesse et al. 2012; Waqar et al. 2012), the classification of urban features presents difficulties due to the variability of the materials used, the diversity of spectral responses encountered, and the similarity of these responses to natural features such as rock outcrops and bare soil. Although there are numerous proposals on how to approach this issue / problem, the procedures cannot be extrapolated to other areas without a prior evaluation [d] [e] of the performance obtained. This tutorial presents a workflow created to compare, with GRASS GIS, the performance of two different classification strategies[f][g] of medium resolution multispectral imagery, for the determination of built-up area within the case study (Figure 1). The classification techniques used were unsupervised pixel-based classification by classifying with four classes and subsequently reclassifying to two classes corresponding to "Built-up", "Non built-up"; and supervised object-based image analysis (OBIA) through Machine Learning.

![Figure 1. Steps followed to achieve the goals proposed in GRASS GIS.](https://github.com/dcstlln/Alfa/blob/RGrass/WorkFlow.jpg)

### 1. Input data (ingredients)
The data used for this analysis were:

|description | filename|preview|
|---------------------------------------------------|----------------------------|-------------------------|
|Basin or mask corresponding to the area (0-1 raster data) |cuenca-lago.tif| |
|Digital elevation model (DEM-raster data) of the area |DemCcaFillSink.tif| |
|Landsat image from 02_15_2020 |LC08_L1TP_229082_20200215_20200225_01_T1.tar.gz | |
|Ground truth (Raster data) containing classified area |VerdCampoEdificados2020.tif|
|Ground truth (Points shape type data) containing 150 training points randomly sampled and exported as a vector |VCEdPoli.shp| |

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
r.mapcalc "Máscara = MascaraNubes * cuenca" 
r.null map = Mascara setnull=0,0.0 
```

### 6. Conditioning the landsat image
Moving from digital numbers to surface reflectance and brightness temperature
```
i.landsat.toar input=LC08_L1TP_229082_20200215_20200225_01_T1_B output=LC08_L1TP_229082_20200215_20200225_01_T1_c_B \
sensor=oli8 metfile=$HOME/grassgis/LC08_L1TP_229082_20200215_20200225_01_T1_MTL.txt  method=dos1
```
correcting topographic shading (only 30m optical, excluding cirrus)
Obtain position solar variables from the metadata file (MTL file, zenith and azimuth values "zenith angle = 90° - elevation").
Z=37.68999506 #Z=90.0-52.31000494
```
Lista=`g.list rast pattern='*_B*', exclude='*_B11,*_B10,*_B8,*_B9' sep=comma`
i.group group=L8 subgroup=L8 input=$Lista
Z=37.68999506 
A=67.86241430
i.topo.corr -i base=dem zenith=$Z azimuth=$A output=L8.ilu
i.topo.corr base=L8.ilu input=$Lista output=t_ zenith=$Z method=percent #correct topography
```

### 7. Generation of features and additional characteristics derived from Landsat spectral information.
Fill in null values in all bands _(cycle for C syntax only works on Linux investigte sintax for Windows)_.
```
for (( i=1; i<=7; i++ )); do r.fillnulls input=t_.LC08_L1TP_229082_20200215_20200225_01_T1_c_B$i output=L8_ctf$i;done
```


```
```


```
```
 
