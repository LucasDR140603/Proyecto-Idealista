# Proyecto Idealista

### CONTEXTO Y ANÁLISIS PREVIO
Los csv con los que se han trabajado contienen datos de pisos de obtenidos de :
1. Madrid y Barcelona
2. Madrid y Valencia

Como ambos CVs contienen datos de Madrd, se decide trabajar con un único _DataFrame_ creado con  `.concat([df1,df2],ignore_index=True).drop_duplicates()`<br>
Además se eliminan los posibles duplicados de filas (500 filas aproximadamente).


### ANÁLISIS DEL TIPO DE COLUMNAS

Se hace una análisis y clasificación preliminar de cada columna.
+ Hay 46 columnas.
+ Se defiene unas series para analizar: 
    + `unicos` -> Listado de la cantidad de valores únicos de cada columna.
    + `tipos` -> Listado del tipo d cada columna.
    + `nnans` -> Listado de la contidad de Nans en exstentes en cada columna ordenados de forma descendente.
+ Con las series se creaadas se crean los indices:
    + `constantes` ->  Listado de nombres de columnas que tiene un único valor en todo el _DataFrame_.
    + `binarios` -> Listado de nombres de columnas que tiene 2 únicos valores.
    + `nans` -> Listado de nombres de columnas que contienen NaNs.

Con estos datos se decide eliminar las columnas listadas en `constantes` una vez averiguado que su valor es "HOME" y "SALE". Por lo que se confirma que el conjutno del _DataFrame_ representa valores de venta de viviendas.

Además se confirma que sólo existen 4 columnas con NaNs, el impacto de estos se estudiará más adelante.

Se puede ver que las columnas listas den `binarios` son del tipo `Int64`pero realidad únicemente contienen valores de tipo "0" o "1" por lo que en realidad estos valores se pueden pasar a tipo booleano, aun que se decide no dejarlas como están.
De entre las columnas contenidas en `binarios` se tratan de forma distinta las llamadas `BUILTTYPEID_1`, `BUILTTYPEID_2` y `BUILTTYPEID_3`. Para seguir reduciendo el número de columnas se decide transformar las 3 columnas en una única llamada `BUILTTYPEID`.

La columna `PERIOD` sólo tiene 4 valores: "201803", "201806", "201809" y "201812". Por lo que se deduce que los datos corresponden a un periodo de 2018 y se divide por trimestres. Se desconoce si estos datos correponede los pisos vendidos o publicados u otro criterio. 

### ANÁLISIS DE DATOS ANÓMALOS.

Al realizar una análisis visual de todas las columnas numéricas se detectan y tratan las siguientes anomalías:
+ En las columas de "distancia a" (`DISTANCE_TO_CITY_CENTER`,`DISTANCE_METRO` y `DISTANCE_TO_STREET`):
    + Se detecta que hay 1 fila que se encuentra a más de 400 kms. Como esta fila no corresponde a ninguna ciudad de las estudiadas se decide eliminar esta fila por considerarse un dato no confiable. Es posible que los datos sean correctos pero no se debe tener encuenta para trabajar con datos representativos de las ciudades estudiadas.

+ Análisis sobre las columnas `ISPARKINGSPACEINCLUDEDINPRICE` y `PARKINGSPACEPRICE`
    + Cuando`ISPARKINGSPACEINCLUDEDINPRICE` es "True", en casi todos las filas (31892) el valor de `PARKINGSPACEPRICE` es de 1, lo que parece indicar que 1 es un valor por defecto cuendo `ISPARKINGSPACEINCLUDEDINPRICE` es TRUE.<br>
    Pero existen 13 valores mayores a 1 que curiosamente acaban en 1 (16501 €, por ejemplo) esto lleva a pensar que cuando se crearon los datos se añadió automaticamente el valor 1 a una columna en la que se esperaba que el valor fuese nulo.<br>
    Por homogeneizar el df se van cambiar los valores a 1 al considerarlos errores.
    
    + Siguiendo con la columna `PARKINGSPACEPRICE` en ella se detectan un numero elevado de _outliers_.<br>
    En el DataFrame el precio del parking como valor 1 representa el 97,5%.<br>
    Pero lo más curioso es que el 100% de los inmuebles que tienen marcado un precio superior a 1 tambien tiene como valor de "HASPARKINGSPACE" un False.<br>
    Estos valores tienen un precio muy raro ya que en algunos casos tiene un precio superior al valor del propio piso y de media representan un 92% del valor de piso.<br>
    Por estos motivos se decide transformar estas filas en "PARKINGSPACEPRICE" = 1.<br>
    Como resultado de esto el 100% del valor de la columna PARKINGSPACEPRICE es 1. Lo que la hace inutil pero ya de por si los datos eran muy poco fiables.




### ANÁLISIS DEL TIPO DE NANs.

Se detectan que las columnas `CONSTRUCTIONYEAR` , `FLATLOCATIONID` , `FLOORCLEAN` y `CADASTRALQUALITYID` contienen NaNs.

El caso más llamativo es el de `CONSTRUCTIONYEAR` ya que casi 48% de los valores son Nans.
Al hacer una comparativa con el columna `CADCONSTRUCTIONYEAR`, que no contienen NaNs, se determina qye es posible asumir que la columna `CADCONSTRUCTIONYEAR` es muy silimiar (99% similar) a la columna `CONSTRUCTIONYEAR` en aquellas filas donde contiene datos por lo que se decide descartar la columna `CONSTRUCTIONYEAR` en favor de `CADCONSTRUCTIONYEAR`.

En el caso de la columna `FLATLOCATIONID` (Aprox. 10% de valores NaN). Esta es una columan que por lo general contiene valores "1" o "2" haciendo una tabla de correlación `df[df["FLATLOCATIONID"].notna()].corr(numeric_only=True)['FLATLOCATIONID'].abs().sort_values(ascending=False).iloc[1:]` No se optiene ningún valor alto de correlación con ninguna otra columna por lo que se entiende que los NaNs existentes son completamente aletorios.

En el caso de la columna `FLOORCLEAN` (9829 NaNs, el 5% de las filas) la columan tiene valores entre "-1" y "11".
Representa el numero de planta del piso.
Haciendo una tabla de correlación `df[df["FLOORCLEAN"].notna()].corr(numeric_only=True)['FLOORCLEAN'].abs().sort_values(ascending=False).iloc[1:]` No se optiene ningún valor alto de correlación con ninguna otra columna por lo que se entiende que los NaNs existentes son completamente aletorios.
Se podría pensar que los NaNs podrían venir de los pisos que son especiales como los "estudios" (`ISSTUDIO`), los duplex (`ISDUPLEX`) o las entreplantas (`ISINTOPFLOOR`) pero al contarlos con `df[(df['ISDUPLEX']== True) | (df['ISINTOPFLOOR']== True) | (df['ISSTUDIO']== True) ]['FLOORCLEAN'].isna().sum()` se obtiene un total de 481. Aproximadamente un 5% de los NaNs por lo que refuerza la idea de que la distribución los valores NaN de esta columna es aletoria y por tanto confiables.

Por último la columna `CADASTRALQUALITYID` solo contiene un único valor de NaN. 









