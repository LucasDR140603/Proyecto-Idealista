# Proyecto Idealista

## NOTEBOOK 1

### CONTEXTO Y ANÁLISIS PREVIO
Los csv con los que se han trabajado contienen datos de pisos de obtenidos de :
1. Madrid y Barcelona
2. Madrid y Valencia

Como ambos CVs contienen datos de Madrid, se decide trabajar con un único _DataFrame_ creado con  `.concat([df1,df2],ignore_index=True).drop_duplicates()`<br>
Además se eliminan los posibles duplicados de filas (500 filas aproximadamente).


### ANÁLISIS DEL TIPO DE COLUMNAS

Se hace un análisis y clasificación preliminar de cada columna.
+ Hay 46 columnas.
+ Se definen unas series para analizar: 
    + `unicos` -> Listado de la cantidad de valores únicos de cada columna.
    + `nnans` -> Listado de la cantidad de Nans en existentes en cada columna ordenados de forma descendente.
+ Con las series creadas se crean los índices:
    + `constantes` ->  Listado de nombres de columnas que tienen un único valor en todo el _DataFrame_.
    + `binarios` -> Listado de nombres de columnas que tienen 2 únicos valores.
    + `nans` -> Listado de nombres de columnas que contienen NaNs.

Con estos datos se decide eliminar las columnas listadas en `constantes` una vez averiguado que su valor es "HOME" y "SALE". Por lo que se confirma que el conjunto del _DataFrame_ representa valores de venta de viviendas.

Además se confirma que sólo existen 4 columnas con NaNs, el impacto de estos se estudiará más adelante.

Se puede ver que las columnas listadas en `binarios` son del tipo `Int64` pero en realidad únicamente contienen valores de tipo "0" o "1" por lo que estos valores se pasaron a tipo booleano.
De entre las columnas contenidas en `binarios` se tratan de forma distinta las llamadas `BUILTTYPEID_1`, `BUILTTYPEID_2` y `BUILTTYPEID_3`. Para seguir reduciendo el número de columnas se decide transformar las 3 columnas en una única llamada `BUILTTYPEID`.

La columna `PERIOD` sólo tiene 4 valores: "201803", "201806", "201809" y "201812". Por lo que se deduce que los datos corresponden a un período de 2018 y se divide por trimestres. Se desconoce si estos datos correponden a los pisos vendidos o publicados u otro criterio. 

### ANÁLISIS DE DATOS ANÓMALOS.

Al realizar una análisis visual de todas las columnas numéricas se detectan y tratan las siguientes anomalías:
+ En las columas de "distancia a" (`DISTANCE_TO_CITY_CENTER`,`DISTANCE_METRO` y `DISTANCE_TO_STREET`):
    + Se detecta que hay 1 fila que se encuentra a más de 400 kms. Como esta fila no corresponde a ninguna ciudad de las estudiadas se decide eliminarla por considerarse un dato no confiable. Es posible que los datos sean correctos pero no se debe tener en cuenta para trabajar con datos representativos de las ciudades estudiadas.

+ Análisis sobre las columnas `ISPARKINGSPACEINCLUDEDINPRICE` y `PARKINGSPACEPRICE`
    + Todas las filas con `HASPARKINGSPACE` en "True" tienen `ISPARKINGSPACEINCLUDEDINPRICE` en "True" también, en otras palabras, todos los pisos con plaza de aparcamiento, la tienen incluída en el precio.<br>
    + Cuando `ISPARKINGSPACEINCLUDEDINPRICE` es "True", en casi todas las filas (31892) el valor de `PARKINGSPACEPRICE` es de 1, lo que parece indicar que 1 es un valor por defecto cuando `ISPARKINGSPACEINCLUDEDINPRICE` es TRUE.<br>
    Pero existen 13 valores mayores a 1 que curiosamente acaban en 1 (16501 €, por ejemplo) esto lleva a pensar que cuando se crearon los datos se añadió automaticamente el valor 1 a una columna en la que se esperaba que el valor fuese nulo.<br>
    Por homogeneizar el df se van cambiar los valores a 1 al considerarlos errores.
    
    + Siguiendo con la columna `PARKINGSPACEPRICE` en ella se detecta un numero elevado de _outliers_.<br>
    En el DataFrame el precio del parking como valor 1 representa el 97,5%.<br>
    Pero lo más curioso es que el 100% de los inmuebles que tienen marcado un precio superior a 1 tambien tiene como valor de "HASPARKINGSPACE" un False.<br>
    Estos valores tienen un precio muy raro ya que en algunos casos tiene un precio superior al valor del propio piso y de media representan un 92% del valor de piso.<br>
    Por estos motivos se decide transformar estas filas en "PARKINGSPACEPRICE" = 1.<br>
    Como resultado de esto el 100% del valor de la columna PARKINGSPACEPRICE es 1. Lo que la hace inútil pero ya de por si los datos eran muy poco fiables.




### ANÁLISIS DEL TIPO DE NANs.

Se detectan que las columnas `CONSTRUCTIONYEAR` , `FLATLOCATIONID` , `FLOORCLEAN` y `CADASTRALQUALITYID` contienen NaNs.

El caso más llamativo es el de `CONSTRUCTIONYEAR` ya que casi 48% de los valores son Nans.
Al hacer una comparativa con el columna `CADCONSTRUCTIONYEAR`, que no contienen NaNs, se determina que es posible asumir que la columna `CADCONSTRUCTIONYEAR` es muy similar (99% similar) a la columna `CONSTRUCTIONYEAR` en aquellas filas donde contiene datos por lo que se decide descartar la columna `CONSTRUCTIONYEAR` en favor de `CADCONSTRUCTIONYEAR`.

En el caso de la columna `FLATLOCATIONID` (Aprox. 10% de valores NaN). Esta es una columna que por lo general contiene valores "1" o "2" haciendo una tabla de correlación `df[df["FLATLOCATIONID"].notna()].corr(numeric_only=True)['FLATLOCATIONID'].abs().sort_values(ascending=False).iloc[1:]` No se obtiene ningún valor alto de correlación con ninguna otra columna por lo que se entiende que los NaNs existentes son completamente aleatorios.

En el caso de la columna `FLOORCLEAN` (9829 NaNs, el 5% de las filas) la columna tiene valores entre "-1" y "11".
Representa el número de planta del piso.
Haciendo una tabla de correlación `df[df["FLOORCLEAN"].notna()].corr(numeric_only=True)['FLOORCLEAN'].abs().sort_values(ascending=False).iloc[1:]` No se obtiene ningún valor alto de correlación con ninguna otra columna por lo que se entiende que los NaNs existentes son completamente aleatorios.
Se podría pensar que los NaNs podrían venir de los pisos que son especiales como los "estudios" (`ISSTUDIO`), los duplex (`ISDUPLEX`) o las entreplantas (`ISINTOPFLOOR`) pero al contarlos con `df[(df['ISDUPLEX']== True) | (df['ISINTOPFLOOR']== True) | (df['ISSTUDIO']== True) ]['FLOORCLEAN'].isna().sum()` se obtiene un total de 481. Aproximadamente un 5% de los NaNs por lo que refuerza la idea de que la distribución los valores NaN de esta columna es aleatoria y por tanto confiables.

Por último la columna `CADASTRALQUALITYID` solo contiene un único valor de NaN.

### ANÁLISIS DE PRICE, UNITPRICE, CONSTRUCTEDAREA Y CORRELACIONES

Una vez limpiados los datos se analiza la forma y distribución de las columnas de PRICE, UNITPRICE y CONSTRUCTEDAREA tanto en escala normal como en logarítmica. Se llega a la conclusión de que la distribución de la colmna UNITPRICE varía mucho dependiendo de la ciudad. Se ve que existe una correlación fuerte entre CONSTRUCTEDAREA y PRICE, por lo que se decide analizar en profundidad en el siguiente notebook. También se ve que en el caso de Madrid se ve un impacto de la latitud sobre el precio, por la forma del centro de la ciudad.

Se crea el csv datos_limpios.csv como output de este notebook

## NOTEBOOK 2

Trabajamos con el csv de datos_limpios.csv.
Analizamos la columna `CADASTRALQUALITYID` y según su análisis con respecto al precio unitario parece que a menor valor mayor es la calidad, sin embargo los valores 0 son abundantes y no presentan datos extraños en las demás columnas, por lo que no parece que sean errores.
Por otro lado la columna `BUILTTYPEID` va del 1 al 3 a mayor valor mayores pueden llegar a ser los precios, aunque todos tienen mínimos similares.

### REGRESIÓN LINEAL

Elegimos las columnas `LOG_CONSTRUCTEDAREA` y `LOG_PRICE` como variables predictora y objetivo respectivamente, debido a su alta correlación y relativa normalidad. Al crear, entrenar y probar el modelo. Vemos que:
- MAE: 0.44
- MSE: 0.29
- RMSE: 0.54 -> Un poco más alto que el mean absolute error, por lo que tenemos outliers.
- R2: 0.46 -> No llega a 1 pero tampoco es 0, por lo que es el modelo es mejor que usar el promedio pero no es perfecto.

El análisis de residuos muestra una curva sobre la línea de la regresión lineal y desviaciones más pronunciadas en los extremos, lo que nos indica que el modelo está sesgado y con outliers.

La gráfica de distribución de los residuos no encaja con la curva de normalidad teórica y el test Kolmogorov-Smirnov (con los residuos estandarizados) nos dá un pvalue muy inferior a 0.05, por lo que no son normales. Es decir, los errores no son solo ruído.

### CONSTRASTE DE HIPÓTESIS

Analizando el precio medio de los inmuebles con y sin ascensor llegamos a la conclusión de que a priori estas medias son distintas y queremos contrastarlo. Para ello se ha comprobado la normalidad de los precios y las similitud de varianzas. Como resultado de la normalidad de los LOG_PRICE de los pisos tanto si tienen o no tienen ascensor se asemeja mucho a una normal pero sigue sin ser una normal estadísticamente. Se puede afirmar sin ningún tipo de duda que las varianzas son distintas, esto hace no recomendable el uso del ttest, aún así se decide hacerlo dando como resultado que se descarta la hipótesis nula del ttest.
Como las varianzas son distintas los resultados del ttest no son determinantes pero afirmamos que las medias de precio son distintas.

Se cogen las columnas de log-price filtradas en Madrid y Barcelona para analizar si lo que se observa de que las medias de los precios entre las 2 ciudades son iguales es cierto o no. Se fija con hipótesis nula que son iguales, se realizan las mismas pruebas previas que en el apartado anterior, se observa que los resultados previos las distribuciones de log-precios de las 2 ciudades son prácticamente normales sin llegar a serlo. Por desgracia las varianzas son distintas sin llegar a error y como resultado del ttest se obtiene que se puede descartar la hipótesis nula. Por lo que seguramente a pesar de que el ttest no es el mejor método debido a la diferencia de varianzas lo más razonable es pensar que la igualdad de las medias de los precios es casual.