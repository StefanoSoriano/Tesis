# Pronóstico de la inflación en México, 2019-2020, a través del Índice Nacional de Precios al Consumidor (INPC) general. Modelo ARIMA estacional.

#### En este repositotio desarrollaré un modelo predictivo para el pronóstico de la inflación en México a partir de la segunda quincena de julio 2019 a la primera quicena de julio 2020; analizando el INPC general quincenal con el lenguaje de programación estadística R. 
#### La metodología predictiva utilizada es Box-Jenkins para generar un modelo *ARIMA* por sus siglas en inglés *Autoregressive Integrated Moving Average* o modelo Autorregresivo Intregrado de Promedio Móvil.

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


