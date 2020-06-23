
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
test.raiz <- adf.test(INPC)
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
n_diff_x <- forecast::ndiffs(INPC, test = c("adf"))
    INPC_diff <- diff(INPC, n_diff_x)
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
test.raiz <- adf.test(INPC_diff)
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
    diff_orden <- diff(INPC, lag = 24)
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

## 1. Identificación
### Gráficas de la función de autocorrelación (ACF) y de la función de autocorrelación parcial (PACF) para obtener los patrones del proceso AR() y MA().

```r
acf2(ts(INPC_diff, frequency = 1))
INPC_dif_orden <- diff(diff(INPC), lag = 24) ### Diferencias estacionales
```
### Gráficas de la función de autocorrelación (ACF) y de la función de autocorrelación parcial (PACF) para obtener los patrones del proceso AR() y MA() del componente estacional.

```r
acf2(ts(INPC_dif_orden, frequency = 1))
```
#### Criterios AIC y BIC de los modelos de ajuste

```r
INPC <- INPC
ajuste_INPC_1 <- arima(INPC, order=c(0,1,0), seasonal = list(order = c(1,1,2), period = 24),method="ML", include.mean = TRUE)
ajuste_INPC_2 <- arima(INPC, order=c(1,1,0), seasonal = list(order = c(1,1,2), period = 24),method="ML", include.mean = TRUE)
ajuste_INPC_3 <- arima(INPC, order=c(0,1,1), seasonal = list(order = c(1,1,2), period = 24),method="ML", include.mean = TRUE)
ajuste_INPC_4 <- arima(INPC, order=c(1,1,1), seasonal = list(order = c(1,1,2), period = 24),method="ML", include.mean = TRUE)
65
ajuste_INPC_5 <- arima(INPC, order=c(0,1,2), seasonal = list(order = c(1,1,2), period = 24),method="ML", include.mean = TRUE)
ajuste_INPC_6 <- arima(INPC, order=c(2,1,0), seasonal = list(order = c(1,1,2), period = 24),method="ML", include.mean = TRUE)
ajuste_INPC_7 <- arima(INPC, order=c(1,1,2), seasonal = list(order = c(1,1,2), period = 24),method="ML", include.mean = TRUE)
ajuste_INPC_8 <- arima(INPC, order=c(2,1,1), seasonal = list(order = c(1,1,2), period = 24),method="ML", include.mean = TRUE)
ajuste_INPC_9 <- arima(INPC, order=c(2,1,2), seasonal = list(order = c(1,1,2), period = 24),method="ML", include.mean = TRUE)
```
```r
Modelo_Ajustado <-c("ajuste_INPC_1","ajuste_INPC_2","ajuste_INPC_3","ajuste_INPC_4","ajuste_INPC_5","ajuste_INPC_6","ajuste_INPC_7","ajuste_INPC_8", "ajuste_INPC_9")
Patrones_pdq_y_PDQ <- c("(0,1,0) (1,1,2)","(1,1,0) (1,1,2)","(0,1,1) (1,1,2)","(1,1,1) (1,1,2)","(0,1,2) (1,1,2)","(2,1,0) (1,1,2)","(1,1,2) (1,1,2)","(2,1,1) (1,1,2)","(2,1,2) (1,1,2)")
AIC <- c(ajuste_INPC_1$aic, ajuste_INPC_2$aic, ajuste_INPC_3$aic, ajuste_INPC_4$aic,ajuste_INPC_5$aic, ajuste_INPC_6$aic, ajuste_INPC_7$aic,ajuste_INPC_8$aic, ajuste_INPC_9$aic)
BIC <- c(BIC(ajuste_INPC_1), BIC(ajuste_INPC_2), BIC(ajuste_INPC_3), BIC(ajuste_INPC_4),BIC(ajuste_INPC_5), BIC(ajuste_INPC_6),
BIC(ajuste_INPC_7),BIC(ajuste_INPC_8), BIC(ajuste_INPC_9
```
```r
Summary_stat <- cbind(Patrones_pdq_y_PDQ, AIC, BIC)
Summary_stat
```
```r
Modelo_Ajustado <- as.data.frame(Modelo_Ajustado)
AIC <- as.data.frame(AIC)
BIC <- as.data.frame(BIC)
Summary_stat <- cbind(Modelo_Ajustado, AIC, BIC)
# AIC
index_Summary_stat_Akaike_bajo <- which(Summary_stat[,2] == min(Summary_stat[,2]))
Akaike_mas_bajo <- cbind(Summary_stat[index_Summary_stat_Akaike_bajo,1])
Ajuste_Akaike_mas_bajo <- cbind(Summary_stat[index_Summary_stat_Akaike_bajo,2])
# BIC
index_Summary_stat_BIC_bajo <- which(Summary_stat[,3] == min(Summary_stat[,3]))
BIC_mas_bajo <- cbind(Summary_stat[index_Summary_stat_BIC_bajo,1])
Ajuste_BIC_mas_bajo <- cbind(Summary_stat[index_Summary_stat_BIC_bajo,3])
cat("Modelo de ajuste con valor Akaike más bajo:", Akaike_mas_bajo[1,1])
cat("Valor Akaike:", Ajuste_Akaike_mas_bajo[1,1])
cat("Modelo de ajuste con valor BIC más bajo:", BIC_mas_bajo[1,1])
cat("Valor BIC:", Ajuste_BIC_mas_bajo[1,1])
#### Medidas de precisión de los modelos de ajuste
66
accuracy_fit_INPC_1 <- forecast::accuracy(ajuste_INPC_1)
accuracy_fit_INPC_2 <- forecast::accuracy(ajuste_INPC_2)
accuracy_fit_INPC_3 <- forecast::accuracy(ajuste_INPC_3)
accuracy_fit_INPC_4 <- forecast::accuracy(ajuste_INPC_4)
accuracy_fit_INPC_5 <- forecast::accuracy(ajuste_INPC_5)
accuracy_fit_INPC_6 <- forecast::accuracy(ajuste_INPC_6)
accuracy_fit_INPC_7 <- forecast::accuracy(ajuste_INPC_7)
accuracy_fit_INPC_8 <- forecast::accuracy(ajuste_INPC_8)
accuracy_fit_INPC_9 <- forecast::accuracy(ajuste_INPC_9)
MAPE <- c(accuracy_fit_INPC_1[5],accuracy_fit_INPC_2[5],accuracy_fit_INPC_3[5],
accuracy_fit_INPC_4[5],accuracy_fit_INPC_5[5],accuracy_fit_INPC_6[5],
accuracy_fit_INPC_7[5],accuracy_fit_INPC_8[5],accuracy_fit_INPC_9[5])
ME <- c(accuracy_fit_INPC_1[1],accuracy_fit_INPC_2[1],accuracy_fit_INPC_3[1],
accuracy_fit_INPC_4[1],accuracy_fit_INPC_5[1],accuracy_fit_INPC_6[1],
accuracy_fit_INPC_7[1],accuracy_fit_INPC_8[1],accuracy_fit_INPC_9[1])
RMSE <- c(accuracy_fit_INPC_1[2],accuracy_fit_INPC_2[2],accuracy_fit_INPC_3[2],
accuracy_fit_INPC_4[2],accuracy_fit_INPC_5[2],accuracy_fit_INPC_6[2],
accuracy_fit_INPC_7[2],accuracy_fit_INPC_8[2],accuracy_fit_INPC_9[2])
MAE <- c(accuracy_fit_INPC_1[3],accuracy_fit_INPC_2[3],accuracy_fit_INPC_3[3],
accuracy_fit_INPC_4[3],accuracy_fit_INPC_5[3],accuracy_fit_INPC_6[3],
accuracy_fit_INPC_7[3],accuracy_fit_INPC_8[3],accuracy_fit_INPC_9[3])
MPE <- c(accuracy_fit_INPC_1[4],accuracy_fit_INPC_2[4],accuracy_fit_INPC_3[4],
accuracy_fit_INPC_4[4],accuracy_fit_INPC_5[4],accuracy_fit_INPC_6[4],
accuracy_fit_INPC_7[4],accuracy_fit_INPC_8[4],accuracy_fit_INPC_9[4])
MASE <- c(accuracy_fit_INPC_1[6],accuracy_fit_INPC_2[6],accuracy_fit_INPC_3[6],
accuracy_fit_INPC_4[6],accuracy_fit_INPC_5[6],accuracy_fit_INPC_6[6],
accuracy_fit_INPC_7[6],accuracy_fit_INPC_8[6],accuracy_fit_INPC_9[6])
Summary_accuracy <- cbind(MAPE,ME,RMSE,MAE,MPE,MASE)
Summary_accuracy

setwd(ruta_exp)
write.csv(Summary_accuracy, "Medidas_de_precisión.csv")
setwd(ruta)
Summary_accuracy <- cbind(Modelo_Ajustado,MAPE,ME,RMSE,MAE,MPE,MASE)
# Calculando índices para obtener los valores más bajos del resumen estadístico de precisión de los modelos de ajuste
index_MAPE <- which(Summary_accuracy[,2] == min(Summary_accuracy[,2]))
MAPE_mas_bajo <- cbind(Summary_accuracy[index_MAPE ,1])
Valor_MAPE_mas_bajo <- cbind(Summary_accuracy[index_MAPE,2])
cat("Modelo de ajuste con MAPE más bajo:", MAPE_mas_bajo[1,1])
cat("Valor MAPE:", Valor_MAPE_mas_bajo[1,1])
index_ME <- which(Summary_accuracy[,3] == min(Summary_accuracy[,3]))
ME_mas_bajo <- cbind(Summary_accuracy[index_ME ,1])
Valor_ME_mas_bajo <- cbind(Summary_accuracy[index_ME,3])
cat("Modelo de ajuste con ME más bajo:", ME_mas_bajo[1,1])
cat("Valor ME:", Valor_ME_mas_bajo[1,1])
67
index_RMSE <- which(Summary_accuracy[,4] == min(Summary_accuracy[,4]))
RMSE_mas_bajo <- cbind(Summary_accuracy[index_RMSE ,1])
Valor_RMSE_mas_bajo <- cbind(Summary_accuracy[index_RMSE,4])
cat("Modelo de ajuste con RMSE más bajo:", RMSE_mas_bajo[1,1])
cat("Valor RMSE:", Valor_RMSE_mas_bajo[1,1])
index_MAE <- which(Summary_accuracy[,5] == min(Summary_accuracy[,5]))
MAE_mas_bajo <- cbind(Summary_accuracy[index_MAE ,1])
Valor_MAE_mas_bajo <- cbind(Summary_accuracy[index_MAE,5])
cat("Modelo de ajuste con MAE más bajo:", MAE_mas_bajo[1,1])
cat("Valor MAE:", Valor_MAE_mas_bajo[1,1])
index_MPE <- which(Summary_accuracy[,6] == min(Summary_accuracy[,6]))
MPE_mas_bajo <- cbind(Summary_accuracy[index_MPE ,1])
Valor_MPE_mas_bajo <- cbind(Summary_accuracy[index_MPE,6])
cat("Modelo de ajuste con MPE más bajo:", MPE_mas_bajo[1,1])
cat("Valor MPE:", Valor_MPE_mas_bajo[1,1])
index_MASE <- which(Summary_accuracy[,7] == min(Summary_accuracy[,7]))
MASE_mas_bajo <- cbind(Summary_accuracy[index_MASE,1])
Valor_MASE_mas_bajo <- cbind(Summary_accuracy[index_MASE,7])
cat("Modelo de ajuste con MASE más bajo:", MASE_mas_bajo[1,1])
cat("Valor MASE:", Valor_MASE_mas_bajo[1,1])
INPC <- INPC
ajuste_x <- arima(INPC, order=c(1,1,1), seasonal = list(order = c(1,1,2), period = 24),method="ML", include.mean = TRUE)
tsdiag(ajuste_x)
INPC_General <- INPC

setwd(ruta_exp)
h_x <- forecast::forecast(ajuste_x)
h_x <- as.data.frame(h_x)
write.csv(h_x, "h_arima_x.csv")
setwd(ruta)
qqnorm(ajuste_x$resid, main = "Normal Q-Q Plot",
xlab = "Theoretical Quantiles", ylab = "Sample Quantiles")
qqline(ajuste_x$resid)
#### Valor de los cuantiles.
resid_ajuste_x <- ajuste_x$residuals
plot(resid_ajuste_x, main = "Graficando los residuos del ajuste del modelo.", ylab = "Residuos", type = "o")
abline(h = mean(resid_ajuste_x), col = "blue")
hist(resid_ajuste_x, main = "Histograma de los residuos del ajuste del modelo.", col= "black", xlab = "Residuos")
68

setwd(ruta_exp)
write.csv(resid_ajuste_x, "residuos_x.csv")
setwd(ruta)
#### Resumen estadístico del ajuste del modelo
cat("Sigma Cuadrado:", ajuste_x$sigma2)
cat("Media de los residuos:", mean(resid_ajuste_x))
cat("Akaike:",ajuste_x$aic)
### Simulación de pronóstico: INPC General.
INPC_General <- INPC_contraste
pronostico <- sarima.for(INPC_General, n.ahead = 24, p = 1, d =1, q = 1, P = 1, D = 1, Q = 2, S = 24, no.constant = FALSE)
values.predict_x <- pronostico$pred
##### Línea **roja** valores observados de la serie de tiempo
##### Línea **azul** valores pronosticados de la serie de tiempo
Período_x <- gsub("/(*)", " ", test_x[,1])
Observed_x <- test_x[,2]
Forecasted_x <- values.predict_x
simulation_x <- data.frame(Período_x, Observed_x, Forecasted_x)
simulation_plot_x <- ggplot(data= simulation_x, aes(x=Período_x, y=Observed_x, group=1)) +
geom_line(col="red2", size = 1.1) +
geom_point(col="red2", size = 3) +
geom_line(col="deeppink2", size = 3.5, alpha = 0.18) +
labs(title = "Valores observados y valores pronosticados.",x = "Fecha", y = "INPC General")+
geom_line(col = "Cyan4", size = 1.1, aes(y=Forecasted_x)) +
geom_point(col = "Cyan4", size = 3, aes(y=Forecasted_x)) +
geom_line(col="Cyan", size = 3.5, alpha = 0.18,aes(y=Forecasted_x))
simulation_plot_x

setwd(ruta_exp)
write.csv(Observed_x, "Observed_x.csv")
write.csv(Forecasted_x, "Forecasted_x.csv")
setwd(ruta)
### **Pronóstico de 24 quincenas para el INPC General de México.**
setwd(ruta)
INPC_General <- read.csv('INPC General quincenal.csv', header = TRUE, dec = '.', stringsAsFactors = FALSE)
INPC <- INPC_General
INPC <- ts(INPC, start = c(1988, 1), frequency = 24)
INPC <- INPC[, -1]
INPC <- zoo::na.approx(INPC)
69
INPC_General <- INPC
pronostico_x <- sarima.for(INPC_General, n.ahead = 24, p = 1, d =1, q = 1, P = 1, D = 1, Q = 2, S = 24, no.constant = FALSE)
### Valor de los predictores
(rbind(ajuste_x$coef))
### Valores de la predicción
(pronostico_x$pred)

setwd(ruta_exp)
write.csv(pronostico_x$pred, "Hforecasted_x.csv")
setwd(ruta)
### Modelo II
setwd(ruta)
INPC_General <- read.csv('INPC General quincenal.csv', header = TRUE, dec = '.', stringsAsFactors = FALSE)
### Particionando la serie de tiempo en dos conjuntos
## Conjunto 1 -- Base de entrenamiento (train)
## Conjunto 2 -- Base de prueba (test)
var_y <- INPC_General
longitud_y <- length(var_y[,2])
numTrain_y <- longitud_y - 24
train_y <- var_y[1:numTrain_y,]
test_y <- tail(var_y, 24) # Valores reales de la serie que van a ser contrastados con los pronosticados.
var_y_contraste <- train_y # Asignando valores del entrenamiento a la variable y que será contrastada
var_y_contraste <- ts(var_y_contraste, start = c(1988, 1), frequency = 24)
var_y_contraste <- var_y_contraste[, -1]
var_y_contraste <- zoo::na.approx(var_y_contraste)
var_y <- ts(var_y, start = c(1988, 1), frequency = 24)
var_y <- var_y[, - 1]
var_y <- zoo::na.approx(var_y)
### El segundo modelo contiene el siguiente patrón **AR(2)I(1)MA(2)** junto con los patrones estacionales (P,D,Q): **AR(1)I(1)MA(2)** . Se deduce el siguiente modelo de ajuste:
###### **ajusteARIMA <- arima(INPC_General, order=c(2,1,2), seasonal = list(order = c(1,1,2), period = 24), method="ML", include.mean = TRUE)**
INPC <- var_y
ajuste_y <- arima(INPC, order=c(2,1,2), seasonal = list(order = c(1,1,2), period = 24),method="ML", include.mean = TRUE)
70
tsdiag(ajuste_y)
INPC_General <- var_y

setwd(ruta_exp)
h_y <- forecast::forecast(ajuste_y)
h_y <- as.data.frame(h_y)
write.csv(h_y, "h_arima_y.csv")
setwd(ruta)
### Graficando los cuantiles de la distribución normal de los residuos del modelo ajustado.
qqnorm(ajuste_y$resid, main = "Normal Q-Q Plot",
xlab = "Theoretical Quantiles", ylab = "Sample Quantiles")
qqline(ajuste_y$resid)
#### Valor de los cuantiles.
resid_ajuste_y <- ajuste_y$residuals
plot(resid_ajuste_y, main = "Graficando los residuos del ajuste del modelo.", ylab = "Residuos", type = "o")
abline(h = mean(resid_ajuste_y), col = "blue")
hist(resid_ajuste_y, main = "Histograma de los residuos del ajuste del modelo.", col= "black", xlab = "Residuos")

setwd(ruta_exp)
write.csv(resid_ajuste_y, "residuos_y.csv")
setwd(ruta)
#### Resumen estadístico del ajuste del modelo
cat("Sigma Cuadrado:", ajuste_y$sigma2)
cat("Media de los residuos:", mean(resid_ajuste_y))
cat("Akaike:",ajuste_y$aic)
### Simulación de pronóstico: INPC General.
INPC_General <- var_y_contraste
pronostico <- sarima.for(INPC_General, n.ahead = 24, p = 2, d =1, q = 2, P = 1, D = 1, Q = 2, S = 24, no.constant = FALSE)
values.predict_y <- pronostico$pred
##### Línea **roja** valores observados de la serie de tiempo
##### Línea **azul** valores pronosticados de la serie de tiempo
Período_y <- gsub("/(*)", " ", test_y[,1])
Observed_y <- test_y[,2]
Forecasted_y <- values.predict_y
simulation_y <- data.frame(Período_y, Observed_y, Forecasted_y)
simulation_plot_y <- ggplot(data= simulation_y, aes(x=Período_y, y=Observed_y, group=1)) +
71
geom_line(col="red2", size = 1.1) +
geom_point(col="red2", size = 3)+
geom_line(col="deeppink2", size = 3.5, alpha = 0.18) +
labs(title = "Valores observados y valores pronosticados.",x = "Fecha", y = "INPC General") +
geom_line(col = "Cyan4", size = 1.1, aes(y=Forecasted_y)) +
geom_point(col = "Cyan4", size = 3, aes(y=Forecasted_y)) +
geom_line(col="Cyan", size = 3.5, alpha = 0.18,aes(y=Forecasted_y))
simulation_plot_y

setwd(ruta_exp)
write.csv(Observed_y, "Observed_y.csv")
write.csv(Forecasted_y, "Forecasted_y.csv")
setwd(ruta)
### **Pronóstico de 24 quincenas para el INPC General de México.**
setwd(ruta)
INPC_General <- read.csv('INPC General quincenal.csv', header = TRUE, dec = '.', stringsAsFactors = FALSE)
var_y <- INPC_General
var_y <- ts(var_y, start = c(1988, 1), frequency = 24)
var_y <- var_y[, - 1]
var_y <- zoo::na.approx(var_y)
INPC_General <- var_y
pronostico_y <- sarima.for(INPC_General, n.ahead = 24, p = 2, d =1, q = 2, P = 1, D = 1, Q = 2, S = 24, no.constant = FALSE)
### Valor de los predictores
(rbind(ajuste_y$coef))
### Valores de la predicción
(pronostico_y$pred)

setwd(ruta_exp)
write.csv(pronostico_y$pred, "Hforecasted_y.csv")
setwd(ruta)
### Modelo III
setwd(ruta)
INPC_General <- read.csv('INPC General quincenal.csv', header = TRUE, dec = '.', stringsAsFactors = FALSE)
### Particionando la serie de tiempo en dos conjuntos
## Conjunto 1 -- Base de entrenamiento (train)
## Conjunto 2 -- Base de prueba (test)
var_z <- INPC_General
longitud_z <- length(var_z[,2])
72
numTrain_z <- longitud_z - 24
train_z <- var_z[1:numTrain_z,]
test_z <- tail(var_z, 24) # Valores reales de la serie
# que van a ser contrastados con los pronosticados.
var_z_contraste <- train_z # Asignando valores del entrenamiento a la variable z que será contrastada
var_z_contraste <- ts(var_z_contraste, start = c(1988, 1), frequency = 24)
var_z_contraste <- var_z_contraste[, -1]
var_z_contraste <- zoo::na.approx(var_z_contraste)
var_z <- ts(var_z, start = c(1988, 1), frequency = 24)
var_z <- var_z[, - 1]
var_z <- zoo::na.approx(var_z)
### El tercer modelo contiene el siguiente patrón **AR(3)I(1)MA(1)** junto con los patrones estacionales (P,D,Q): **AR(1)I(1)MA(2)** . Se deduce el siguiente modelo de ajuste:
###### **ajusteARIMA <- arima(INPC_General, order=c(3,1,1), seasonal = list(order = c(1,1,2), period = 24), method="ML", include.mean = TRUE)**
INPC <- var_z
ajuste_z <- arima(INPC, order=c(3,1,1), seasonal = list(order = c(1,1,2), period = 24),method="ML", include.mean = TRUE)
tsdiag(ajuste_z)
INPC_General <- var_z

setwd(ruta_exp)
h_z <- forecast::forecast(ajuste_z)
h_z <- as.data.frame(h_z)
write.csv(h_z, "h_arima_z.csv")
setwd(ruta)
### Graficando los cuantiles de la distribución normal de los residuos del modelo ajustado.
qqnorm(ajuste_z$resid, main = "Normal Q-Q Plot",
xlab = "Theoretical Quantiles", ylab = "Sample Quantiles")
qqline(ajuste_z$resid)
#### Valor de los cuantiles.
resid_ajuste_z <- ajuste_z$residuals
plot(resid_ajuste_z, main = "Graficando los residuos del ajuste del modelo.", ylab = "Residuos", type = "o")
abline(h = mean(resid_ajuste_z), col = "blue")
hist(resid_ajuste_z, main = "Histograma de los residuos del ajuste del modelo.", col= "black", xlab = "Residuos")

setwd(ruta_exp)
write.csv(resid_ajuste_z, "residuos_z.csv")
setwd(ruta)
73
#### Resumen estadístico del ajuste del modelo
cat("Sigma Cuadrado:", ajuste_z$sigma2)
cat("Media de los residuos:", mean(resid_ajuste_z))
cat("Akaike:",ajuste_z$aic)
### Simulación de pronóstico: INPC General.
INPC_General <- var_z_contraste
pronostico <- sarima.for(INPC_General, n.ahead = 24, p = 3, d =1, q = 1, P = 1, D = 1, Q = 2, S = 24, no.constant = FALSE)
values.predict_z <- pronostico$pred
##### Línea **roja** valores observados de la serie de tiempo
##### Línea **azul** valores pronosticados de la serie de tiempo
Período_z <- gsub("/(*)", " ", test_z[,1])
Observed_z <- test_z[,2]
Forecasted_z <- values.predict_z
simulation_z <- data.frame(Período_z, Observed_z, Forecasted_z)
simulation_plot_z <- ggplot(data= simulation_z, aes(x=Período_z, y=Observed_z, group=1)) +
geom_line(col="red2", size = 1.1) +
geom_point(col="red2", size = 3)+
geom_line(col="deeppink2", size = 3.5, alpha = 0.18) +
labs(title = "Valores observados y valores pronosticados.",x = "Fecha", y = "INPC General") +
geom_line(col = "Cyan4", size = 1.1, aes(y=Forecasted_z)) +
geom_point(col = "Cyan4", size = 3, aes(y=Forecasted_z)) +
geom_line(col="Cyan", size = 3.5, alpha = 0.18,aes(y=Forecasted_z))
simulation_plot_z

setwd(ruta_exp)
write.csv(Observed_z, "Observed_z.csv")
write.csv(Forecasted_z, "Forecasted_z.csv")
setwd(ruta)
### **Pronóstico de 24 quincenas para el INPC General de México.**
setwd(ruta)
INPC_General <- read.csv('INPC General quincenal.csv', header = TRUE, dec = '.', stringsAsFactors = FALSE)
var_z <- INPC_General
var_z <- ts(var_z, start = c(1988, 1), frequency = 24)
var_z <- var_z[, - 1]
var_z <- zoo::na.approx(var_z)
INPC_General <- var_z
pronostico_z <- sarima.for(INPC_General, n.ahead = 24, p = 3, d =1, q = 1, P = 1, D = 1, Q = 2, S = 24, no.constant = FALSE)
74
### Valor de los predictores
(rbind(ajuste_z$coef))
### Valores de la predicción
(pronostico_z$pred)

setwd(ruta_exp)
write.csv(pronostico_z$pred, "Hforecasted_z.csv")
setwd(ruta)
#-----------------------------------------------------------------------
# -Proyecto Terminal Modelos de ajuste I AR() MA()
# Auto Regressive Integrate Moving Average
#-----------------------------------------------------------------------
setwd(ruta)
INPC_General <- read.csv('INPC General quincenal.csv', header = TRUE, dec = '.', stringsAsFactors = FALSE)
INPC <- INPC_General
INPC <- ts(INPC, start = c(1988, 1), frequency = 24)
INPC <- INPC[, -1]
INPC <- zoo::na.approx(INPC)
INPC <- INPC
Ajuste_INPC_1 <- arima(INPC, order=c(0,1,0), seasonal = list(order = c(1,1,2), period = 24),method="ML", include.mean = TRUE)
Ajuste_INPC_2 <- arima(INPC, order=c(1,1,1), seasonal = list(order = c(1,1,2), period = 24),method="ML", include.mean = TRUE)
Ajuste_INPC_3 <- arima(INPC, order=c(2,1,2), seasonal = list(order = c(1,1,2), period = 24),method="ML", include.mean = TRUE)
Ajuste_INPC_4 <- arima(INPC, order=c(3,1,3), seasonal = list(order = c(1,1,2), period = 24),method="ML", include.mean = TRUE)
Ajuste_INPC_5 <- arima(INPC, order=c(4,1,4), seasonal = list(order = c(1,1,2), period = 24),method="ML", include.mean = TRUE)
Modelo_Ajustado <- c("Ajuste_INPC_1","Ajuste_INPC_2","Ajuste_INPC_3","Ajuste_INPC_4","Ajuste_INPC_5")
#Patrones_pdq_y_PDQ <- c("(0,1,0) (1,0,0)","(1,1,1) (1,0,0)","(2,1,2) (1,0,0)","(3,1,3) (1,0,0)","(4,1,4) (1,0,0)")
AIC <- c(Ajuste_INPC_1$aic, Ajuste_INPC_2$aic, Ajuste_INPC_3$aic, Ajuste_INPC_4$aic,Ajuste_INPC_5$aic)
BIC <- c(BIC(Ajuste_INPC_1), BIC(Ajuste_INPC_2), BIC(Ajuste_INPC_3), BIC(Ajuste_INPC_4),BIC(Ajuste_INPC_5))
Modelo_Ajustado <- as.data.frame(Modelo_Ajustado)
#Patrones_pdq_y_PDQ <- cbind(Patrones_pdq_y_PDQ)
AIC <- as.data.frame(AIC)
BIC <- as.data.frame(BIC)
Summary_stat <- cbind(Modelo_Ajustado, AIC, BIC)
Summary_stat
# AIC
75
index_Summary_stat_Akaike_bajo <- which(Summary_stat[,2] == min(Summary_stat[,2]))
Akaike_mas_bajo <- cbind(Summary_stat[index_Summary_stat_Akaike_bajo,1])
Ajuste_Akaike_mas_bajo <- cbind(Summary_stat[index_Summary_stat_Akaike_bajo,2])
# BIC
index_Summary_stat_BIC_bajo <- which(Summary_stat[,3] == min(Summary_stat[,3]))
BIC_mas_bajo <- cbind(Summary_stat[index_Summary_stat_BIC_bajo,1])
Ajuste_BIC_mas_bajo <- cbind(Summary_stat[index_Summary_stat_BIC_bajo,3])
cat("Modelo de ajuste con valor Akaike más bajo:", Akaike_mas_bajo[1,1])
cat("Valor Akaike:", Ajuste_Akaike_mas_bajo[1,1])
cat("Modelo de ajuste con valor BIC más bajo:", BIC_mas_bajo[1,1])
cat("Valor BIC:", Ajuste_BIC_mas_bajo[1,1])
#### Medidas de precisión de los modelos de ajuste
accuracy_fit_INPC_1 <- forecast::accuracy(Ajuste_INPC_1)
accuracy_fit_INPC_2 <- forecast::accuracy(Ajuste_INPC_2)
accuracy_fit_INPC_3 <- forecast::accuracy(Ajuste_INPC_3)
accuracy_fit_INPC_4 <- forecast::accuracy(Ajuste_INPC_4)
accuracy_fit_INPC_5 <- forecast::accuracy(Ajuste_INPC_5)
MAPE <- c(accuracy_fit_INPC_1[5],accuracy_fit_INPC_2[5],accuracy_fit_INPC_3[5],
accuracy_fit_INPC_4[5],accuracy_fit_INPC_5[5])
ME <- c(accuracy_fit_INPC_1[1],accuracy_fit_INPC_2[1],accuracy_fit_INPC_3[1],
accuracy_fit_INPC_4[1],accuracy_fit_INPC_5[1])
RMSE <- c(accuracy_fit_INPC_1[2],accuracy_fit_INPC_2[2],accuracy_fit_INPC_3[2],
accuracy_fit_INPC_4[2],accuracy_fit_INPC_5[2])
MAE <- c(accuracy_fit_INPC_1[3],accuracy_fit_INPC_2[3],accuracy_fit_INPC_3[3],
accuracy_fit_INPC_4[3],accuracy_fit_INPC_5[3])
MPE <- c(accuracy_fit_INPC_1[4],accuracy_fit_INPC_2[4],accuracy_fit_INPC_3[4],
accuracy_fit_INPC_4[4],accuracy_fit_INPC_5[4])
MASE <- c(accuracy_fit_INPC_1[6],accuracy_fit_INPC_2[6],accuracy_fit_INPC_3[6],
accuracy_fit_INPC_4[6],accuracy_fit_INPC_5[6])
Summary_accuracy <- cbind(Modelo_Ajustado, MAPE,ME,RMSE,MAE,MPE,MASE)
Summary_accuracy
#Summary_accuracy <- cbind(Modelo_Ajustado,MAPE,ME,RMSE,MAE,MPE,MASE)
# Calculando índices para obtener los valores más bajos del resumen estadístico de precisión de los modelos de ajuste
index_MAPE <- which(Summary_accuracy[,2] == min(Summary_accuracy[,2]))
MAPE_mas_bajo <- cbind(Summary_accuracy[index_MAPE ,1])
Valor_MAPE_mas_bajo <- cbind(Summary_accuracy[index_MAPE,2])
cat("Modelo de ajuste con MAPE más bajo:", MAPE_mas_bajo[1,1])
cat("Valor MAPE:", Valor_MAPE_mas_bajo[1,1])
index_ME <- which(Summary_accuracy[,3] == min(Summary_accuracy[,3]))
ME_mas_bajo <- cbind(Summary_accuracy[index_ME ,1])
Valor_ME_mas_bajo <- cbind(Summary_accuracy[index_ME,3])
76
cat("Modelo de ajuste con ME más bajo:", ME_mas_bajo[1,1])
cat("Valor ME:", Valor_ME_mas_bajo[1,1])
index_RMSE <- which(Summary_accuracy[,4] == min(Summary_accuracy[,4]))
RMSE_mas_bajo <- cbind(Summary_accuracy[index_RMSE ,1])
Valor_RMSE_mas_bajo <- cbind(Summary_accuracy[index_RMSE,4])
cat("Modelo de ajuste con RMSE más bajo:", RMSE_mas_bajo[1,1])
cat("Valor RMSE:", Valor_RMSE_mas_bajo[1,1])
index_MAE <- which(Summary_accuracy[,5] == min(Summary_accuracy[,5]))
MAE_mas_bajo <- cbind(Summary_accuracy[index_MAE ,1])
Valor_MAE_mas_bajo <- cbind(Summary_accuracy[index_MAE,5])
cat("Modelo de ajuste con MAE más bajo:", MAE_mas_bajo[1,1])
cat("Valor MAE:", Valor_MAE_mas_bajo[1,1])
index_MPE <- which(Summary_accuracy[,6] == min(Summary_accuracy[,6]))
MPE_mas_bajo <- cbind(Summary_accuracy[index_MPE ,1])
Valor_MPE_mas_bajo <- cbind(Summary_accuracy[index_MPE,6])
cat("Modelo de ajuste con MPE más bajo:", MPE_mas_bajo[1,1])
cat("Valor MPE:", Valor_MPE_mas_bajo[1,1])
index_MASE <- which(Summary_accuracy[,7] == min(Summary_accuracy[,7]))
MASE_mas_bajo <- cbind(Summary_accuracy[index_MASE,1])
Valor_MASE_mas_bajo <- cbind(Summary_accuracy[index_MASE,7])
cat("Modelo de ajuste con MASE más bajo:", MASE_mas_bajo[1,1])
cat("Valor MASE:", Valor_MASE_mas_bajo[1,1])

setwd(ruta_exp)
write.csv(Summary_accuracy, "Medidas de precisión.csv")
write.csv(Summary_stat, "AIC BIC.csv")
setwd(ruta)
#-----------------------------------------------------------------------
# -Proyecto Terminal Modelos de ajuste II AR() MA()
# Auto Regressive Integrate Moving Average
#-----------------------------------------------------------------------
setwd(ruta)
INPC_General <- read.csv('INPC General quincenal.csv', header = TRUE, dec = '.', stringsAsFactors = FALSE)
INPC <- INPC_General
INPC <- ts(INPC, start = c(1988, 1), frequency = 24)
INPC <- INPC[, -1]
INPC <- zoo::na.approx(INPC)
INPC <- INPC
Ajuste_INPC_1 <- arima(INPC, order=c(3,1,1), seasonal = list(order = c(1,1,2), period = 24),method="ML", include.mean = TRUE)
77
Ajuste_INPC_2 <- arima(INPC, order=c(3,1,2), seasonal = list(order = c(1,1,2), period = 24),method="ML", include.mean = TRUE)
Ajuste_INPC_3 <- arima(INPC, order=c(3,1,3), seasonal = list(order = c(1,1,2), period = 24),method="ML", include.mean = TRUE)
Modelo_Ajustado <- c("Ajuste_INPC_1","Ajuste_INPC_2","Ajuste_INPC_3")
#Patrones_pdq_y_PDQ <- c("(3,1,1) (1,1,2)","(3,1,2) (1,1,2)","(3,1,3) (1,1,2)")
AIC <- c(Ajuste_INPC_1$aic, Ajuste_INPC_2$aic, Ajuste_INPC_3$aic)
BIC <- c(BIC(Ajuste_INPC_1), BIC(Ajuste_INPC_2), BIC(Ajuste_INPC_3))
Modelo_Ajustado <- as.data.frame(Modelo_Ajustado)
#Patrones_pdq_y_PDQ <- cbind(Patrones_pdq_y_PDQ)
AIC <- as.data.frame(AIC)
BIC <- as.data.frame(BIC)
Summary_stat <- cbind(Modelo_Ajustado, AIC, BIC)
Summary_stat
# AIC
index_Summary_stat_Akaike_bajo <- which(Summary_stat[,2] == min(Summary_stat[,2]))
Akaike_mas_bajo <- cbind(Summary_stat[index_Summary_stat_Akaike_bajo,1])
Ajuste_Akaike_mas_bajo <- cbind(Summary_stat[index_Summary_stat_Akaike_bajo,2])
# BIC
index_Summary_stat_BIC_bajo <- which(Summary_stat[,3] == min(Summary_stat[,3]))
BIC_mas_bajo <- cbind(Summary_stat[index_Summary_stat_BIC_bajo,1])
Ajuste_BIC_mas_bajo <- cbind(Summary_stat[index_Summary_stat_BIC_bajo,3])
cat("Modelo de ajuste con valor Akaike más bajo:", Akaike_mas_bajo[1,1])
cat("Valor Akaike:", Ajuste_Akaike_mas_bajo[1,1])
cat("Modelo de ajuste con valor BIC más bajo:", BIC_mas_bajo[1,1])
cat("Valor BIC:", Ajuste_BIC_mas_bajo[1,1])
#### Medidas de precisión de los modelos de ajuste
accuracy_fit_INPC_1 <- forecast::accuracy(Ajuste_INPC_1)
accuracy_fit_INPC_2 <- forecast::accuracy(Ajuste_INPC_2)
accuracy_fit_INPC_3 <- forecast::accuracy(Ajuste_INPC_3)
MAPE <- c(accuracy_fit_INPC_1[5],accuracy_fit_INPC_2[5],accuracy_fit_INPC_3[5])
ME <- c(accuracy_fit_INPC_1[1],accuracy_fit_INPC_2[1],accuracy_fit_INPC_3[1])
RMSE <- c(accuracy_fit_INPC_1[2],accuracy_fit_INPC_2[2],accuracy_fit_INPC_3[2])
MAE <- c(accuracy_fit_INPC_1[3],accuracy_fit_INPC_2[3],accuracy_fit_INPC_3[3])
MPE <- c(accuracy_fit_INPC_1[4],accuracy_fit_INPC_2[4],accuracy_fit_INPC_3[4])
MASE <- c(accuracy_fit_INPC_1[6],accuracy_fit_INPC_2[6],accuracy_fit_INPC_3[6])
Summary_accuracy <- cbind(Modelo_Ajustado, MAPE,ME,RMSE,MAE,MPE,MASE)
Summary_accuracy
# Calculando índices para obtener los valores más bajos del resumen estadístico de precisión de los modelos de ajuste
index_MAPE <- which(Summary_accuracy[,2] == min(Summary_accuracy[,2]))
MAPE_mas_bajo <- cbind(Summary_accuracy[index_MAPE ,1])
78
Valor_MAPE_mas_bajo <- cbind(Summary_accuracy[index_MAPE,2])
cat("Modelo de ajuste con MAPE más bajo:", MAPE_mas_bajo[1,1])
cat("Valor MAPE:", Valor_MAPE_mas_bajo[1,1])
index_ME <- which(Summary_accuracy[,3] == min(Summary_accuracy[,3]))
ME_mas_bajo <- cbind(Summary_accuracy[index_ME ,1])
Valor_ME_mas_bajo <- cbind(Summary_accuracy[index_ME,3])
cat("Modelo de ajuste con ME más bajo:", ME_mas_bajo[1,1])
cat("Valor ME:", Valor_ME_mas_bajo[1,1])
index_RMSE <- which(Summary_accuracy[,4] == min(Summary_accuracy[,4]))
RMSE_mas_bajo <- cbind(Summary_accuracy[index_RMSE ,1])
Valor_RMSE_mas_bajo <- cbind(Summary_accuracy[index_RMSE,4])
cat("Modelo de ajuste con RMSE más bajo:", RMSE_mas_bajo[1,1])
cat("Valor RMSE:", Valor_RMSE_mas_bajo[1,1])
index_MAE <- which(Summary_accuracy[,5] == min(Summary_accuracy[,5]))
MAE_mas_bajo <- cbind(Summary_accuracy[index_MAE ,1])
Valor_MAE_mas_bajo <- cbind(Summary_accuracy[index_MAE,5])
cat("Modelo de ajuste con MAE más bajo:", MAE_mas_bajo[1,1])
cat("Valor MAE:", Valor_MAE_mas_bajo[1,1])
index_MPE <- which(Summary_accuracy[,6] == min(Summary_accuracy[,6]))
MPE_mas_bajo <- cbind(Summary_accuracy[index_MPE ,1])
Valor_MPE_mas_bajo <- cbind(Summary_accuracy[index_MPE,6])
cat("Modelo de ajuste con MPE más bajo:", MPE_mas_bajo[1,1])
cat("Valor MPE:", Valor_MPE_mas_bajo[1,1])
index_MASE <- which(Summary_accuracy[,7] == min(Summary_accuracy[,7]))
MASE_mas_bajo <- cbind(Summary_accuracy[index_MASE,1])
Valor_MASE_mas_bajo <- cbind(Summary_accuracy[index_MASE,7])
cat("Modelo de ajuste con MASE más bajo:", MASE_mas_bajo[1,1])
cat("Valor MASE:", Valor_MASE_mas_bajo[1,1])

setwd(ruta_exp)
write.csv(Summary_accuracy, "Medidas de precisión_3.csv")
write.csv(Summary_stat, "AIC BIC_3.csv")
setwd(ruta)
#-----------------------------------------------------------------------
# -Proyecto Terminal Modelos de ajuste III AR() MA()
# Auto Regressive Integrate Moving Average
#-----------------------------------------------------------------------
setwd(ruta)
INPC_General <- read.csv('INPC General quincenal.csv', header = TRUE, dec = '.', stringsAsFactors = FALSE)
INPC <- INPC_General
79
INPC <- ts(INPC, start = c(1988, 1), frequency = 24)
INPC <- INPC[, -1]
INPC <- zoo::na.approx(INPC)
INPC <- INPC
Ajuste_INPC_1 <- arima(INPC, order=c(3,1,1), seasonal = list(order = c(1,1,2), period = 24),method="ML", include.mean = TRUE)
Ajuste_INPC_2 <- arima(INPC, order=c(1,1,2), seasonal = list(order = c(1,1,2), period = 24),method="ML", include.mean = TRUE)
Ajuste_INPC_3 <- arima(INPC, order=c(2,1,2), seasonal = list(order = c(1,1,2), period = 24),method="ML", include.mean = TRUE)
Modelo_Ajustado <- c("Ajuste_INPC_1","Ajuste_INPC_2","Ajuste_INPC_3")
#Patrones_pdq_y_PDQ <- c("(3,1,1) (1,1,2)","(1,1,2) (1,1,2)","(2,1,2) (1,1,2)")
AIC <- c(Ajuste_INPC_1$aic, Ajuste_INPC_2$aic, Ajuste_INPC_3$aic)
BIC <- c(BIC(Ajuste_INPC_1), BIC(Ajuste_INPC_2), BIC(Ajuste_INPC_3))
Modelo_Ajustado <- as.data.frame(Modelo_Ajustado)
AIC <- as.data.frame(AIC)
BIC <- as.data.frame(BIC)
Summary_stat <- cbind(Modelo_Ajustado, AIC, BIC)
Summary_stat
# AIC
index_Summary_stat_Akaike_bajo <- which(Summary_stat[,2] == min(Summary_stat[,2]))
Akaike_mas_bajo <- cbind(Summary_stat[index_Summary_stat_Akaike_bajo,1])
Ajuste_Akaike_mas_bajo <- cbind(Summary_stat[index_Summary_stat_Akaike_bajo,2])
# BIC
index_Summary_stat_BIC_bajo <- which(Summary_stat[,3] == min(Summary_stat[,3]))
BIC_mas_bajo <- cbind(Summary_stat[index_Summary_stat_BIC_bajo,1])
Ajuste_BIC_mas_bajo <- cbind(Summary_stat[index_Summary_stat_BIC_bajo,3])
cat("Modelo de ajuste con valor Akaike más bajo:", Akaike_mas_bajo[1,1])
cat("Valor Akaike:", Ajuste_Akaike_mas_bajo[1,1])
cat("Modelo de ajuste con valor BIC más bajo:", BIC_mas_bajo[1,1])
cat("Valor BIC:", Ajuste_BIC_mas_bajo[1,1])
#### Medidas de precisión de los modelos de ajuste
accuracy_fit_INPC_1 <- forecast::accuracy(Ajuste_INPC_1)
accuracy_fit_INPC_2 <- forecast::accuracy(Ajuste_INPC_2)
accuracy_fit_INPC_3 <- forecast::accuracy(Ajuste_INPC_3)
MAPE <- c(accuracy_fit_INPC_1[5],accuracy_fit_INPC_2[5],accuracy_fit_INPC_3[5])
ME <- c(accuracy_fit_INPC_1[1],accuracy_fit_INPC_2[1],accuracy_fit_INPC_3[1])
RMSE <- c(accuracy_fit_INPC_1[2],accuracy_fit_INPC_2[2],accuracy_fit_INPC_3[2])
MAE <- c(accuracy_fit_INPC_1[3],accuracy_fit_INPC_2[3],accuracy_fit_INPC_3[3])
MPE <- c(accuracy_fit_INPC_1[4],accuracy_fit_INPC_2[4],accuracy_fit_INPC_3[4])
MASE <- c(accuracy_fit_INPC_1[6],accuracy_fit_INPC_2[6],accuracy_fit_INPC_3[6])
Summary_accuracy <- cbind(Modelo_Ajustado, MAPE,ME,RMSE,MAE,MPE,MASE)
Summary_accuracy
80
#Summary_accuracy <- cbind(Modelo_Ajustado,MAPE,ME,RMSE,MAE,MPE,MASE)
# Calculando índices para obtener los valores más bajos del resumen estadístico de precisión de los modelos de ajuste
index_MAPE <- which(Summary_accuracy[,2] == min(Summary_accuracy[,2]))
MAPE_mas_bajo <- cbind(Summary_accuracy[index_MAPE ,1])
Valor_MAPE_mas_bajo <- cbind(Summary_accuracy[index_MAPE,2])
cat("Modelo de ajuste con MAPE más bajo:", MAPE_mas_bajo[1,1])
cat("Valor MAPE:", Valor_MAPE_mas_bajo[1,1])
index_ME <- which(Summary_accuracy[,3] == min(Summary_accuracy[,3]))
ME_mas_bajo <- cbind(Summary_accuracy[index_ME ,1])
Valor_ME_mas_bajo <- cbind(Summary_accuracy[index_ME,3])
cat("Modelo de ajuste con ME más bajo:", ME_mas_bajo[1,1])
cat("Valor ME:", Valor_ME_mas_bajo[1,1])
index_RMSE <- which(Summary_accuracy[,4] == min(Summary_accuracy[,4]))
RMSE_mas_bajo <- cbind(Summary_accuracy[index_RMSE ,1])
Valor_RMSE_mas_bajo <- cbind(Summary_accuracy[index_RMSE,4])
cat("Modelo de ajuste con RMSE más bajo:", RMSE_mas_bajo[1,1])
cat("Valor RMSE:", Valor_RMSE_mas_bajo[1,1])
index_MAE <- which(Summary_accuracy[,5] == min(Summary_accuracy[,5]))
MAE_mas_bajo <- cbind(Summary_accuracy[index_MAE ,1])
Valor_MAE_mas_bajo <- cbind(Summary_accuracy[index_MAE,5])
cat("Modelo de ajuste con MAE más bajo:", MAE_mas_bajo[1,1])
cat("Valor MAE:", Valor_MAE_mas_bajo[1,1])
index_MPE <- which(Summary_accuracy[,6] == min(Summary_accuracy[,6]))
MPE_mas_bajo <- cbind(Summary_accuracy[index_MPE ,1])
Valor_MPE_mas_bajo <- cbind(Summary_accuracy[index_MPE,6])
cat("Modelo de ajuste con MPE más bajo:", MPE_mas_bajo[1,1])
cat("Valor MPE:", Valor_MPE_mas_bajo[1,1])
index_MASE <- which(Summary_accuracy[,7] == min(Summary_accuracy[,7]))
MASE_mas_bajo <- cbind(Summary_accuracy[index_MASE,1])
Valor_MASE_mas_bajo <- cbind(Summary_accuracy[index_MASE,7])
cat("Modelo de ajuste con MASE más bajo:", MASE_mas_bajo[1,1])
cat("Valor MASE:", Valor_MASE_mas_bajo[1,1])

setwd(ruta_exp)
write.csv(Summary_accuracy, "Medidas de precisión_III.csv")
write.csv(Summary_stat, "AIC BIC_III.csv")
setwd(ruta)
```


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



