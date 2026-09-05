# Conclusiones · Actividad 2 Big Data

**Integrantes:** Natalia Flores Pérez · Santiago Arcila Gutiérrez · Alejandro Restrepo Uribe

---

## Parte 1 · Titanic

### Qué se pidió y cómo se resolvió

El enunciado pide tres algoritmos: uno que clasifique sobrevivientes hombres frente a
mujeres, otro según el estrato social y otro según la edad. Leídas literalmente, esas tres
tareas son filtros sobre columnas y no requieren un algoritmo. Las entendimos como tres
modelos de clasificación de supervivencia, cada uno centrado en una variable, y añadimos
modelos completos que las combinan para medir cuánto aporta cada una en competencia con
las demás.

### Resultados

| Algoritmo | Variables | Exactitud | Precisión | Recall | F1 | AUC |
|---|---|---|---|---|---|---|
| Género | `gender` | 0,785 | 0,750 | 0,663 | 0,704 | 0,762 |
| Clase social | `class` | 0,646 | 0,561 | 0,372 | 0,448 | 0,635 |
| Edad (árbol) | `edad` | 0,632 | 0,667 | 0,093 | 0,163 | 0,581 |
| Edad (logística) | `edad` | 0,614 | 0,000 | 0,000 | 0,000 | 0,537 |
| Completo (logística) | todas | 0,794 | 0,763 | 0,674 | 0,716 | 0,823 |
| Completo (árbol) | todas | 0,794 | 0,823 | 0,593 | 0,689 | 0,847 |
| Completo (bosque) | todas | 0,785 | 0,732 | 0,698 | 0,714 | 0,836 |

Todos los valores corresponden al conjunto de prueba (223 pasajeros). Los de
validación cruzada se reportan aparte en el párrafo siguiente.

### Lectura

**El género es el factor decisivo.** Un modelo con esa sola variable binaria alcanza casi
el desempeño del modelo completo. El coeficiente de la regresión logística da una razón de
momios de 0,088: ser hombre divide por 11 las probabilidades relativas de sobrevivir. Las
tasas crudas lo confirman, 74,2% de las mujeres frente a 18,9% de los hombres.

**La clase social influye, pero menos.** La tasa cae de 63% en primera a 24% en tercera.
Sin embargo, una mujer de tercera clase tuvo mejor pronóstico que un hombre de primera:
el género pesó más que el dinero.

**La edad solo discrimina en el tramo infantil.** Por eso el árbol supera a la regresión
logística en esa tarea (AUC 0,581 frente a 0,537): la relación no es monótona, los niños
se salvan mucho más y entre adultos la tasa se aplana. Un modelo lineal no puede capturar
eso.

**Una advertencia sobre la clase social.** El modelo entrenado solo con la clase tiene un
recall bajo en la categoría "sobrevivió", porque ninguna de las tres clases supera el 50%
de supervivencia y el modelo termina prediciendo "no sobrevivió" para casi todos. Es el
ejemplo clásico de por qué la exactitud sola engaña, y la razón por la que reportamos
también F1 y AUC.

---

## Parte 2 · Retrasos de vuelos

### Las 10 aerolíneas con más retrasos

La pregunta admite dos lecturas que dan rankings distintos, y las dos son correctas según
a quién se le responda.

**Por retraso promedio** (minutos de retraso en la llegada por vuelo):

| Aerolínea | Minutos | Vuelos |
|---|---|---|
| Spirit Air Lines | 14,47 | 115.193 |
| Frontier Airlines | 12,50 | 90.090 |
| JetBlue Airways | 6,68 | 262.042 |
| Atlantic Southeast | 6,59 | 554.752 |
| American Eagle | 6,46 | 278.791 |
| Skywest Airlines | 5,85 | 576.814 |
| United Air Lines | 5,43 | 507.762 |
| Virgin America | 4,74 | 61.248 |
| Southwest Airlines | 4,37 | 1.242.403 |
| US Airways | 3,71 | 194.223 |

**Por cantidad de vuelos retrasados** (más de 15 minutos, criterio del Departamento de
Transporte de Estados Unidos): encabeza Southwest con 227.317, seguida de American con
125.238 y Delta con 113.112.

Los dos rankings no coinciden porque miden cosas distintas. Southwest lidera en cantidad
simplemente porque opera 1,2 millones de vuelos, más del doble que cualquier otra; su
porcentaje de vuelos retrasados es del 18,3%, por debajo del de Spirit, que llega al
28,8%. Para un pasajero que elige aerolínea sirve el ranking por promedio; para un
regulador que quiere reducir el total de horas perdidas sirve el de cantidad.

### Variación a lo largo del año

| Mes | Retraso promedio (min) | Cancelaciones (%) |
|---|---|---|
| Enero | 5,81 | 2,55 |
| Febrero | 8,32 | 4,78 |
| Marzo | 4,92 | 2,18 |
| Abril | 3,16 | 0,93 |
| Mayo | 4,49 | 1,15 |
| Junio | 9,60 | 1,81 |
| Julio | 6,43 | 0,92 |
| Agosto | 4,61 | 0,99 |
| Septiembre | −0,77 | 0,45 |
| Octubre | −0,78 | 0,50 |
| Noviembre | 1,10 | 0,98 |
| Diciembre | 6,09 | 1,68 |

Junio es el peor mes, seguido de febrero, julio y diciembre. Junio y julio coinciden con
las vacaciones de verano en Estados Unidos y diciembre con las fiestas. Septiembre y
octubre tienen retraso promedio negativo, o sea que en promedio los vuelos llegaron antes
de lo programado.

**Febrero merece una nota aparte.** Tiene la tasa de cancelación más alta del año, 4,78%,
casi el doble que enero. Las tormentas de invierno cancelan vuelos en lugar de
retrasarlos, y un vuelo cancelado desaparece de la estadística de retrasos. Eso significa
que **el retraso promedio subestima el mal desempeño de los meses de invierno**, y
conviene decirlo al presentar la tabla.

### A lo largo del día

El retraso promedio es negativo hasta media mañana y crece durante todo el día. La causa
es acumulativa: un avión que sale tarde temprano arrastra ese retraso en todos sus vuelos
siguientes. La recomendación práctica que sale de los datos es volar temprano.

---

## Parte 3 · Taxis de Nueva York y Dask

### Qué hace Dask

Dask es una librería de Python para computación paralela que permite trabajar con
conjuntos de datos más grandes que la memoria RAM disponible.

Un DataFrame de Dask no es una tabla: es un conjunto de DataFrames de pandas llamados
**particiones**, más un plan de operaciones. Cuando se pide un cálculo, Dask lo aplica
partición por partición y combina los resultados, de modo que en cualquier momento solo
hay unas pocas particiones en memoria.

Su rasgo característico es la **evaluación perezosa**: nada se ejecuta hasta que se llama
a `.compute()`. En el notebook se ve con claridad, porque `dd.read_csv` sobre un archivo
de 592 MB devuelve en 0,3 segundos (solo construye el plan) mientras que el `len()`
posterior tarda 39 segundos, que es cuando lee de verdad.

**Cuándo no usarlo.** Dask es más lento que pandas cuando los datos caben en memoria,
porque dividir, coordinar y volver a juntar tiene un costo fijo. Lo medimos: con las 891
filas del Titanic, Dask resultó 46 veces más lento. Incluso con 1,4 millones de filas de
taxis, pandas fue más rápido (0,9 s frente a 2,9 s) porque el archivo aún cabía en
memoria. La ventaja de Dask no es la velocidad pura, es poder procesar archivos que de
otro modo tumbarían el programa.

### Distribución de las tarifas

El conjunto enlazado en el enunciado no contiene tarifas. Analizamos la duración del
viaje, que es el dato real, y estimamos la tarifa con la fórmula de la Taxi & Limousine
Commission de Nueva York para 2016 (2,50 USD de bajada de bandera más 0,50 USD por
minuto).

| Percentil | Duración (min) | Tarifa estimada (USD) |
|---|---|---|
| 10% | 4,2 | 4,60 |
| 25% | 6,7 | 5,85 |
| 50% (mediana) | 11,1 | 8,05 |
| 75% | 18,0 | 11,48 |
| 90% | 27,2 | 16,10 |
| 95% | 34,9 | 19,95 |
| 99% | 55,5 | 30,24 |

La distribución es asimétrica: la media de 14,0 minutos supera la mediana de 11,1, señal
de una cola de viajes largos. Antes de calcular esto eliminamos 10.707 viajes (0,73%) con
duraciones imposibles, desde 1 segundo hasta más de 24 horas, que son errores del
taxímetro.

### Variación a lo largo del día

El pico de demanda es a las 18:00 con 90.028 viajes, coincidiendo con la salida de las
oficinas. El valle está a las 5:00 con 14.770, la única hora en que la ciudad casi se
detiene.

**La duración promedio no sigue el mismo patrón.** Alcanza su máximo entre las 15:00 y las
16:00, con 16,2 minutos, no a las 18:00 cuando hay más viajes. La hora de mayor demanda no
es la hora en que peor se mueve la ciudad. Como la tarifa por tiempo crece con la
duración, el mismo trayecto sale más caro a media tarde que en el pico de la noche.

El cruce por hora y día de la semana muestra dos patrones distintos: entre semana hay dos
franjas marcadas de mañana y tarde, propias de los desplazamientos al trabajo, mientras
que el fin de semana la actividad se corre hacia la noche y se extiende hasta la
madrugada.

---

## Limitaciones que declaramos

1. **Las tarifas de taxi son una estimación nuestra**, calculada a partir de la duración y
   la fórmula de la TLC de 2016. No son un dato del conjunto, y al no incluir el
   componente por distancia recorrida son una cota inferior del valor real.
2. **El 20% de las edades del Titanic fue imputado.** Lo hicimos con criterio, usando el
   tratamiento del pasajero para que un `Master.` no recibiera la mediana de un adulto,
   pero sigue siendo un valor estimado y no observado. Las conclusiones sobre edad heredan
   esa incertidumbre.
3. **El archivo del Titanic no es el estándar de Kaggle.** Carece de `SibSp` y `Parch`,
   las variables de familiares a bordo, que en la literatura resultan predictivas. Nuestro
   modelo completo probablemente rendiría algo mejor con ellas.
4. **Los datos de vuelos corresponden solo a 2015** y a un solo país. Las conclusiones
   estacionales dependen del calendario de vacaciones estadounidense y no son
   extrapolables sin verificación.
5. **El retraso promedio como métrica tiene un sesgo estructural**: subestima el mal
   desempeño de los meses de invierno, porque en esa temporada los vuelos se cancelan en
   lugar de retrasarse y un vuelo cancelado no aporta minutos de retraso a la estadística.
