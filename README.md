
# Pronóstico de la inflación mexicana 2019-2020, a través del Índice Nacional de Precios al Consumidor (INPC) general. Modelo ARIMA estacional.

## Capítulo V. Aplicación de la metodología Box-Jenkins al INPC con el lenguaje R
 
#### En este repositorio desarrollaré un modelo predictivo para el pronóstico de la inflación en México a partir de la segunda quincena de julio de 2019 a la primera quincena de julio de 2020, a través de un modelo ARIMA estacional (SARIMA); analizando el INPC general con el lenguaje de programación estadística R. _El desarrollo del modelo predictivo pertenece al quinto capítulo de mi tesina._
#### La metodología utilizada para generar un modelo *Seasonal Autoregressive Integrated Moving Average* por sus siglas en inglés *SARIMA* o, modelo Estacional Autorregresivo Intregrado de Promedio Móvil, es Box-Jenkins .

#### Antes de comenzar a analizar la serie de tiempo es necesario instalar y cargar en [RStudio®](https://rstudio.com/) las librerías a utilizar, importar los datos del INPC general almacenados en el directorio y posteriormente crear un objeto de serie de tiempo (ts).

* ##### Una librería en R se puede entender como un conjunto de funciones en la que cada función es un bloque de código formado por varias líneas (instrucciones) que R ejecutará una por una de manera descendente para realizar cálculos estadísticos, operaciones matemáticas o, para transformar los datos con los que se está trabajando, entre muchas otras instrucciones.

* ##### Para que R "trate" a los datos a analizar como una serie de tiempo es necesario escribir en la consola la siguiente función:
```r
ts(data = NA, start = 1, end = numeric(), frequency = 1)
```

##### Donde:
* ##### *data* es el nombre de la serie de tiempo
* ##### *start* es el año de inicio de la serie de tiempo
* ##### *end* es el año final de la serie de tiempo
* ##### *frequency* es la periodicidad de la serie de tiempo, en este caso la periodicidad es quincenal

###### Para mayor información de la función *ts*, visitar: [ts rdocumentation](https://www.rdocumentation.org/packages/stats/versions/3.6.1/topics/ts)

# Inicio

####  Instalar librerías

####  La condición if evalúa que la librería no se encuentre dentro de los paquetes instalados, si la condición resulta ser verdad, es decir, si la librería en particular no se encuentra dentro de los paquetes instalados entonces se instala.
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
if (!"RCurl" %in% rownames(installed.packages())) {
 install.packages("RCurl")
}
```

####  Cargar librerías                

```r
library(astsa)
library(tseries)
library(timeSeries)
library(fGarch)
library(stats)
library(zoo)
library(ggplot2)
library(moments)
library(RCurl)
```

####  Importar datos del INPC general desde repositorio en GitHub

```r
GitHubPath <- "https://github.com/StefanoSoriano/Tesis/blob/master/" #  Ruta del repositorio en GitHub
nameSerie <- "INPC general quincenal.csv" #  Nombre del dataset
INPC_URL <- paste0(GitHubPath, nameSerie) #  URL del repositorio donde está almacenado el dataset

INPC <- getURL(INPC_URL) # Obtener URL del dataset
INPC <- read.csv(text = INPC, header = TRUE, dec = '.', na.strings = "NA",stringsAsFactors = FALSE) 
```

####  Crear objeto de serie de tiempo (ts) 


```r
INPC <- ts(INPC, start = c(1, 1988), frequency = 24) 
```


#### Graficando la serie en su forma de niveles



##                                             Gráfica 1
<img src="https://github.com/StefanoSoriano/Tesis/blob/master/Im%C3%A1genes/INPC%20general%20(niveles).jpg?raw=true" alt="drawing"/>

###### Fuente: Elaboración propia en Excel con datos del BIE del INEGI

#### La Gráfica 1 muestra la evolución del INPC, se distingue la tendencia creciente determinista y la falta de estacionariedad en media; debido a que las observaciones no tienen una trayectoria alrededor de su valor promedio, es decir, la media de las mismas cambia a lo largo del periodo, se descartó falta de estacionariedad en varianza debido a que, mientras aumenta el periodo, la varianza de las observaciones no cambia a lo largo del tiempo, para corroborar que se necesita diferenciar la serie de tiempo y estabilizar su varianza.

####  Para corroborar que se necesita diferenciar la serie de tiempo y estabilizar su varianza se aplicó a las observaciones la prueba Augmented Dickey-Fuller (adf.test), esta prueba tiene como hipótesis nula (H<sub>0</sub>) la no estacionariedad de la serie y como hipótesis alternativa (H<sub>a</sub>) la estacionariedad de la serie; formalmente: 
* #### H<sub>0</sub>: serie de tiempo no estacionaria (raíz unitaria).
* #### H<sub>a</sub>: serie de tiempo estacionaria.
#### Para rechazar la H<sub>0</sub> frente a la H<sub>a</sub>, el valor p tiene que ser menor que 0.05; de lo contrario, se acepta la H<sub>0</sub> frente a la H<sub>a</sub> y se aplican las primeras diferencias a la serie.

```r
test.raiz <- adf.test(var_x)
options(digits=2) 
if (test.raiz$p.value < 0.05)
{
valor <- test.raiz$p.value  
    cat("No hay Raíz unitaria (serie de tiempo estacionaria) el valor p es: ", valor,   
        "\npor lo tanto, se rechaza la H0 de no estacionariedad.",
         "\nSe puede entonces calcular el modelo de ajuste.")
} else {
 valor <- test.raiz$p.value
    cat("Raíz unitaria (serie de tiempo no estacionaria) el valor p es: " , valor,
         
        "\nDebido a que la serie no es estacionaria entonces se estabiliza la serie de tiempo",
 "\nantes de estimar el modelo.");     
n_diff_x <- forecast::ndiffs(var_x, test = c("adf"))
    var_diff_x <- diff(var_x, n_diff_x)
 }    
```

### El resultado de la prueba ADF fue el siguiente:

```r
## Raíz unitaria (serie de tiempo no estacionaria) el valor p es:  0.68 
## Debido a que la serie no es estacionaria entonces se estabiliza la serie de tiempo 
## antes de estimar el modelo.
```
#### El resultado de la prueba generó un valor p mayor que 0.05, por ello, se acepta la H<sub>0</sub> frente a la H<sub>a</sub> y se diferencia una vez la serie de tiempo, se vuelve a aplicar la prueba para comprobar que ya no hay raíz unitaria, antes de ver los resultados de la prueba, se graficó la serie del INPC diferenciada para observar si tiene un comportamiento de ruido blanco: 

##                                             Gráfica 2
<img src="https://github.com/StefanoSoriano/Tesis/blob/master/Im%C3%A1genes/Primeras%20diferencias.jpg?raw=true" alt="drawing"/>

###### Fuente: Elaboración propia en Excel con datos del BIE del INEGI

#### La Gráfica 2 muestra que las observaciones del INPC diferenciadas tienen una trayectoria alrededor del cero (valor medio de la serie del INPC) y con varianza constante, es decir, es ruido blanco, lo que significa que la serie es estacionaria en varianza, se vue
```r
test.raiz <- adf.test(var_diff_x)
options(digits=2) 
if (test.raiz$p.value < 0.05)
{
valor <- test.raiz$p.value  
     cat("Serie de tiempo estacionaria el valor p es: ", valor, 
    "
por lo tanto, se rechaza la H0 de no estacionariedad.
Se puede entonces calcular el modelo de ajuste")
} else {
 valor <- test.raiz$p.value
    cat("Serie de tiempo no estacionaria.
",
       "Debido a que la serie sigue presentando no estacionariedad",
       "se calculan las primeras diferencias de la serie",
        "junto con el rezago de orden 24, el valor p es el siguiente: ")
    diff_orden <- diff(var_x, lag = 24)
    test.raiz <- adf.test(diff_orden)
       options(digits=2) 
       if (test.raiz$p.value < 0.05)
       {
          valor <- test.raiz$p.value  
          cat(valor, 
           "
por lo tanto, la serie es estacionaria. 
Se puede entonces calcular el modelo de ajuste")
       } else {
           cat("Serie de tiempo no estacionaria.")
         }
}  
```
#### El resultado de la prueba ADF fue:

```r
## Serie de tiempo estacionaria el valor p es:  0.01 
## por lo tanto, se rechaza la H0 de no estacionariedad.
## Se puede entonces calcular el modelo de ajuste
```
#### En efecto, el valor p es menor que 0.05, por lo que se rechazó la H<sub>0</sub> de no estacionariedad frente a la H<sub>a</sub> de estacionariedad; como la serie de tiempo es estacionaria, se prosiguió a la fase de identificación de los órdenes ARIMA.

##                                             Gráfica 3
<img src="https://github.com/StefanoSoriano/Tesis/blob/master/Im%C3%A1genes/Diferencias%20estacionales.jpg?raw=true" alt="drawing"/>

###### Fuente: Elaboración propia en Excel con datos del BIE del INEGI


##                                             Gráfica 4
<img src="https://github.com/StefanoSoriano/Tesis/blob/master/Im%C3%A1genes/Medidas%20de%20precisi%C3%B3n.jpg?raw=true" alt="drawing"/>

###### Fuente: Elaboración propia en Excel con datos del BIE del INEGI

\begin{equation}
   $$MAPE = \frac{1}{n}\sum_{i = 1}^{n}\frac{\left | {A_i} - {F_i} \right |* 100}{{A_i}}$$
\end{equation}

##                                             Gráfica 5
<img src="https://github.com/StefanoSoriano/Tesis/blob/master/Im%C3%A1genes/Exactitud.jpg?raw=true" alt="drawing"/>

###### Fuente: Elaboración propia en Excel con datos del BIE del INEGI

##                                             Gráfica 6
<img src="https://github.com/StefanoSoriano/Tesis/blob/master/Im%C3%A1genes/Akaike%20y%20BIC.jpg?raw=true" alt="drawing"/>

###### Fuente: Elaboración propia en Excel con datos del BIE del INEGI

##                                             Gráfica 7
<img src="https://github.com/StefanoSoriano/Tesis/blob/master/Im%C3%A1genes/Simulaci%C3%B3n.jpg?raw=true" alt="drawing"/>

###### Fuente: Elaboración propia en Excel con datos del INPC observado y pronosticado

##                                             Gráfica 8
<img src="https://github.com/StefanoSoriano/Tesis/blob/master/Im%C3%A1genes/Pron%C3%B3stico%20del%20INPC.jpg?raw=true"/>

###### Fuente: Elaboración propia en Excel con datos del INPC pronosticado


##                                             Gráfica 9
<img src="https://github.com/StefanoSoriano/Tesis/blob/master/Im%C3%A1genes/Pron%C3%B3stico%20de%20la%20inflaci%C3%B3n.jpg?raw=true" alt="drawing"/>

###### Fuente: Elaboración propia en Excel con datos del INPC pronosticado

# Seguimiento al comportamiento del INPC e inflación

<img src="https://github.com/StefanoSoriano/Tesis/blob/master/Im%C3%A1genes/Seguimiento a inflación e INPC.png" alt="drawing"/>

###### Fuente: Elaboración propia en Excel con datos del INPC pronosticado e INPC observado

## Datos del INPC general y de la inflación (período quincenal)
###### *NOTA: El cálculo de la inflación quincenal mostrada enseguida es anualizada, el INEGI proporciona el valor de la inflación quincenal inmediata.* 

##### Fecha y hora de consulta: 27 de marzo del 2020 a las 10:34 pm

|	Período	|	INPC Pronosticado	|	INPC Observado	|  	Inflación Pronosticada	|	Inflación Observada	|
|	----------	|	----------	|	----------	|	----------	|	----------	|
|	2Q julio 2019	|	103.79	|	103.72	|	3.79%	|	3.72%	|
|	1Q agosto 2019	|	104.05	|	103.642	|	3.70%	|	3.29%	|
|	2Q agosto 2019	|	104.19	|	103.697	|	3.53%	|	3.04%	|
|	1Q septiembre 2019	|	104.52	|	103.877	|	3.62%	|	2.99%	|
|	2Q septiembre 2019	|	104.55	|	104.007	|	3.55%	|	3.01%	|
|	1Q octubre 2019	|	105.02	|	104.42	|	3.60%	|	3.01%	|
|	2Q octubre 2019	|	105.14	|	104.587	|	3.58%	|	3.03%	|
|	1Q noviembre 2019	|	105.84	|	105.299	|	3.63%	|	3.10%	|
|	2Q noviembre 2019	|	105.90	|	105.393	|	3.34%	|	2.85%	|
|	1Q diciembre 2019	|	106.29	|	105.763	|	3.15%	|	2.63%	|
|	2Q diciembre 2019	|	106.48	|	106.105 |	3.38%	|	3.02%	|
|	1Q enero 2020	|	106.88	|	106.388 |	3.66%	|	3.18%	|
|	2Q enero 2020	|	107.10	|	106.506	|	3.86%	|	3.29%	|
|	1Q febrero 2020	|	107.27	|	106.636	|	4.14%	|	3.52%	|
|	2Q febrero 2020	|	107.37	|	107.141	|	4.10%	|	3.87%	|
|	1Q marzo 2020	|	107.62	|	107.254	|	4.06%	|	3.71%	|
|	2Q marzo 2020	|	107.78	|		|	4.10%	|		|
|	1Q abri 2020	|	107.55	|		|	3.91%	|		|
|	2Q abril 2020	|	107.61	|		|	3.91%	|		|
|	1Q mayo 2020	|	107.26	|		|	3.88%	|		|
|	2Q mayo 2020	|	107.39	|		|	4.05%	|		|
|	1Q junio 2020	|	107.49	|		|	4.13%	|		|
|	2Q junio 2020	|	107.58	|		|	4.07%	|		|
|	1Q julio 2020	|	107.82	|		|	4.01%	|		|

###### Fuente: Elaboración propia con datos del BIE del INEGI
###### [Los datos del INPC general quincenal (actualizados) se pueden consultar dando click aquí](https://www.inegi.org.mx/sistemas/bie/?idserPadre=10000500001500600040)



