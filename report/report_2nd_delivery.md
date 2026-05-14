# Sistema de Recomendación con MovieLens 100K

## Entrega 2 — Comparación de Familias y Validación

**Institución:** EAFIT  
**Curso:** Aprendizaje de Máquina Aplicado  
**Profesor:** Marco Teran  
**Fecha límite:** 14/05/2026  
**Dataset:** MovieLens 100K — GroupLens Research  
**Integrantes:**
- Jerónimo Pérez Baquero (jperezb5@eafit.edu.co)
- David Grisales Posada (dgrisalesp@eafit.edu.co)
- Esteban Vergara Giraldo (evergarag@eafit.edu.co)

---

## Tabla de Contenidos

1. [Pregunta central](#1-pregunta-central)
2. [Modelos comparados y justificación](#2-modelos-comparados-y-justificación)
3. [Preprocesamiento y prevención de data leakage](#3-preprocesamiento-y-prevención-de-data-leakage)
4. [Validación](#4-validación)
5. [Resultados comparativos](#5-resultados-comparativos)
6. [Análisis e interpretación](#6-análisis-e-interpretación)
7. [Familia más prometedora](#7-familia-más-prometedora)
8. [Limitaciones abiertas](#8-limitaciones-abiertas)

---

## 1. Pregunta central

¿Es posible predecir el rating (1–5) que un usuario le daría a una película que no ha visto? ¿Cuál familia de modelos lo hace mejor con el dataset MovieLens 100K?

---

## 2. Modelos comparados y justificación

Se seleccionaron tres familias de modelos que representan enfoques fundamentalmente distintos para el problema de recomendación:

**SVD (Singular Value Decomposition) — Factores Latentes:** Descompone la matriz usuario-película en factores latentes que capturan patrones abstractos de gusto (por ejemplo, preferencia por cierto tipo de cine independientemente de la etiqueta de género). Es el estándar del estado del arte en sistemas de recomendación colaborativos.

**KNN (K-Nearest Neighbors) — Filtrado Colaborativo:** Predice el rating de un ítem buscando los *k* ítems más similares (en esta implementación, item-based con similitud coseno). Es altamente interpretable: recomienda algo porque ítems parecidos recibieron calificaciones similares.

**Lasso y Ridge (Regresión Lineal Regularizada) — Modelos Basados en Contenido:** Predicen el rating como una función lineal de los metadatos del usuario (edad, género, ocupación) y de la película (géneros, año). Permiten identificar qué variables demográficas y de contenido tienen mayor peso predictivo, y sirven como punto de comparación para cuantificar el valor del historial de interacciones frente a los datos de perfil.

La combinación de estas tres familias permite contrastar el filtrado colaborativo puro (SVD, KNN) contra el enfoque basado en contenido (Lasso/Ridge), cubriendo los pilares clásicos del área.

---

## 3. Preprocesamiento y prevención de data leakage

### Construcción de la tabla maestra

Para los modelos de regresión se construyó una tabla maestra mediante un merge de los tres archivos del dataset (ratings, users, movies). Cada fila del conjunto de ratings quedó enriquecida con los metadatos del usuario y de la película. SVD y KNN utilizan únicamente la terna [user_id, item_id, rating], según los requerimientos de la librería Surprise.

### Feature engineering y limpieza

- Se descartaron columnas sin valor predictivo: video_release_date (100% nulos), imdb_url, title, zip_code, timestamp y los IDs como variables numéricas.
- De release_date se extrajo únicamente el año (release_year), tratado como variable numérica continua. El único valor nulo se imputó con la mediana.
- Los 19 géneros de las películas se conservaron tal como vienen en el dataset: variables binarias (0/1), sin transformación adicional.

### Transformaciones

Se definió un ColumnTransformer con tratamiento diferenciado por tipo de variable:

- age y release_year: estandarización con StandardScaler.
- gender y occupation: codificación con OneHotEncoder (sin escalar, ya que son binarias tras la codificación).
- Géneros de películas: passthrough — ya están en escala binaria y no requieren transformación.

### Prevención de data leakage

El StandardScaler se ajustó (fit) exclusivamente sobre el conjunto de entrenamiento. La misma transformación (parámetros ya fijados) se aplicó con transform a los conjuntos de validación y test. Esto garantiza que ninguna información estadística de los conjuntos de evaluación influyera en el preprocesamiento. Adicionalmente, se fijó una semilla aleatoria global (FIXED_RANDOM_SEED = 42) en todas las operaciones de división y entrenamiento para asegurar reproducibilidad.

---

## 4. Validación

Se adoptó una partición **holdout en tres conjuntos**: 60% entrenamiento, 20% validación y 20% test. La partición se realizó de forma aleatoria estratificada mediante train_test_split con semilla fija. La validación cruzada de 3 pliegues (cv=3) se utilizó exclusivamente dentro del conjunto de entrenamiento para la búsqueda de hiperparámetros (GridSearchCV), evitando que los datos de validación y test participaran en la selección de parámetros.

Esta estrategia garantiza que las métricas reportadas en el test reflejen el desempeño real sobre datos no vistos en ningún momento del proceso de modelado.

---

## 5. Resultados comparativos

El baseline de referencia consiste en predecir siempre el promedio global del conjunto de entrenamiento ($\hat{y} = 3.53$), con un RMSE de **1.1253** en test.

| Modelo   | RMSE (Test) | Mejora vs. Baseline |
|----------|------------|---------------------|
| SVD      | **0.9214** | 18.1%              |
| KNN      | 0.9464     | 15.9%              |
| Ridge    | 1.0978     | 2.4%               |
| Lasso    | 1.0986     | 2.4%               |
| Baseline | 1.1253     | —                   |

Los hiperparámetros óptimos se obtuvieron mediante búsqueda en grilla:
- **Lasso**: alpha = 0.001
- **Ridge**: alpha = 10.0
- **KNN**: item-based, similitud coseno, k = 80, min_k = 1
- **SVD**: n_factors = 150, n_epochs = 30, lr_all = 0.01, reg_all = 0.1

---

## 6. Análisis e interpretación

**SVD y KNN superan ampliamente a los modelos lineales.** Ambos operan sobre la matriz de interacciones, lo que les permite capturar patrones de gusto individual que no están escritos en el perfil demográfico. SVD, en particular, puede detectar afinidades latentes (por ejemplo, preferencia por cierto estilo narrativo) sin que esa categoría exista explícitamente en los datos.

**Lasso y Ridge apenas superan el baseline.** El motivo es la llamada *content-only limitation*: dos usuarios con idénticos metadatos (misma edad, género y ocupación) pueden tener gustos radicalmente distintos. Los modelos lineales no tienen acceso al historial de interacciones, por lo que asignan predicciones similares a perfiles similares, ignorando la variabilidad individual real.

**Ridge supera ligeramente a Lasso** (1.0978 vs. 1.0986). En datasets de tamaño moderado como MovieLens 100K, la penalización $$L_2$$ de Ridge resulta más adecuada que la eliminación agresiva de variables de Lasso: mantener señales pequeñas de múltiples variables demográficas aporta más que descartarlas.

**Análisis de coeficientes Lasso:** Las variables que sobrevivieron a la penalización (coeficiente ≠ 0) revelan que los géneros de la película tienen mayor peso predictivo que las variables demográficas del usuario, lo que confirma que el contenido de la película es más determinante que el perfil del espectador para los modelos lineales.

---

## 7. Familia más prometedora

El **filtrado colaborativo basado en factores latentes (SVD)** se posiciona como la familia más prometedora, con un RMSE de 0.9214 — una mejora del 18% sobre el baseline. Su capacidad de capturar preferencias individuales a partir del historial de interacciones, sin depender de metadatos externos, lo convierte en la opción más robusta para este problema.

KNN representa una alternativa interpretable con desempeño competitivo (RMSE 0.9464), aunque con mayor costo computacional en producción. Los modelos lineales son útiles para analizar qué variables influyen en los ratings, pero no son candidatos principales para la recomendación final.

---

## 8. Limitaciones abiertas

**Cold start:** SVD y KNN dependen del historial de interacciones. Para usuarios nuevos o películas con pocas calificaciones, su desempeño se degrada. Los modelos lineales (Lasso/Ridge) son más robustos en este escenario, ya que pueden hacer predicciones a partir del perfil demográfico incluso sin historial.

**Sesgo demográfico del dataset:** Aproximadamente el 75% de los usuarios son hombres, con una mediana de edad de 31 años y una ocupación predominantemente estudiantil. El modelo aprende principalmente de este perfil, lo que limita su generalización a otros grupos.

**Sesgo temporal:** La mayoría de los ratings corresponden a películas de los años 90. El modelo puede subestimar películas de otras épocas con menor densidad de calificaciones.

**Sparsity:** La matriz usuario-película tiene una sparsity del 93.7%. Zonas de la matriz con muy pocas interacciones (usuarios o películas con pocos ratings) son inherentemente más difíciles de predecir, independientemente del modelo utilizado.

**Hiperparámetros de SVD:** La búsqueda en grilla realizada cubre un subespacio limitado de configuraciones. Una búsqueda más exhaustiva (Optuna, Bayesian optimization) podría mejorar adicionalmente el RMSE.