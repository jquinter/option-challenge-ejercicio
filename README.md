# option-challenge-ejercicio

Primero analizo el dataset. Al final, realizo los ejercicios propuestos.

## Análisis Titanic

Primero, descargo datos desde el repositorio de github a mi máquina.

![zip en github](https://github.com/jquinter/option-challenge-ejercicio/raw/master/img/Screen_Shot_2019-09-03_at_23.14.13.png "zip en github")

Luego, creo un proyecto en GCP para alojar todo el análisis

![new project gcp](https://github.com/jquinter/option-challenge-ejercicio/raw/master/img/Screen_Shot_2019-09-03_at_23.15.05.png "new project gcp")

![new project gcp ready](https://github.com/jquinter/option-challenge-ejercicio/raw/master/img/Screen_Shot_2019-09-03_at_23.15.48.png "new project gcp ready")

Con el proyecto ya creado, procedo a hacer ingesta de información:  subo datos a un bucket en GCS

![upload zip to gcs](https://github.com/jquinter/option-challenge-ejercicio/raw/master/img/Screen_Shot_2019-09-03_at_23.16.42.png "upload zip to gcs")

Para hacer exploración inicial y primeras tareas (limpieza, etc) usaré dataprep:

![activate dataprep](https://github.com/jquinter/option-challenge-ejercicio/raw/master/img/Screen_Shot_2019-09-03_at_23.18.24.png "activate dataprep")

Ups, pequeño detalle: necesito que datos ya estén descomprimidos. Hago lo necesario en cloud shell, rápidamente

![unzip and upload](https://github.com/jquinter/option-challenge-ejercicio/raw/master/img/Screen_Shot_2019-09-03_at_23.23.21.png "unzip and upload")

Ahora cargo los datos: el preview ya me confirma que tengo información!

![preview dataprep](https://github.com/jquinter/option-challenge-ejercicio/raw/master/img/Screen_Shot_2019-09-03_at_23.23.44.png "preview dataprep")

Ok, puedo hacer exploración. Acá vemos varios tipos numéricos, como el id de pasajero (_PassengerId_), un indicador binario (_Survived_), la clase del pasajero (_Pclass_). Tenemos como strings el nombre (viene entre comillas), y el género. Algo llamativo del nombre es que las mujeres casadas aparecían registradas bajo el nombre del respectivo esposo (sus nombres aparecían entre paréntesis).

![preview dataprep](https://github.com/jquinter/option-challenge-ejercicio/raw/master/img/Screen_Shot_2019-09-03_at_23.24.40.png "preview dataprep")

Y también encontramos edad (no todos tienen edad), y datos respecto al valor del ticket (creo que hay varios outliers) y otros (_Fare_!). No tengo idea qué pueden significar _SibSp_, _Parch_ o Embarked (este último tiene 3 valores posibles).

![preview dataprep](https://github.com/jquinter/option-challenge-ejercicio/raw/master/img/Screen_Shot_2019-09-03_at_23.25.32.png "preview dataprep")

Lo primero que hago es limpiar los nombres: no hice mayor tratamiento para el caso de las mujeres casadas.

![wrangling names](https://github.com/jquinter/option-challenge-ejercicio/raw/master/img/Screen_Shot_2019-09-03_at_23.27.18.png "wrangling names")

Habían algunas edades que no correspondían a números enteros: decidí redondearlas _hacia abajo_, es decir, consideré "edad cumplida" como el valor final.

![wrangling age](https://github.com/jquinter/option-challenge-ejercicio/raw/master/img/Screen_Shot_2019-09-03_at_23.29.44.png "wrangling age")

![wrangling age](https://github.com/jquinter/option-challenge-ejercicio/raw/master/img/Screen_Shot_2019-09-03_at_23.30.21.png "wrangling age")

En el caso de los valores de tickets, habían registros con mezcla de algún prefijo (no pude descifrarlo rápido) y el costo del boleto. Ah, no filtré los outliers que sospecho distorsionan la muestra (tickets que costaban millones de dólares?)

![wrangling tickets](https://github.com/jquinter/option-challenge-ejercicio/raw/master/img/Screen_Shot_2019-09-03_at_23.31.32.png "wrangling tickets")

![wrangling tickets](https://github.com/jquinter/option-challenge-ejercicio/raw/master/img/Screen_Shot_2019-09-03_at_23.32.01.png "wrangling tickets")

Independiente del ajuste, 4 registros/personas no tenían valor de ticket

![null tickets](https://github.com/jquinter/option-challenge-ejercicio/raw/master/img/Screen_Shot_2019-09-03_at_23.33.28.png "null tickets")

Ok, creo que eran los ajustes mínimos necesarios como para cargar datos. Elegí bigquery como repositorio (costos, facilidad de integración... y además, vamos a hacer analítica! -sobre muy pocos datos, lo sé, pero funciona). Primero, necesito un dataset donde dejar la información.

PS: sí, aún uso la interfaz clásica. Aprendí BigQuery con ella.

![create dataset](https://github.com/jquinter/option-challenge-ejercicio/raw/master/img/Screen_Shot_2019-09-03_at_23.34.57.png "create dataset")

Ok, volviendo a dataprep, configuro un _job_ que escribirá los datos con todas las transformaciones anteriores, en una tabla de BigQuery. No pensé mucho los nombres la verdad.

![create job in dataprep](https://github.com/jquinter/option-challenge-ejercicio/raw/master/img/Screen_Shot_2019-09-03_at_23.35.21.png "create job in dataprep")

![monitoring job in dataprep](https://github.com/jquinter/option-challenge-ejercicio/raw/master/img/Screen_Shot_2019-09-03_at_23.35.48.png "monitoring job in dataprep")

Los datos quedaron cargados: validamos esquema

![table schema in bigquery](https://github.com/jquinter/option-challenge-ejercicio/raw/master/img/Screen_Shot_2019-09-03_at_23.40.43.png "table schema in bigquery")

detalles de la tabla

![table details in bigquery](https://github.com/jquinter/option-challenge-ejercicio/raw/master/img/Screen_Shot_2019-09-03_at_23.40.52.png "table details in bigquery")

confirmación: archivo original tenía n+1 filas (+1 por el encabezado!)

![row count validation](https://github.com/jquinter/option-challenge-ejercicio/raw/master/img/Screen_Shot_2019-09-03_at_23.41.24.png "row count validation")

vista previa de datos

![table preview in bigquery](https://github.com/jquinter/option-challenge-ejercicio/raw/master/img/Screen_Shot_2019-09-03_at_23.41.39.png "table preview in bigquery")

Ok, está listo para poder visualizar: conecto BigQuery a DataStudio

![connect bigquery to datastudio](https://github.com/jquinter/option-challenge-ejercicio/raw/master/img/Screen_Shot_2019-09-03_at_23.42.27.png "connect bigquery to datastudio")

Elaboré este pequeño informe: podemos ver distribución de géneros en el registro de 891 pasajeros, la distribución de sobrevivientes según géneros (primera tabla, arriba), así como la distribución de sobrevivientes según géneros y la clase del pasajero (segunda tabla), y finalmente un histograma del costo promedio de tickets, según la clase del pasajero (esto está distorsionado por los outliers que no removí!).

![datastudio report](https://github.com/jquinter/option-challenge-ejercicio/raw/master/img/Screen_Shot_2019-09-04_at_00.20.20.png "datastudio report")

Tarea adicional: consulta para agregar prioridad de salvataje. Reviso registros para validar que cálculo/asignación esté correcto

![additional task](https://github.com/jquinter/option-challenge-ejercicio/raw/master/img/Screen_Shot_2019-09-04_at_00.22.14.png "additional task")

Por si no se ve, esta es:

```
SELECT
  CASE
    WHEN age < 18 THEN 'p1'
    WHEN Sex = 'female' THEN 'p2'
    ELSE 'p3'
  END AS ps,
  *
FROM
  [option-challenge-dataeng:titanic.titanic]

```

## Ejercicios

1. Dado un número _N_, desarrolle el algoritmo que encuentre el resultado de `1 + 2 + 3 + 4 + ... + N`

Un enfoque _naive_ es hacer la suma directa

```
sum = 0
for i = 1 to N do:
	sum += i
done
return sum
```

Sin embargo, es mucho mejor aplicar la fórmula para suma de enteros correlativos:

```
return N*(N+1)/2
```


