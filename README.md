# Tesina

##### En este proyecto subiré los resultados obtenidos de mi tesina para recibir el grado de Economista, el proyecto es un pronóstico de la inflación en México para el año 2019; analizando el INPC general quincenal con el lenguaje de programación estadística R. 


##### Para el análisis predictivo seguí la metodología Box-Jenkins 

### PRIMERA FASE

##### Para comenzar se instalan las librerías que se van a utilizar

```r	
if (!"astsa" %in% rownames(installed.packages())) {
 install.packages("astsa")
}
if (!"tseries" %in% rownames(installed.packages())) {
 install.packages("tseries")
}
if (!"timeSeries" %in% rownames(installed.packages())) {
 install.packages("timeSeries")
}
if (!"fGarch" %in% rownames(installed.packages())) {
 install.packages("fGarch")
}
if (!"forecast" %in% rownames(installed.packages())) {
 install.packages("forecast")
}
if (!"dplyr" %in% rownames(installed.packages())) {
 install.packages("dplyr")
}
if (!"stats" %in% rownames(installed.packages())) {
 install.packages("stats")
}
if (!"zoo" %in% rownames(installed.packages())) {
 install.packages("zoo")
}
if (!"moments" %in% rownames(installed.packages())) {
 install.packages("moments")
}
		
```


# Graficando la serie en su forma de niveles



##                                             Gráfica 1
<img src="https://github.com/StefanoSoriano/Tesis/blob/master/imágenes/Inflación%20inmediata%202018.png" alt="drawing"/>

###### Fuente: Elaboración propia con datos del BIE del INEGI.


