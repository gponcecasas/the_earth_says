# the_earth_says
## ESTE ES UN PAQUETE QUE REQUIERE INSTALACION PREVIA DE EXIFTOOL
Aqui se describen las funciones base del codigo empleado para analizar imagenes flir, no obstante, la condicion del filtro de datos aun no funciona correctamente por lo cual queda a criterio de cada usuario el _como_ implementara dicho paso previo al analisis por cluster
### Extraccion y analisis de datos de una imagen FLIR
#### PAQUETES NECESARIOS
- Puedes encontrar mas info sobre ThermImage [AQUI](https://github.com/gtatters/Thermimage)

- Puedes encontrar mas info aqui sobre Cluster [AQUI](https://rpubs.com/a_siewarga/image_clustering)
```
install.packages("Thermimage", repos='http://cran.us.r-project.org')
install.packages("cluster")
install.packages("jpeg")
install.packages("imagefx")
```
##### CARGAR IMAGEN FLIR Y SU METADATA
jpg es una forma de compresión de datos usada para imagenes, flir emplea este formato para almacenar grandes volumenes de información la cual deberá ser extraida para su analisis posterior
|imagen FLIR.jpg RAW|
|:---------:|
|![flir_1](https://user-images.githubusercontent.com/68933213/124394170-32e15780-dccc-11eb-95da-25e4a4caa5ca.jpg)|
Las funciones necesarias para extraer la matriz de datos termicos en una imagen FLIR  se describen en los vectores img y cams, el primero esta encargado de obtener toda la información de la imagen y el segundo se encarga de extraer la metadata que se utilizara para las transformaciones de la matriz de datos en informacion util
###### CARGAR IMAGEN FLIR
```
img <- Thermimage::readflirJPG(k, exiftoolpath = "installed") #-readflirJPG("string", exiftoolpath="installed") 
```
###### CARGAR METADATA 
```
cams <- Thermimage::flirsettings(k ,exiftoolpath = "installed", camvals="") 
```
***Para generar una matriz que contenga los datos de temperatura correspondientes a cada pixel en la matriz JPG de la imagen FLIR se deben crear vectores que individualicen la metadata de esta imagen***

```
ObjectEmissivity<-  cams$Info$Emissivity              # Image Saved Emissivity - should be ~0.95 or 0.96
dateModif<-   cams$Dates$FileModificationDateTime     # Modification date/time extracted from file
PlanckR1<-    cams$Info$PlanckR1                      # Planck R1 constant for camera  
PlanckB<-     cams$Info$PlanckB                       # Planck B constant for camera  
PlanckF<-     cams$Info$PlanckF                       # Planck F constant for camera
PlanckO<-     cams$Info$PlanckO                       # Planck O constant for camera
PlanckR2<-    cams$Info$PlanckR2                      # Planck R2 constant for camera
ATA1<-        cams$Info$AtmosphericTransAlpha1        # Atmospheric Transmittance Alpha 1
ATA2<-        cams$Info$AtmosphericTransAlpha2        # Atmospheric Transmittance Alpha 2
ATB1<-        cams$Info$AtmosphericTransBeta1         # Atmospheric Transmittance Beta 1
ATB2<-        cams$Info$AtmosphericTransBeta2         # Atmospheric Transmittance Beta 2
ATX<-         cams$Info$AtmosphericTransX             # Atmospheric Transmittance X
OD<-          cams$Info$ObjectDistance                # object distance in metres
ReflT<-       cams$Info$ReflectedApparentTemperature  # Reflected apparent temperature
AtmosT<-      cams$Info$AtmosphericTemperature        # Atmospheric temperature
IRWinT<-      cams$Info$IRWindowTemperature           # IR Window Temperature
IRWinTran<-   cams$Info$IRWindowTransmission          # IR Window transparency
RH<-          cams$Info$RelativeHumidity              # Relative Humidity
h<-           cams$Info$RawThermalImageHeight         # sensor height (i.e. image height)
w<-           cams$Info$RawThermalImageWidth          # sensor width (i.e. image width)
```
Estos vectores serán los argumentos de la función raw2temp la cual creara la matriz con las temperaturas de la imagen
```
temperatura <- Thermimage::raw2temp(img, ObjectEmissivity, OD, ReflT, AtmosT, IRWinT, IRWinTran,
                        RH, PlanckR1, PlanckB, PlanckF, PlanckO, PlanckR2, ATA1, ATA2,
                        ATB1, ATB2, ATX)
```
***COMO ALTERNATIVA Y PARA REVISAR CADA PASO _SE PUEDE GENERAR UNA IMAGEN_ AL TRANSFORMAR LA MATRIZ EN UN PLOT CON LA SIGUIENTE FUNCIÓN***
```
Thermimage::plotTherm(temperatura, h=h, w=w, minrangeset = 0, maxrangeset = 25)
```
|Imagen flir desde metadata|
|:--------:|
|![flir_raw](https://user-images.githubusercontent.com/68933213/124394362-2b6e7e00-dccd-11eb-85e7-bcfdfde62834.jpg)|


Para filtrar es necesario definir un filtro y guardar la imagen generada para ser utilizada en pasos posteiores, como bien se explico antes la definicion de este paso es a discreción del usuario por lo que aqui solo se presentara un ejemplo

```
if(max>(percentil90+5){
  filtro = temperatura>percentil90
  jpeg(nombre2) #CREA UNA IMAGEN PNG EN EL DIRECTORIO DE TRABAJO
  Thermimage::plotTherm(filtro, h=h, w=w, minrangeset = 0, maxrangeset = 20)
  dev.off()
  print("IMAGEN FILTRADA EXITOSAMENTE")
 }else if(max<=(percentil90+5){
  filtro = temperatura
  jpeg(test2.jpg) #CREA UNA IMAGEN PNG EN EL DIRECTORIO DE TRABAJO
  Thermimage::plotTherm(filtro, h=h, w=w, minrangeset = 0, maxrangeset = 15)
  dev.off()
  print("ATENCIÓN: IMAGEN GENERADA SIN FILTRO")
 }else{
 print("ERROR")
}
```
|![img_temp](https://user-images.githubusercontent.com/68933213/124394083-bfd7e100-dccb-11eb-929b-cffc3293394f.png)|![flir1_1](https://user-images.githubusercontent.com/68933213/124402177-8f5c6b00-dcfc-11eb-8e58-7c49e65943de.jpg)|
|:-----|:------|
|Antes y despues de realizar filtro de metadata|Filtro realizado mediante condicion max>16 y filtrando por percentil90|

***CORTAR IMAGEN FILTRADA***
Para mejorar el calculo de area se recomienda recortar el plot generado en pasos anteriores, se puede realizar de varias formas y aqui se presentara 2 formas de lograr esto 
***Opción 1***
```
test <- jpeg::readJPEG("C:/Users/usuario/Desktop/data_analisis/earth_says/test_1.jpg")
flir_crop <- imagefx::crop.image(img = test) #EL CORTE SERÁ POR SELECCION MANUAL AUNQUE SE PUEDEN INGRESAR COORDENADAS
image2(flir_crop[[1]],asp=1)
jpeg("flir_crop.jpg")
image2(flir_crop[[1]],asp=1)
dev.off()
```
|![flir_crop](https://user-images.githubusercontent.com/68933213/124396165-f87cb800-dcd5-11eb-837d-0e80bf0ff6d1.jpg)|
|:-----:|
|Con esta función es posible eliminar la barra de temperaturas, no obstante, no se consigue una imagen aislada|
 ```
 flir = jpeg::readJPEG(imageFile, native = TRUE)
 output = flir(1:480, 1:480) # PIXELES DE EJEMPLO NO TIENEN REPRESENTACION EN LA IMAGEN
 writeJPEG(image = output, target = flir_recortado)
 
 ```
| ![crop](https://user-images.githubusercontent.com/68933213/124402284-4e188b00-dcfd-11eb-91db-74f77eaa1dc8.jpg)|
|:------:|
|Imagen referencial de un corte sin parametros de grafico|

Esta funcion si permite guardar una imagen sin los parametros del plot, no obstante hay que definir las coordenadas de pixeles a recortar

### ANALISIS POR CLUSTER IMAGING
#### TRANSFORMAR IMAGEN JPG EN RGB
CORRER EL CODIGO TAL CUAL, SOLO MODIFICANDO LA DIRECCION DE LA IMAGEN A USAR

Se analizaron los cluster necesarios para definir la imagen, se obtuvo que se requieren solo 2 grupos para describir el FLIR posterior al filtro como se observa en el grafico silueta versus grupos
|ANALISIS DE GRUPO DE COLOR|
|:-------:|
| ![cluster_optimos](https://user-images.githubusercontent.com/68933213/124401660-9a150100-dcf8-11eb-9553-5a07355aec41.png) |
MAS DEL 90% DE LA IMAGEN SE CONFORMA SOLO DE 2 GRUPOS DE COLORES
Aun asi en los pasos siguientes el analisis se realiza considerando 3 grupos de colores debido a la existencia del blanco en los plot obtenidos, sin embargo, si se aisla la imagen sin el fondo del grafico no se requeriria considerar un 3r cluster.

El archivo JPG debe ser descrito en los pixeles fundamentales rojo, azul y verde (RGB) y luego agrupados en grupos de colores mediante la funcion clara(), esta posee dos argumentos: El primero es el dataframe del RGB y el segundo es el numero de cluster a considerar
```
test_flir <- jpeg::readJPEG("~/imagen.jpg")
dm <- dim(test_flir) #VECTOR QUE CONTIENE LAS DIMENSIONES DE LA IMAGEN
#TRANSFORMAR LA MATRIZ JPEG A UNA RGB
rgbImage1 <- data.frame(
  x=rep(1:dm[2], each=dm[1]),
  y=rep(dm[1]:1, dm[2]),
  r.value=as.vector(test_flir[,,1]),
  g.value=as.vector(test_flir[,,2]),
  b.value=as.vector(test_flir[,,3]))
testflir = rgbImage1[, c("r.value", "g.value", "b.value")]
# Filtro por numero de cluster elegidos - clara(testflir, nro de cluster) -
clara <- cluster::clara(testflir, 3)
colours <- rgb(clara$medoids[clara$clustering, ])
dominantColours <- as.data.frame(table(colours))
```
AQUI SE DESCRIBEN 3 ESTADISTICOS PARA 3 CLUSTER y ENTRE MAS CLUSTER - MAS ESTADISTICOS SE DEBEN DEFINIR 
###### PARA 2 CLUSTER, SOLO OCUPAR MAX Y MIN
```
max_col  <- max(dominantColours$Freq)/sum(dominantColours$Freq)
min_col  <- min(dominantColours$Freq)/sum(dominantColours$Freq)
medium_col <- 1-max_col - min_col
```
### CREAR PIE CHART PARA 3 CLUSTER
```
dominantColours$distribution <- round((c(medium_col, min_col, max_col) * 100), 2)
dominantColours
dominantColours$colours <- as.character(dominantColours$colours)
pie(dominantColours$Freq, labels = dominantColours$distribution,
    col = dominantColours$colours,
    main= "Porcentaje de cobertura",
    xlab = "Color",
    ylab = "Frecuencia")
 ```
|Analisis del plot|Analisis de imagen cortada manualmente|
|:----:|:------:|
![analisis1_1](https://user-images.githubusercontent.com/68933213/124402226-e6624000-dcfc-11eb-97fa-2686d00e3d85.jpg)|![analisis_cluster_frec](https://user-images.githubusercontent.com/68933213/124402262-2a554500-dcfd-11eb-8e2d-2c2c881fe11a.png)|
