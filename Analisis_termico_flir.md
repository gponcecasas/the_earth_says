# The Earth Says
# VERSIÓN 2

 ## LIBRERIAS Y FUNCIONES----
library(Thermimage) #PAQUETE DE ANALISIS TERMICAS
library(fields)     #PERMITE ROTAR IMAGENES
library(jpeg)       #ABRIR Y GUARDAR IMAGENES EN JPEG - PNG - TIFF
library(cluster)    #ANALISIS DE COMPONENTES EN UNA IMAGEN RGB

Aqui se definen 2 funciones que generar filtros sin diferenciar las caracteristicas propias de cada elemento, por lo que requiere una supervisión y evaluación manual del operador ya que se ha observado que el analisis por cluster no diferencia cada elemento ingresado y eso es algo que queda pendiente


### FILTRO SEGÚN -> max-percentil90<=2
x se define como el directorio de elementos a utilizar y se puede describir de la siguiente manera:
```
allFiles=list.files(path="C:/Users/CARPETA_DE_ELEMENTOS", pattern=".jpg", full.names=TRUE)
flir <- allFiles
```
y se describe como el directorio donde se van a guardar los analisis y se puede definir como:

```
allAnalisis=list.files(path="C:/Users/CARPETA_DE_ANALISIS", pattern=".jpg", full.names=TRUE)
Analisis <- allAnalisis
```
Entonces las funciones quedan:

```
analizar1 <- function(x, y){
 #EXTRAER METADATA 
   img <- Thermimage::readflirJPG(x, exiftoolpath = "installed")#CREAR CON LA IMAGEN FLIR Y SU METADATA
  cams <- Thermimage::flirsettings(x, exiftoolpath = "installed", camvals="") 
  ObjectEmissivity<-  cams$Info$Emissivity              # Image Saved Emissivity - should be ~0.95 or 0.96
  dateModif <-   cams$Dates$FileModificationDateTime     # Modification date/time extracted from file
  PlanckR1 <-    cams$Info$PlanckR1                      # Planck R1 constant for camera  
  PlanckB <-     cams$Info$PlanckB                       # Planck B constant for camera  
  PlanckF <-     cams$Info$PlanckF                       # Planck F constant for camera
  PlanckO <-     cams$Info$PlanckO                       # Planck O constant for camera
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
  temperatura <- Thermimage::raw2temp(img, ObjectEmissivity, OD, ReflT, AtmosT, IRWinT, IRWinTran,
                                      RH, PlanckR1, PlanckB, PlanckF, PlanckO, PlanckR2, ATA1, ATA2,
                                      ATB1, ATB2, ATX)
  
  #FILTRO
  prom <- mean(temperatura) #PROMEDIO DE TEMPERATURA
  max <- max(temperatura)
  percentil90 = quantile(temperatura, .93)#PERCENTIL 93
  filtro = temperatura #se filtra por los valores mayores al percentil 93
  
  #GUARDAR IMAGEN FILTRADA
  ff <- 1:length(flir)
  nn <- paste("flir4_", ff[i],".jpg", sep="")
  jpeg(nn) #CREA UNA IMAGEN PNG EN EL DIRECTORIO DE TRABAJO
  Thermimage::plotTherm(filtro, h=h, w=w, minrangeset = 0, maxrangeset = 20)
  dev.off()
  
  #ANALISIS POR CLUSTER
  allAnalisis=list.files(path= y, pattern=".jpg", full.names=TRUE)
  analisis <- allAnalisis
  test_flir <- jpeg::readJPEG(analisis[i])
  dm <- dim(test_flir) #VECTOR QUE CONTIENE LAS DIMENSIONES DE LA IMAGEN
  rgbImage1 <- data.frame(
    x=rep(1:dm[2], each=dm[1]),
    y=rep(dm[1]:1, dm[2]),
    r.value=as.vector(test_flir[,,1]),
    g.value=as.vector(test_flir[,,2]),
    b.value=as.vector(test_flir[,,3]))
  testflir = rgbImage1[, c("r.value", "g.value", "b.value")]
  clara <- cluster::clara(testflir, 3) # clara(testflir, nro de cluster)
  colours <- rgb(clara$medoids[clara$clustering, ])
  dominantColours <- as.data.frame(table(colours))
  max_col  <- max(dominantColours$Freq)/sum(dominantColours$Freq)
  min_col  <- min(dominantColours$Freq)/sum(dominantColours$Freq)
  medium_col <- 1-max_col - min_col
  dominantColours$distribution <- round((c(min_col,medium_col, max_col) * 100), 2)
  dominantColours
  dominantColours$colours <- as.character(dominantColours$colours)
  
  #GUARDAR PIE CHART GENERADO
  fff <- 1:length(flir)
  y <- paste("analisis4_", fff[i],".jpg", sep="")
  jpeg(y)
  pie(dominantColours$Freq, labels = dominantColours$distribution,
      col = dominantColours$colours,
      main= "Porcentaje de cobertura",
      xlab = "Color",
      ylab = "Frecuencia")
  dev.off()
  print("ANALISIS EXITOSO")
}

 ```

##### FILTRO según -> max-percentil90>2
 ```
analizar2 <- function(x, y){
  img <- Thermimage::readflirJPG(x, exiftoolpath = "installed")#CREAR CON LA IMAGEN FLIR Y SU METADATA
  cams <- Thermimage::flirsettings(x, exiftoolpath = "installed", camvals="") 
  ObjectEmissivity<-  cams$Info$Emissivity              # Image Saved Emissivity - should be ~0.95 or 0.96
  dateModif <-   cams$Dates$FileModificationDateTime     # Modification date/time extracted from file
  PlanckR1 <-    cams$Info$PlanckR1                      # Planck R1 constant for camera  
  PlanckB <-     cams$Info$PlanckB                       # Planck B constant for camera  
  PlanckF <-     cams$Info$PlanckF                       # Planck F constant for camera
  PlanckO <-     cams$Info$PlanckO                       # Planck O constant for camera
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
  temperatura <- Thermimage::raw2temp(img, ObjectEmissivity, OD, ReflT, AtmosT, IRWinT, IRWinTran,
                                      RH, PlanckR1, PlanckB, PlanckF, PlanckO, PlanckR2, ATA1, ATA2,
                                      ATB1, ATB2, ATX)
  prom <- mean(temperatura) #PROMEDIO DE TEMPERATURA
  max <- max(temperatura)
  percentil90 = quantile(temperatura, .93)#PERCENTIL 93
  filtro = temperatura>percentil90 #se filtra por los valores mayores al percentil 93
  ff <- 1:length(flir)
  nn <- paste("flir3_", ff[i],".jpg", sep="")
  jpeg(nn) #CREA UNA IMAGEN PNG EN EL DIRECTORIO DE TRABAJO
  Thermimage::plotTherm(filtro, h=h, w=w, minrangeset = 0, maxrangeset = max(filtro)+1)
  dev.off()
  allAnalisis=list.files(path= y, pattern=".jpg", full.names=TRUE) #
  analisis <- allAnalisis
  test_flir <- jpeg::readJPEG(analisis[i])
    dm <- dim(test_flir) #VECTOR QUE CONTIENE LAS DIMENSIONES DE LA IMAGEN
    rgbImage1 <- data.frame(
      x=rep(1:dm[2], each=dm[1]),
      y=rep(dm[1]:1, dm[2]),
      r.value=as.vector(test_flir[,,1]),
      g.value=as.vector(test_flir[,,2]),
      b.value=as.vector(test_flir[,,3]))
    testflir = rgbImage1[, c("r.value", "g.value", "b.value")]
    clara <- cluster::clara(testflir, 3) # clara(testflir, nro de cluster)
    colours <- rgb(clara$medoids[clara$clustering, ])
    dominantColours <- as.data.frame(table(colours))
    max_col  <- max(dominantColours$Freq)/sum(dominantColours$Freq)
    min_col  <- min(dominantColours$Freq)/sum(dominantColours$Freq)
    medium_col <- 1-max_col - min_col
    dominantColours$distribution <- round((c(medium_col, min_col, max_col) * 100), 2)
    dominantColours
    dominantColours$colours <- as.character(dominantColours$colours)
    fff <- 1:length(flir)
    y <- paste("analisis3_", fff[i],".jpg", sep="")  
    jpeg(y)
       pie(dominantColours$Freq, labels = dominantColours$distribution,
          col = dominantColours$colours,
          main= "Porcentaje de cobertura",
          xlab = "Color",
          ylab = "Frecuencia")
      dev.off()
    print("ANALISIS EXITOSO")
  }
 ```
 ***ESTAS FUNCIONES SE APLICAN DE LA SIGUIENTE FORMA***
Se debe definir una carpeta para guardar los elementos por EJEMPLO:
```
setwd("C:/Users/analisis2")
allFiles=list.files(path="C:/Users/gapon/Desktop/data_analisis/earth_says/the_earth_says/25.06.21", pattern=".jpg", full.names=TRUE)
flir <- allFiles
for(i in 1:length(flir)){
  analizar1(flir[i])
}
setwd("C:/Users/analisis2")
for(i in 1:length(flir)){
  analizar2(flir[i])
}
```

## VERSIÓN 3
Aqui las dos funciones no se definen por separado, sino, que se aplica el analisis de forma bruta en un Loop, esto se piensa para automatizar el analisis, no obstante, aun presenta problemas con el output de la clausterización ya que de forma individual se diferencian las areas de calor pero al aplicar a un grupo esta diferenciacion se pierde

```
library(Thermimage)
library(fields)
```
Se tiene que definir un directorio para guardar los archivos mediante setwd() y tambien el directorio donde encontraremos los documentos a analizar mediante allFiles = ...
```
setwd("C:/Users/analisis")
allFiles=list.files(path="C:/Users/ELEMENTOS_A_ANALIZAR", pattern=".jpg", full.names=TRUE)
flir <- allFiles
```
El Loop queda como sigue

```
for(i in 1:length(flir)) {
  img <- Thermimage::readflirJPG(flir[i], exiftoolpath = "installed")#FLIR RAW
  cams <- Thermimage::flirsettings(flir[i], exiftoolpath = "installed", camvals="") #FLIR METADATA
  ObjectEmissivity<-  cams$Info$Emissivity              # Image Saved Emissivity - should be ~0.95 or 0.96
  dateModif <-   cams$Dates$FileModificationDateTime     # Modification date/time extracted from file
  PlanckR1 <-    cams$Info$PlanckR1                      # Planck R1 constant for camera  
  PlanckB <-     cams$Info$PlanckB                       # Planck B constant for camera  
  PlanckF <-     cams$Info$PlanckF                       # Planck F constant for camera
  PlanckO <-     cams$Info$PlanckO                       # Planck O constant for camera
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
  #CREA UNA MATRIZ QUE DEFINI TEMPERATURA POR PIXEL
  temperatura <- Thermimage::raw2temp(img, ObjectEmissivity, OD, ReflT, AtmosT, IRWinT, IRWinTran,
                                      RH, PlanckR1, PlanckB, PlanckF, PlanckO, PlanckR2, ATA1, ATA2,
                                      ATB1, ATB2, ATX)
  prom <- mean(temperatura) #PROMEDIO DE TEMPERATURA
  max <- max(temperatura)   #TEMP. MAXIMA
  percentil90 = quantile(temperatura, .90)#PERCENTIL 90
  if(max>16){
    filtro = temperatura>percentil90 
    imagen1 <- 1:length(flir)
    nombre1 <- paste("flir1_", imagen1[i],".jpg", sep="")
    jpeg(nombre1) #CREA UNA IMAGEN PNG EN EL DIRECTORIO DE TRABAJO
    Thermimage::plotTherm(filtro, h=h, w=w, minrangeset = 0, maxrangeset = max(filtro)+1, trans="rotate270.matrix")
    dev.off()
    allAnalisis=list.files(path="C:/Users/gapon/Desktop/data_analisis/earth_says/the_earth_says/analisis", pattern=".jpg", full.names=TRUE)
    analisis <- allAnalisis
    test_flir <- jpeg::readJPEG(analisis[i])
    dm <- dim(test_flir) #VECTOR QUE CONTIENE LAS DIMENSIONES DE LA IMAGEN
    rgbImage1 <- data.frame(
      x=rep(1:dm[2], each=dm[1]),
      y=rep(dm[1]:1, dm[2]),
      r.value=as.vector(test_flir[,,1]),
      g.value=as.vector(test_flir[,,2]),
      b.value=as.vector(test_flir[,,3]))
    testflir = rgbImage1[, c("r.value", "g.value", "b.value")]
    clara <- cluster::clara(testflir, 3) # clara(testflir, nro de cluster)
    colours <- rgb(clara$medoids[clara$clustering, ])
    dominantColours <- as.data.frame(table(colours))
    max_col  <- max(dominantColours$Freq)/sum(dominantColours$Freq)
    min_col  <- min(dominantColours$Freq)/sum(dominantColours$Freq)
    medium_col <- 1-max_col - min_col
    dominantColours$distribution <- round((c(medium_col, min_col, max_col) * 100), 2)
    dominantColours
    dominantColours$colours <- as.character(dominantColours$colours)
    chart1 <- 1:length(flir)
    nombre1_2 <- paste("analisis1_", chart1[i],".jpg", sep="")  
    jpeg(nombre1_2)
    pie(dominantColours$Freq, labels = dominantColours$distribution,
        col = dominantColours$colours,
        main= "Porcentaje de cobertura",
        xlab = "Color",
        ylab = "Frecuencia")
    dev.off()
  }else{
    filtro = temperatura
    imagen2 <- 1:length(flir)
    nombre2 <- paste("flir2_", imagen2[i],".jpg", sep="")
    jpeg(nombre2) #CREA UNA IMAGEN PNG EN EL DIRECTORIO DE TRABAJO
    Thermimage::plotTherm(filtro, h=h, w=w, minrangeset = 0, maxrangeset = max+2, trans="rotate270.matrix")
    dev.off()
    allAnalisis=list.files(path="C:/Users/gapon/Desktop/data_analisis/earth_says/the_earth_says/analisis", pattern=".jpg", full.names=TRUE)
    analisis <- allAnalisis
    test_flir <- jpeg::readJPEG(analisis[i])
    dm <- dim(test_flir) #VECTOR QUE CONTIENE LAS DIMENSIONES DE LA IMAGEN
    rgbImage1 <- data.frame(
      x=rep(1:dm[2], each=dm[1]),
      y=rep(dm[1]:1, dm[2]),
      r.value=as.vector(test_flir[,,1]),
      g.value=as.vector(test_flir[,,2]),
      b.value=as.vector(test_flir[,,3]))
    testflir = rgbImage1[, c("r.value", "g.value", "b.value")]
    clara <- cluster::clara(testflir, 3) # clara(testflir, nro de cluster)
    colours <- rgb(clara$medoids[clara$clustering, ])
    dominantColours <- as.data.frame(table(colours))
    max_col  <- max(dominantColours$Freq)/sum(dominantColours$Freq)
    min_col  <- min(dominantColours$Freq)/sum(dominantColours$Freq)
    medium_col <- 1-max_col - min_col
    dominantColours$distribution <- round((c(medium_col, min_col, max_col) * 100), 2)
    dominantColours
    dominantColours$colours <- as.character(dominantColours$colours)
    chart2 <- 1:length(flir)
    y <- paste("analisis2_", chart2[i],".jpg", sep="")  
    jpeg(y)
    pie(dominantColours$Freq, labels = dominantColours$distribution,
        col = dominantColours$colours,
        main= "Porcentaje de cobertura",
        xlab = "Color",
        ylab = "Frecuencia")
    dev.off()
  }
  print("ANALISIS EXITOSO")
}
```
 
