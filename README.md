# Tesis 

##### En este proyecto iré subiendo los avances de mi tesis para recibir el grado de Economista, el proyecto es un pronóstico de la inflación para el año 2019 en México; analizando el INPC General con el lenguaje de programación estadística R. 

##### Iré reportando los resultados en el archivo *Reporte del pronóstico del INPC General.html*.

# En cuanto al reporte:

El archivo es extenso para que GitHub lo pueda abrir, así que se tiene que descargar en crudo (raw). 
Al descargar se abre una ventana en el navegador que contiene el código HTML del reporte, 
se selecciona todo el texto con Ctrl + a, después se copia con Ctrl + c.

Después de descargar te recomiendo que tengas instalado en tu computador un **IDE** ***(Entorno de Desarrollo Integrado)***, 
yo tengo instalado ***Visual Studio Code***; estando dentro del IDE generas un nuevo archivo HTML, y ahí pegas el código HTML,
ese código se encuentra codificado en *UTF-8*, por lo que los acentos y caracteres latinoamericanos aparecerán con un signo de 
interrogación **"?"**.

Para evitar que ese signo sustituya los caracteres latinos vas a la parte inferior derecha del IDE, en mi caso
Visual Studio Code, y das click en el botón de selección de codificación; ***[UTF-8]***, se abrirá una lista de opciones
de esa lista seleccionas *"Save with Encoding"* y selecionas la codificación ***Western (ISO 8859-1)***, entonces los 
signos de interrogación que sustituían a los caracteres latinos desaparecerán y aparecerán los caracteres adecuados.

Guardas el archivo HTML dentro de un directorio en el computador y lo nombras como *"Reporte de Tesis INPC General"*, vas a la 
ruta del directorio y das doble click al archivo: Reporte de Tesis INPC General, abrirá una pestaña en tu navegador web predeterminado.

# En cuanto a los resultados del pronóstico:

Generé dos modelos de ajuste del INPC General y mediante las funciones de autocorrelación y autocorrelación parcial obtuve los siguiente patrones ARIMA y patrones estacionales:

  * **Modelo 1** AR(1)I(2)MA(1) junto con el patrón de residuos (P,D,Q): AR(2)I(0)MA(0)
  * **Modelo 2** AR(6)I(2)MA(6) junto con el patrón de residuos (P,D,Q): AR(2)I(0)MA(0)

##                                           Gráfica 1
<img src="https://github.com/StefanoSoriano/Tesis/blob/master/imágenes/Inflación%20inmediata%202018.png" alt="drawing"/>

###### Fuente: Elaboración propia con datos del BIE del INEGI.
