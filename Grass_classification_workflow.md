# -Título
`i.landsat = 1 #lo toma como códigoi

```
<style>
    body {
        background-color: #ABBAEA;
    }
    div {
        height: 200px;
        margin: 20px;
        border: 5px solid;
        background-color: #FBD603;
    }
</style>```
```

# -Autores
# -Palabras claves:
# -Introducción
  Los clasificadores basados en píxel y los basados en clasificación orientada a objetos se han utilizado y comparado ambos para la determinación de elementos urbanos en imágenes multiespectrales de resolución media...bla bla

(Algo de chachara)... 
Diversos investigadores abordan la clasificación de áreas urbanas y el etiquetado de pı́xeles corres-
pondientes a superficies urbanizadas, buscando mejorar la exactitud de la clasificación, comparando
algoritmos en distintas entornos y ambientes. Estos esfuerzos se deben principalmente a la dificultad que presenta clasificar elementos urbanos debida a la variabilidad de los materiales empleados, a la diversidad de respuestas espectrales encontradas y a la similaridad que presentan estas respuestas con elementos naturales como afloraciones rocosas y suelo desnudo.

  Este artículo presenta el flujo de tareas necesarias para comparar en grass, desde cero, los desempeños de dos estrategias distintas de clasificación temática de imágenes multiespectrales de resolución media, para la determinación de espacio edificado.  Las técnicas de clasificación utilizadas

## En este artículo se implementan en Grass los siguientes pasos:

    1. Cargaa y acondicionamiento de los datos
        ◦ re-proyección
        ◦ calibración radiométrica
        ◦ corrección atmosférica
        ◦ corrección de sombreado topográfico
    2. Visualización datos originales
        ◦ corrección de colores
        ◦ composición de las imágenes
    3. Clasificación de los datos
        ◦ Clasificación no supervizada
        ◦ Clasificación supervizada por OBIA
        ◦ Reclasificación de los datos
    4. Visualización mapas temáticos
    5. Medición de la veracidad de clasificación
       
		
 # -Descripción / Link a las imágenes y recursos utilizados
	-Cuenca o máscara de la zona
	-Dem de la zona
	-Imagen Landsat XXX
	-Verdad de campo (Shape Puntos) conteniendo puntos de entrenamiento
	-Verdad de campo (Raster) conteniendo zona clasificada 

# -Desarrollo
<div id="foo"
  class="bar">
</div>

    • lineas de código utilizadas
    • imágenes antes y después cuando corresponda
    • tablas de resultados o resultados principales sacados de la estadística
    • No incluir lineas que no sean importantes (sacar toda la estadística superflua)

# -Conclusiones

# -Lista de ayudas  de grass a  librerías y citas a artículos complementarios

1: W. Tadesse, T. Coleman, and T. Tsegaye, “Improvement of land use and land cover classification of an urban area using image segmentation from landsat etm + data,” 10 2011.

2: J. Mas, J. Dı́az-Gallegos, and A. Vega, “Evaluación de la confiabilidad temática de mapas o de imágenes clasificadas: una revisión,” Investigaciones geográficas, pp. 53–72, 08 2003.

3: D. Riaño, E. Chuvieco, J. Salas, and I. Aguado, “Assessment of different topographic corrections in landsat-tm data for mapping vegetation types (2003),” Geoscience and Remote Sensing, IEEE Transactions on, vol. 41, pp. 1056 – 1061, 06 2003.

4: P. Chavez, Jr, “An improved dark-object subtraction technique for atmospheric scattering co-rrection of multispectral data,” Remote Sensing of Environment, vol. 24, pp. 459–479, 04 1988.

5: A. Felicisimo, Modelos Digitales del Terreno. Introducción y Aplicaciones en las Ciencias Am-bientales. Pentalfa, 1994.

6: V. Henrich, “Index DataBase. https://www.indexdatabase.de, 2012.

7: G. Chander and B. Markham, “Summary of current radiometric calibration coefficients for landsat mss, tm, etm+, and eo-1 ali sensors,” Remote Sensing of Environment, vol. 113, pp. 893– 903, 05 2009.

8: D. Riaño, E. Chuvieco, J. Salas, and I. Aguado, “Assessment of different topographic corrections in landsat-tm data for mapping vegetation types (2003),” Geoscience and Remote Sensing, IEEE Transactions on, vol. 41, pp. 1056 – 1061, 06 2003.
