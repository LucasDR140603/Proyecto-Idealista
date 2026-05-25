# Proyecto Idealista
# Contexto
Los csv con los que se han trabajado contienen datos de pisos de:
1. Madrid y Barcelona
2. Madrid y Valencia

Se hace una análisis del tipo de columnas. Hay 46 col

Se detecta que todos los datos son pisos en venta y son viviendas, yaq ue el contenido de las columanas DTYPOLOGYID y ADTYPOLOGY es "HOME" y el contenido de las columnas ADOPERATIONID y ADOPERATION es "SALE"
Además de esto se detectaque son iguales por lo que se pueden eliminar las columnas repetidas.


Se puede ver que existen una serie de columnas que, aunque son de tipo Int, en realidad solo tienen valores de 0 o 1. Estos valores se pueden pasar a tipo booleano.

Existen 3 columnas de tipo booleano que describoien el tipo de construcción.
Se decide pasar a una unica columna de tenga como valores posibles 1, 2 o 3 llamada  BUILTTYPEIDID

Primer anális: precio. Se cree que puede ser una distribución log nomral pero está pendiente de confirmar.




*Tarea pendiente: Hacer un Análisi de las columnas que tienen Nans y ver porque son esos Nan, si se pueden usar esas columnas o no.


Analisis del periodo indica que los datos reflejan datos agrupados por 

