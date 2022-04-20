# Cajamar-Water-Footprint
## UNIVERSITYHACK 2022 DATATHON
### Universitat Autònoma de Barcelona
### OvoCAD
- Sergi Cantón
- Bernat Espinet
- Marc Llopart

## Introducción

El reto de Cajamar Water Footprint consiste en un problema de predicción. La idea inicial es predecir la demanda de agua potable para facilitar la planificación, diseño y operación de su suministro. Para ello, tenemos un dataset correspondiente a 2747 contadores de la costa de la Comunidad Valenciana, los cuáles pertenecen a una variedad de edificios, yendo desde viviendas a industrias. 

## EDAs
Cualquier problema relacionado con la ciencia de datos incluye ciertos procesos fundamentales, entre los que encontramos el entendimiento del problema y el análisis exploratorio de las variables. Estos dos pasos son clave para realizar un buen análisis y obtener un resultado acorde a los requisitos. 

Para realizar las predicciones se nos dio un dataset con seis columnas distintas, cuya información era la siguiente. En la primera columna encontramos los diversos identificadores de los contadores. La siguiente columna nos daba información sobre el momento de la lectura del contador, estas eran lecturas *entre-diarias* ya que teníamos varias lecturas por un mismo día difiriendo en las horas. A partir de aquí las columnas siguientes nos daban la medida del contador, tanto la parte entera como la decimal por separado y lo mismo ocurría con el consumo calculado.

Una vez tuvimos los datos decidimos hacer pequeñas visualizaciones para ver como estaban organizados, la cual cosa nos facilitó a la comprensión del problema y poder hacer un buen análisis.

## Data cleaning
Una vez comprendido el problema había que hacer unas modificaciones en el dataset para hacer más fácil el estudio. Para ello, juntamos las medidas decimales con las enteras en una sola columna. A continuación, realizamos las primeras gráficas para ver cómo se comportaban los datos. Sin embargo, antes de esto necesitamos hacer la lectura de los datos en *chunks* debido al gran volumen de datos. 

Después, juntamos las medidas de cada día en un solo valor por día, ya que esto simplifica el análisis y permite hacer un estudio que ayuda a acercarnos más a la solución pedida. 

Una vez ya tuvimos el dataset estructurado tuvimos que limpiarlo. Para ello fue preciso mirar qué problemas había, ya fueran valores nulos, valores negativos o falta de valores. Para el caso de los valores nulos vimos que solo se encontraban en la parte decimal, así que decidimos poner todos esos valores a cero, debido a que semejante cambio no iba a marcar una gran diferencia. El caso de los valores negativos nos molestó bastante al principio y le intentamos encontrar el sentido, aún así decidimos que la mejor opción era poner el valor a cero y más adelante ya lo acabaríamos modificando. Para el último caso, la falta de valores, miramos si teníamos datos de todos los días. En caso de no tener datos para todos los días lo que hicimos fue crearlos inicializando esos valores a cero.

Después de rellenar y ajustar los valores del dataset fue necesario limpiarlo de *outliers*. Para ello utilizamos la interpolación con el día previo y el posterior para suavizar valores. Más tarde hicimos uso de la libreria de machine learning *LocalOutlierFactor* de *sckit-learn*. Usamos varios valores de contaminación en un intervalo del 5% i un 7%.

## Tratamiento de Series Temporales
Una serie temporal es una sucesión de valores ordenados cronológicamente, espaciados con intervalos, ya sean iguales o no. Justamente nuestro problema se adecua a esta definición, y por ello lo primero que hicimos fue documentarnos ya que no teníamos gran conocimiento en el tratamiento de estas.

Lo primero que hicimos, aunque ya lo hemos mencionado en el apartado anterior, fue juntar los datos por días para tratar el problema como una serie temporal con un intervalo diario.

Una vez tuvimos los datos organizados para utilizar algoritmos de machine learning era momento de informarse sobre qué se podía usar y cómo funcionaba. La organización recomendaba la librería de *Facebook, Prophet*.

La librería *Prophet* permite trabajar con las series temporales donde no hay tendencias lineales y hay comportamientos ya sean anuales, mensuales, diarios o por estaciones. Además es especialmente buena para trabajar con la falta de datos y valores atípicos. Inicialmente nos pareció una librería ideal para resolver nuestro problema; no obstante, al usarla nos encontramos con diversos problemas. Algunos de ellos fueron la obtención de valores negativos al predecir, a lo que no le encontramos sentido. Al disponer de más posibles librerías, decidimos avanzar con otros métodos esperando que nos diesen resultados mejores o más lógicos. 

Nos decidimos por usar un modelo de regresión de la librería *sckit-learn* para realizar *forecasting*. Para este modelo las predicciones van asociadas a un período temporal que definimos previamente. Nosotros decidimos definir este valor como múltiplo de 7, ya que son los días que tiene una semana y consideramos que podía haber una relación entre el mismo día de dos semanas diferentes.

## Selección del modelo
Una vez ya supimos cómo íbamos a tratar las series temporales y tuvimos los datos preparados fue momento de probar diversos modelos y escoger aquel que obtuviera un error menor. Para valorar los modelos decidimos usar los datos del 2019 como conjunto de entrenamiento y los datos del 2020 como conjunto de validación. De este modo teníamos una métrica de referencia para mejorar nuestro modelo. En concreto usamos el MSE medio de todos los días del conjunto de validación para valorar los diversos modelos. 
Lo primero que hicimos fue comparar dos tipos de procesados distintos, uno que interpolaba los datos usando el mismo día de la semana anterior y posterior (1), y otro que lo hacía con el valor del día siguiente y anterior (2).

| Método  |Métrica (en Millones)|
| ------------- |------------|
| Método 1  | 3,514514634 |
| Método 2  | 2,707845757 |

Después de obtener estos resultados decidimos seguir adelante con el segundo procesado correspondiente al segundo método.

Una vez decidido el procesado de los datos fue momento de probar diferentes parámetros y modelos para ver cuál funcionaba mejor. Para ello empezamos probando varios valores distintos para los *lags*, parámetro correspondiente a la ventana temporal que mira el modelo para hacer la predicción. Más adelante hicimos un *Hyperparameter Search* para agilizar la búsqueda de parámetros y encontrar aquellos que se ajustaban mejor a cada contador.

El modelo que usamos principalmente fue un *Random Forest* ya que informándonos vimos que lo consideraban de las mejores opciones para las series temporales. Otro modelo que probamos fue el *LightGBM* y obtuvimos buenos resultados. Al ser computacionalmente costoso, dividimos el proceso por partes. De este modo obtuvimos 8 particiones distintas que correspondían a un grupo de contadores distintos. Finalmente hemos escogido el modelo que se ajustaba mejor a cada partición, ya que así consideramos que obtenemos el mejor modelo posible.

| Parte | Modelo 1 | Modelo 2 | Modelo 3 | Modelo 4 | Modelo 5 |
| ----- | -------- | -------- | -------- | -------- | -------- |
| 1     | **2,353628705** | 2,360533764 | 3,382935317 | 4,24307438 | 7,132589433 |
| 2     | 0,5184698703 | 0,5196631877 | **0,4630201376** | 0,6139791434 | 0,6485693103 |
| 3     | 5,00167825 | **4,626116225** | 4,638320053 | 4,803988753 | 4,981631147 |
| 4     | 2,250793872 | 4,360722731 | **2,229220031** | 966,5938699 | 497,1228895 |
| 5     | 3,701685004 | 2,612124662 | 4,467443093 | **2,567289292** | 3,337135513 |
| 6     |**0,3775172978** |  0,3847100528  | 0,414105715 | 0,4336035941  | 0,4024841649 |
| 7     | 0,7153604542 |  0,6729787382  | 0,4319758044 |**0,4164548138**| 0,5591924527 |
| 8     | 2,589590463 | 2,578274548 | 2,834078326 | 574339318773 | **1,699845803** |
| TOTAL     | 2,147324603 |  2,223460301 |  2,312407032 | 61887309317 | 64,57321421 |

En esta tabla el el modelo 1 hace referencia a un RandomForestRegressor con un factor de contaminación de los datos del 5% y los valores 14,28,42,56 para los *lags*. Para el resto de modelos hacemos uso de una ventana temporal con los valores $7,14,21,28,35,42,49,56$. El modelo 2 y 3 corresponden a un RandomForestRegressor con un factor de contaminación de los datos del 5% y 7% respectivamente. El modelo 4 hace referencia a un Standard Scaler con regularización Ridge y el modelo 5 hace referencia al LightGBM con 3 árboles de decisión.

Decidimos usar esa ventana temporal ya que coincidimos que lo mejor era comparar con el mismo día de las semanas previas ya que el comportamiento entre semanas tenia cierta correlación. No consideramos mirar más allá de los 56 días ya que no vemos relación alguna.

Destacamos que encontramos valores anormales en el modelo 4 y 5. Lo podemos ver en el total y en la partición 4 en ambos modelos, y en la partición 8 en el modelo 4. Estos valores vienen dados debido a que los modelos usados encuentran ciertas tendencias en esas particiones, y las prediccones hechas no se ajustan a los valores que tenemos. Esto ocurre en el modelo 4 ya que usamos un modelo lineal con una penalización Ridge y en el modelo 5 por falta de árboles de decisón, ya que 3 són muy pocos. Pero debido al alto coste computacional no pudimos sacar valores con más árboles.

## Instrucciones de uso

En el directorio *data* encontramos los ficheros divididos en chunks para hacer las lecturas correspondientes y la limpieza correcta de los datos. El fichero *DF.csv* corresponde al dataset ya limipio y preparado para aplicar diversos modelos. 

El script de predicción contiene varias celdas correspondientes a la ejecución de cada parte del modelo, se pueden correr por separado, en caso de disponer de *AWS SageMaker Studio Lab* se pueden ejecutar en paralelo. 

