# Actividad 2 · Big Data

**Institución Universitaria de Envigado — Big Data**
**Docente:** Andrés Felipe Hernández Marulanda
**Integrantes:** Natalia Flores Pérez · Santiago Arcila Gutiérrez · Alejandro Restrepo Uribe

Tres bloques de análisis: clasificación de supervivientes del Titanic, exploración de
retrasos aéreos sobre 5,8 millones de vuelos, y análisis de viajes de taxi en Nueva York
con la librería Dask.

---

## Archivos de la entrega

| Archivo | Qué es |
|---|---|
| `Actividad2_BigData.ipynb` | Notebook completo sin ejecutar, para correrlo ustedes |
| `Actividad2_EJECUTADO.ipynb` | El mismo notebook con todas las salidas y gráficos generados |
| `Conclusiones_Actividad2.md` | Resumen de resultados y limitaciones |
| `data/titanic.csv` | Conjunto del Titanic (891 filas) |
| `README.md` | Este archivo |

**Los archivos grandes no vienen incluidos** porque pesan cerca de 800 MB entre los dos.
Hay que ubicarlos en la carpeta `data/` antes de ejecutar:

- `data/flights.csv` — está en el `Data.zip` del curso, dentro de `flights.csv.zip`
- `data/taxi_train.csv` — es el `train.csv` de la competencia
  [nyc-taxi-trip-duration](https://www.kaggle.com/c/nyc-taxi-trip-duration/data) de Kaggle

Si prefieren otras rutas, se cambian en la celda de configuración del notebook
(`RUTA_TITANIC`, `RUTA_VUELOS`, `RUTA_TAXIS`).

## Cómo ejecutarlo

```bash
pip install pandas numpy matplotlib seaborn scikit-learn "dask[dataframe]"
jupyter notebook Actividad2_BigData.ipynb
```

En Google Colab, la primera celda instala Dask automáticamente. Después de instalarlo hay
que reiniciar el entorno de ejecución.

La ejecución completa tarda unos 5 minutos. La celda más lenta es el conteo de filas de
`flights.csv`, que es cuando Dask lee el archivo de verdad por primera vez.

---

## Cumplimiento del enunciado

| Punto del enunciado | Dónde está resuelto |
|---|---|
| **1.** Algoritmo que clasifique sobrevivientes hombres vs mujeres | 1.3 · Algoritmo 1 |
| **1a.** Algoritmo que discrimine según estrato social | 1.3 · Algoritmo 2 |
| **1b.** Algoritmo que discrimine según edad | 1.3 · Algoritmo 3 |
| **2.** Análisis exploratorio básico de vuelos | 2.1 y 2.2 |
| **2a.** Descargar los datos y centrarse en `flights.csv` | 2.1 |
| **2b.** Las 10 principales aerolíneas con más retrasos | 2.3 |
| **2c.** Agrupar por aerolínea y calcular el promedio de retraso | 2.3 |
| **2d.** Cómo varían los retrasos a lo largo de los meses | 2.4 |
| **3.** Análisis exploratorio básico de taxis | 3.2 y 3.3 |
| **3a.** Descargar los datos de la competencia | 3.2 |
| **3b.** Revisar la librería Dask: ¿qué hace? | 3.1 |
| **3c.** Distribución de las tarifas y visualización | 3.5 |
| **3d.** Cómo varían los viajes a lo largo de las horas del día | 3.6 |

Adicionalmente, sin que el enunciado lo pida: modelos completos que combinan las tres
variables del Titanic (para medir cuánto aporta cada una), validación cruzada de todos
los modelos, curvas ROC, y una medición empírica de cuándo Dask **no** conviene.

## Dos decisiones de interpretación del enunciado

Las documentamos aquí y en el notebook porque afectan lo que se entrega.

**1. Las tareas del Titanic.** "Clasificar sobrevivientes hombres vs sobrevivientes
mujeres" leído literalmente es un filtro sobre una columna, no un algoritmo. Lo
resolvimos como tres modelos de clasificación de supervivencia, cada uno centrado en una
variable (género, clase, edad), más modelos completos que las combinan para medir cuánto
aporta cada una.

**2. El conjunto de taxis no tiene tarifas.** El enunciado pide analizar "la distribución
de las tarifas" y enlaza `nyc-taxi-trip-duration`, que trae identificador, proveedor,
fechas, pasajeros, coordenadas y duración del viaje, pero ninguna columna de tarifa. El
conjunto con tarifas es otro (*New York City Taxi Fare Prediction*).

Resolvimos analizando la distribución de la **duración de los viajes**, que es el dato
real disponible, y estimando además la tarifa con la fórmula oficial de la Taxi &
Limousine Commission de Nueva York para 2016. La estimación está marcada como tal en todo
el notebook.

---

## Resultados principales

**Titanic.** El género es el factor determinante: un modelo con esa única variable alcanza
AUC de 0,767 en validación cruzada, frente a 0,867 del modelo completo con todas las
variables. La clase social queda en 0,682 y la edad sola en 0,553. Ser hombre divide por
11 las probabilidades relativas de sobrevivir.

**Retrasos aéreos.** Los dos rankings posibles de "aerolíneas con más retrasos" no
coinciden. Por retraso promedio encabezan Spirit (14,5 min) y Frontier (12,5 min). Por
cantidad de vuelos retrasados encabeza Southwest con 227 mil, simplemente porque opera
1,2 millones de vuelos. Junio es el peor mes con 9,6 minutos de retraso promedio;
septiembre y octubre tienen promedio negativo.

**Taxis y Dask.** El pico de demanda es a las 18:00 con 90 mil viajes, y el valle a las
5:00 con 15 mil. La duración promedio es máxima entre las 15:00 y las 16:00, no en el pico
de demanda. La mediana de duración es de 11,1 minutos.

Medimos también el costo de usar Dask donde no corresponde: con las 891 filas del Titanic
resultó 50 veces más lento que pandas.

---

## Limitaciones declaradas

1. Las tarifas de taxi son una estimación nuestra, no un dato del conjunto, y al no
   incluir el componente por distancia son una cota inferior.
2. El 20% de las edades del Titanic fue imputado usando el título del pasajero
   (`Mr.`, `Mrs.`, `Miss.`, `Master.`). Las conclusiones sobre edad heredan esa
   incertidumbre.
3. El archivo del Titanic no es el estándar de Kaggle: no trae `SibSp` ni `Parch`, que en
   la literatura resultan predictivas.
4. Los datos de vuelos son solo de 2015 y de un solo país. Las conclusiones estacionales
   no son extrapolables sin más.
5. El retraso promedio subestima el mal desempeño invernal, porque en invierno los vuelos
   se cancelan en lugar de retrasarse y un vuelo cancelado no aporta minutos de retraso.
