# Data Card — MovieLens 100K

## 1. Resumen

**Nombre del dataset:** MovieLens 100K
**Fuente:** GroupLens Research, University of Minnesota
**URL oficial del dataset:** [https://grouplens.org/datasets/movielens/100k/](https://grouplens.org/datasets/movielens/100k/)
**URL del recomendador MovieLens:** [http://www.movielens.org/](http://www.movielens.org/)
**Página del proyecto GroupLens:** [http://www.grouplens.org/](http://www.grouplens.org/)

El dataset contiene interacciones usuario–película en forma de ratings explícitos, junto con información demográfica de usuarios y metadatos de películas. Es un dataset estándar para el estudio de sistemas de recomendación basados en filtrado colaborativo y modelos híbridos.


## 2. Descripción general

* **Número de ratings:** 100,000
* **Número de usuarios:** 943
* **Número de películas:** 1,682
* **Escala de ratings:** 1 a 5
* **Periodo de recolección:** 19 de septiembre de 1997 – 22 de abril de 1998

Restricción importante:

* Cada usuario tiene al menos **20 ratings**, lo cual reduce sparsity extrema pero introduce sesgo de selección.
* **Usuarios eliminados en limpieza inicial:** Todos los usuarios con menos de 20 ratings o sin información demográfica completa fueron removidos.
* El dataset ha sido limpiado y validado para consistencia de IDs entre tablas.


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
* `u.info`: contiene el número de usuarios, películas y ratings
* `allbut.pl`: script que genera conjuntos de entrenamiento/prueba (todos excepto n ratings por usuario)
* `mku.sh`: script para generar todos los splits (`u1` a `u5`, `ua`, `ub`) a partir de `u.data`


### 4.5 Splits predefinidos

El dataset incluye particiones listas para validación:

* **5-fold cross-validation:**
  * `u1.base` / `u1.test` … `u5.base` / `u5.test`
  * Split 80% (entrenamiento) / 20% (prueba)
  * Conjuntos de prueba disjuntos para validación cruzada

* **Splits con ratings fijos por usuario:**
  * `ua.base` / `ua.test`, `ub.base` / `ub.test`
  * Exactamente **10 ratings por usuario en el conjunto de prueba**
  * `ua.test` y `ub.test` son disjuntos

### 4.6 Descripción detallada de archivos

#### `ml-data.tar.gz`

Archivo comprimido. Para reconstruir los archivos `u.*` ejecutar:

```bash
gunzip ml-data.tar.gz
tar xvf ml-data.tar
mku.sh
```

#### `u.data`

El conjunto de datos completo: 100,000 ratings de 943 usuarios en 1,682 películas.

* Cada usuario ha calificado al menos 20 películas
* Usuarios e películas numerados consecutivamente desde 1
* Datos ordenados aleatoriamente
* Formato: tab-separated con columnas `user_id | item_id | rating | timestamp`
* Timestamps en formato Unix seconds (segundos desde 1/1/1970 UTC)

#### `u.info`

Contiene el número de usuarios, películas y ratings en el dataset.

#### `u.item`

Información sobre películas (items). Tab-separated:

```txt
movie_id | movie_title | release_date | video_release_date |
IMDb URL | unknown | Action | Adventure | Animation |
Children's | Comedy | Crime | Documentary | Drama | Fantasy |
Film-Noir | Horror | Musical | Mystery | Romance | Sci-Fi |
Thriller | War | Western
```

* Los últimos 19 campos son géneros (1 = película de ese género, 0 = no)
* Las películas pueden pertenecer a múltiples géneros simultáneamente
* IDs de películas corresponden a los usados en `u.data`

#### `u.genre`

Lista de géneros disponibles.

#### `u.user`

Información demográfica de usuarios. Tab-separated:

```txt
user_id | age | gender | occupation | zip_code
```

* IDs de usuarios corresponden a los usados en `u.data`

#### `u.occupation`

Lista de ocupaciones disponibles.

#### `u1.base`, `u1.test` a `u5.base`, `u5.test`

* Splits 80%/20% de entrenamiento/prueba del dataset
* Cada conjunto `u1` a `u5` tiene disjuntos los conjuntos de prueba
* Utilizados para **5-fold cross-validation**
* Generados a partir de `u.data` con el script `mku.sh`

#### `ua.base`, `ua.test`, `ub.base`, `ub.test`

* Splits alternativos con exactamente **10 ratings por usuario en el conjunto de prueba**
* `ua.test` y `ub.test` son disjuntos entre sí
* Generados a partir de `u.data` con el script `mku.sh`

#### `allbut.pl`

Script que genera conjuntos de entrenamiento/prueba personalizados donde todos excepto *n* ratings de cada usuario están en entrenamiento.

#### `mku.sh`

Script de shell para generar todos los splits (`u1` a `u5`, `ua`, `ub`) a partir de `u.data`.

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
* Usuarios: información demográfica completa (por criterio de limpieza)
* Películas:

  * `release_date`: algunos faltantes → imputados
  * `video_release_date`: completamente faltante → eliminada


### 6.2 Consistencia

* No se detectaron duplicados en ratings
* IDs consistentes entre tablas
* Tipos de datos correctos tras limpieza
* IDs de usuarios y películas numerados consecutivamente a partir de 1


### 6.3 Fiabilidad

Advertencia oficial del dataset (GroupLens Research):

* **No se garantiza:** la exactitud total de los datos
* **No se asegura:** su idoneidad para todos los usos
* Los datos pueden contener imprecisiones
* Se recomienda validar los resultados antes de aplicaciones críticas


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


## 9. Uso permitido y términos de licencia

El dataset puede usarse bajo las siguientes condiciones:

* **Investigación académica:** Permitida sin restricciones
* **Uso comercial:** NO permitido sin autorización previa de GroupLens Research
* **Redistribución:** NO permitida sin permiso de la Universidad de Minnesota
* **Citación obligatoria:** Se debe citar el dataset en todas las publicaciones resultantes
* **No se puede afirmar:** Respaldo de la Universidad de Minnesota o GroupLens Research

**Contacto:** GroupLens Research Group, University of Minnesota (grouplens-info at cs.umn.edu)


## 10. Trabajos publicados que utilizan este dataset

Este dataset ha sido utilizado en investigación académica de sistemas de recomendación:

* Herlocker, J., Konstan, J., Borchers, A., Riedl, J. (1999).
  *An Algorithmic Framework for Performing Collaborative Filtering.*
  Proceedings of the 1999 Conference on Research and Development in Information Retrieval (August 1999).


## 11. Citación recomendada

> Harper, F. M., & Konstan, J. A. (2015).
> *The MovieLens Datasets: History and Context.*
> ACM Transactions on Interactive Intelligent Systems (TiiS), 5(4), Article 19 (December 2015).
> DOI: [10.1145/2827872](http://dx.doi.org/10.1145/2827872)


## 12. Contexto histórico

El **GroupLens Research Project** es un grupo de investigación en el Departamento de Ciencias de Computación e Ingeniería de la Universidad de Minnesota, dirigido por los profesores John Riedl y Joseph Konstan.

* **Inicios (1992):** Exploración de filtrado colaborativo automatizado
* **Hito (1996):** Ensayo mundial de filtrado colaborativo para noticias de Usenet
* **Legado:** Tecnología base para la fundación de Net Perceptions, Inc.
* **Actualidad:** Continúa investigando filtrado de información, sistemas de recomendación y métodos híbridos

El dataset MovieLens 100K representa un milestone importante en la investigación de sistemas de recomendación de código abierto.


## 13. Conclusión

MovieLens 100K es un dataset clásico y bien estructurado para sistemas de recomendación, con:

* buena calidad de datos
* estructura realista
* desafíos relevantes (sparsity, sesgos, cold start)

Sin embargo, sus limitaciones (sesgos demográficos, antigüedad y sparsity) deben considerarse explícitamente en el modelado y evaluación.

