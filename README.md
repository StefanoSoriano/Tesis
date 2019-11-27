# Pronóstico de la inflación en México, 2019-2020, a través del Índice Nacional de Precios al Consumidor (INPC) general. Modelo ARIMA estacional.

#### En este repositotio desarrollaré un modelo predictivo para el pronóstico de la inflación en México a partir de la segunda quincena de julio 2019 a la primera quicena de julio 2020; analizando el INPC general quincenal con el lenguaje de programación estadística R. 
#### La metodología predictiva utilizada es Box-Jenkins para generar un modelo *ARIMA* por sus siglas en inglés *Autoregressive Integrated Moving Average* o modelo Autorregresivo Intregrado de Promedio Móvil.

#### Antes de comenzar a analizar la serie de tiempo es necesario instalar y cargar en RStudio las librerías a utilizar y posteriormente crear un objeto de serie de tiempo (ts).

* ##### Una librería en R se puede entender como un algoritmo que tiene instrucciones y que R ejecutará para que realizar cálculos, operaciones matemáticas o para transformar los datos con los que se está trabajando, entre muchas otras instrucciones.

* ##### Para que R "trate" a los datos a analizar como una serie de tiempo es necesario escribir en la consola la siguiente función:  `ts(data = NA, start = 1, end = numeric(), frequency = 1)`

##### Donde:
* ##### *data* es el nombre de la serie de tiempo
* ##### *start* es el año de inicio de la serie de tiempo
* ##### *end* es el año final de la serie de tiempo
* ##### *frequency* es la periodicidad de la serie de tiempo, en este caso la periodicidad es quincenal

###### Para mayor información de la función *ts*, visitar [ts rdocumentation](https://www.rdocumentation.org/packages/stats/versions/3.6.1/topics/ts)

##### Instalar las librerías que se van a utilizar

```r	
#|||||||||||||||||||||||||
#  Instalar librerías
#|||||||||||||||||||||||||

#  La condición if evalúa que la librería no se encuentre dentro de los paquetes instalados, si la condición resulta ser verdad, 
#  es decir, si la librería en particular no se encuentra dentro de los paquetes instalados se instala.

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

#|||||||||||||||||||||||||
#  Cargar librerías                
#|||||||||||||||||||||||||

library(astsa)
library(tseries)
library(timeSeries)
library(fGarch)
library(stats)
library(zoo)
library(ggplot2)
library(moments)

```


# Graficando la serie en su forma de niveles



##                                             Gráfica 1
<img src="https://github.com/StefanoSoriano" alt="drawing"/>

###### Fuente: Elaboración propia en Excel con datos del BIE del INEGI


