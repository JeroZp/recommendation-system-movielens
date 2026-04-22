# Data Card — MovieLens 100K

## 1. Resumen

**Nombre del dataset:** MovieLens 100K
**Fuente:** GroupLens Research, University of Minnesota
**URL:** [https://grouplens.org/datasets/movielens/100k/](https://grouplens.org/datasets/movielens/100k/)

El dataset contiene interacciones usuario–película en forma de ratings explícitos, junto con información demográfica de usuarios y metadatos de películas. Es un dataset estándar para el estudio de sistemas de recomendación basados en filtrado colaborativo y modelos híbridos.


## 2. Descripción general

* **Número de ratings:** 100,000
* **Número de usuarios:** 943
* **Número de películas:** 1,682
* **Escala de ratings:** 1 a 5
* **Periodo de recolección:** septiembre 1997 – abril 1998

Restricción importante:

* Cada usuario tiene al menos **20 ratings**, lo cual reduce sparsity extrema pero introduce sesgo de selección.


## 3. Tipo de tarea

Este dataset permite múltiples formulaciones, pero en este proyecto se utiliza para:

* **Tipo de tarea:** Aprendizaje supervisado
* **Problema:** Regresión
* **Objetivo:** Predecir el rating que un usuario daría a una película

Alternativamente, también puede utilizarse para:

* Ranking (top-N recommendations)
* Clasificación (like/dislike)
* Filtrado colaborativo (user-based / item-based)


## 4. Estructura de los datos

### 4.1 Archivo principal: `u.data`

Formato:

```
user_id | item_id | rating | timestamp
```

* `user_id`: identificador del usuario
* `item_id`: identificador de la película
* `rating`: valor entre 1 y 5
* `timestamp`: tiempo en formato Unix


### 4.2 Información de películas: `u.item`

Contiene:

* `movie_id`
* `movie_title`
* `release_date`
* `IMDb URL`
* 19 variables binarias de género (multi-label)

Ejemplo de géneros:

* Action, Comedy, Drama, Romance, Sci-Fi, etc.

**Observaciones del EDA:**

* `video_release_date`: completamente vacía → eliminada
* `IMDb URL`: no aporta valor predictivo → eliminada
* `release_date`: contiene valores faltantes → imputados


### 4.3 Información de usuarios: `u.user`

Formato:

```
user_id | age | gender | occupation | zip_code
```

Variables:

* Edad (numérica)
* Género (categórica: M/F)
* Ocupación (categórica)
* Código postal


### 4.4 Archivos auxiliares

* `u.genre`: lista de géneros
* `u.occupation`: lista de ocupaciones


### 4.5 Splits predefinidos

El dataset incluye particiones listas para validación:

* `u1.base` / `u1.test` … `u5.base` / `u5.test`
  → **5-fold cross-validation**

* `ua.base`, `ua.test`, `ub.base`, `ub.test`
  → splits con exactamente **10 ratings por usuario en test**

### 4.6 

## 5. Características relevantes del dataset

### 5.1 Sparsity

* Aproximadamente **93–94% de la matriz usuario–película está vacía**

**Implicación:**

* Problema típico de sistemas de recomendación
* Dificulta generalización


### 5.2 Distribución de ratings

* Media ≈ 3.5
* Mediana = 4
* Alta concentración en ratings 3 y 4

**Implicación:**

* Sesgo positivo
* Baselines simples (media) son relativamente fuertes


### 5.3 Distribución de interacciones

* Cola larga:

  * Pocos usuarios muy activos
  * Muchas películas con pocos ratings

**Implicación:**

* Problemas de cold start
* Datos desbalanceados


### 5.4 Demografía

* Mayoría de usuarios son hombres (~75%)
* Edad promedio ≈ 34 años
* Alta presencia de estudiantes

**Implicación:**

* Dataset no representativo de la población general
* Posibles sesgos en recomendaciones


## 6. Calidad de los datos

### 6.1 Valores faltantes

* Ratings: sin faltantes
* Usuarios: completos
* Películas:

  * `release_date`: algunos faltantes → imputados
  * `video_release_date`: completamente faltante → eliminada


### 6.2 Consistencia

* No se detectaron duplicados en ratings
* IDs consistentes entre tablas
* Tipos de datos correctos tras limpieza


### 6.3 Fiabilidad

Advertencia oficial del dataset:

* No se garantiza la exactitud total de los datos
* No se asegura su idoneidad para todos los usos


## 7. Limitaciones

### 7.1 Sesgo de selección

* Solo usuarios con ≥20 ratings
  → excluye usuarios poco activos


### 7.2 Sesgo demográfico

* Predominancia de hombres jóvenes
  → posible sesgo en patrones aprendidos


### 7.3 Sesgo temporal

* Datos recolectados en 1997–1998
  → preferencias pueden no generalizar a contextos actuales


### 7.4 Sparsity estructural

* Gran cantidad de valores faltantes implícitos
  → dificulta modelado


### 7.5 Subjetividad del target

* El rating es:

  * subjetivo
  * dependiente del usuario
  * inconsistente entre individuos


## 8. Riesgos potenciales

* **Bias en recomendaciones**:

  * favorecimiento de películas populares
  * subrepresentación de nichos

* **Cold start**:

  * nuevos usuarios/películas sin historial

* **Overfitting a usuarios activos**

* **Generalización limitada**:

  * dominio específico (cine, años 90)


## 9. Uso permitido

El dataset puede usarse para investigación bajo condiciones:

* No uso comercial sin autorización
* No redistribución sin permiso
* Se debe citar el dataset en publicaciones


## 10. Citación recomendada

> Harper, F. M., & Konstan, J. A. (2015).
> *The MovieLens Datasets: History and Context.*
> ACM Transactions on Interactive Intelligent Systems (TiiS), 5(4).


## 11. Conclusión

MovieLens 100K es un dataset clásico y bien estructurado para sistemas de recomendación, con:

* buena calidad de datos
* estructura realista
* desafíos relevantes (sparsity, sesgos, cold start)

Sin embargo, sus limitaciones (sesgos demográficos, antigüedad y sparsity) deben considerarse explícitamente en el modelado y evaluación.

