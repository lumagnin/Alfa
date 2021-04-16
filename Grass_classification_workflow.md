
## Exploratory analysis of classification strategies for the realization of a land use and land cover map oriented to the determination of  built-up/constructed/developed[a][b][c] area.

### Authors: 
Celina Vionnet, Daniel Castellano,  Lucía Magnin, Mónica Pascual.
### Key words: 
building/construction; Landsat; GRASS GIS; pixel based classification; OBIA.

### Introduction 
This article explores two strategies for the identification of pixels corresponding to built-up area in multispectral images of medium spatial resolution. Pixel-based classifiers and object-oriented satellite image analysis have been widely used and compared for the determination of urban features in medium-resolution multispectral images. As discussed in several investigations (i.e. Tadesse et al. 2012; Waqar et al. 2012), the classification of urban features presents difficulties due to the variability of the materials used, the diversity of spectral responses encountered, and the similarity of these responses to natural features such as rock outcrops and bare soil. Although there are numerous proposals on how to approach this issue / problem, the procedures cannot be extrapolated to other areas without a prior evaluation [d] [e] of the performance obtained. This tutorial presents a workflow created to compare, with GRASS GIS, the performance of two different classification strategies[f][g] of medium resolution multispectral imagery, for the determination of built-up area within the case study (Figure 1). The classification techniques used were unsupervised pixel-based classification by classifying with four classes and subsequently reclassifying to two classes corresponding to "Built-up", "Non built-up"; and supervised object-based image analysis (OBIA) through Machine Learning.

![Figure 1. Steps followed to achieve the goals proposed in GRASS GIS.](https://github.com/dcstlln/Alfa/blob/RGrass/WorkFlow.jpg)

### 1. Data input (ingredients)
1 DATA PRE-PROCESSING
Initial data / Links to the images and other resources
    -Basin or mask corresponding to the area cuenca-lago-tif
    -Digital elevation model (DEM) of the area DemCcaFillSink.tif
    -Landsat image from 02_15_2020 LC08_L1TP_229082_20200215_20200225_01_T1.tar.gz 
    -Ground truth (Raster) containing classified area VerdCampoEdificados2020.tif ()
-Ground truth (Shape Points) containing 150 training points randomly sampled and exported as a vector VCEdPoli.shp
