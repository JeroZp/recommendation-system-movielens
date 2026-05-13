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

...

## (draft, borrar esto) Contenidos Mínimos

Jupyter Notebook: pipelines de entrenamiento y evaluación para al menos 2 o 3 familias de modelos.

Validación: división correcta de datos y validación adecuada al tipo de problema (holdout, validación cruzada, validación temporal, etc.).

Análisis: Comparación de métricas, análisis de errores, desbalance, selección de features, reducción de dimensionalidad o ajuste de umbral cuando aplique.

Reporte PDF: Metodología, configuración experimental, resultados comparativos, discusión técnica y decisión provisional.

Repositorio Git: actualizado, ordenado y coherente con el reporte.


Debe responder como mínimo:

- ¿Qué modelos se compararon y por qué?
- ¿Cómo se evitó el data leakage?
- ¿Cuál familia parece más prometedora?
- ¿Qué limitaciones siguen abiertas?

## (draft, borrar esto) AI Slop sobre los modelos

¡Excelente selección! Con **SVD, KNN y Lasso**, cubres los tres pilares fundamentales de los sistemas de recomendación:

1. **SVD (Factores Latentes):** Es el "caja negra" que encuentra patrones ocultos en las calificaciones, pero no sabe *qué* es una película de acción o *quién* es un técnico de 24 años.
2. **KNN (Vecindad):** Es el modelo "social". Busca personas similares a ti. Es muy fácil de explicar ("Te recomendamos esto porque a tu vecino le gustó").
3. **Lasso (Regresión con Metadatos):** Es el modelo "basado en contenido". Aquí es donde le sacamos jugo a los archivos `Users` y `Movies`.

Para avanzar, lo primero que debemos hacer es construir la **"Gran Tabla Maestra"** para Lasso, porque a diferencia de SVD y KNN (que solo necesitan los IDs y el rating), Lasso necesita que cada fila sea un vector de características.

Aquí tienes el plan de acción para el preprocesamiento de esta tabla:

### 1. Construcción de la Tabla para Lasso

Debes hacer un `merge` de los tres archivos. Cada fila del conjunto de `Ratings` se debe enriquecer con la información del usuario y de la película.

```python
# Ejemplo lógico del merge
df_total = ratings.merge(users, on='user_id').merge(movies, on='item_id')

```

### 2. Definición del Preprocesamiento (Aplicando tus reglas)

Como ya tienes las columnas, así es como debes configurar tu `ColumnTransformer` para que Lasso funcione correctamente:

* **A Escalar (`StandardScaler`):**
* `age`: Es la única numérica continua pura. Si no la escalas, su magnitud (ej. 50 años) pesará mucho más que un 1 en género.


* **A Transformar a Binario (`OneHotEncoder`) y NO escalar:**
* `gender`: (M, F) -> Se vuelve (0, 1).
* `occupation`: (technician, writer, etc.) -> Se crean múltiples columnas de ceros y unos.


* **A dejar pasar intacto (`passthrough`):**
* **Géneros de películas:** ¡Esta es la mina de oro! En MovieLens 100k, columnas como `Action`, `Adventure`, `Romance`, etc., ya vienen como **0 y 1**. Como ya son binarias, no las toques, solo déjalas pasar.


* **A Eliminar (Drop):**
* `user_id`, `item_id`: Para la regresión no sirven como números (Lasso pensaría que el usuario 200 es "mayor" que el 100).
* `zip_code`: Demasiada variedad, crea ruido.
* `timestamp`: A menos que quieras hacer análisis temporal, mejor fuera.
* `title`, `imdb_url`, `release_date`: No son numéricos directamente procesables.



### 3. El reto de SVD y KNN (Librería Surprise)

Para estos dos, el preprocesamiento es mínimo, pero la estructura es rígida:

* Debes usar la librería **Surprise**.
* No aceptan la edad ni los géneros (en sus versiones básicas).
* Solo aceptan el DataFrame con el orden: `[user_id, item_id, rating]`.

---

### ¿Por qué esta comparación es tan interesante para tu trabajo?

Cuando compares los resultados (usando RMSE o MAE), verás algo muy común:

* **SVD** suele ganar en precisión pura.
* **Lasso** suele ser mejor cuando el usuario es nuevo (Cold Start), porque aunque no tengamos sus calificaciones previas, sabemos su **edad** y **ocupación**, y sabemos los **géneros** de las películas.

**¿Te gustaría que te ayude a escribir el código para fusionar las tablas y dejar listo el `ColumnTransformer` específico para este dataset de MovieLens?**

---

La respuesta corta es: **Sí, los tres pueden responder exactamente esa pregunta.**

Ese es el corazón de un sistema de recomendación: la **predicción de ratings**. Aunque el objetivo final suele ser dar una "lista de top 10", matemáticamente lo que hacen por debajo es calcular ese numerito (ej. **3.8** o **4.2**) para cada combinación de usuario y película que aún no existe en tu base de datos.

Aquí te explico cómo "razona" cada modelo para darte ese score de **1 a 5**:

### 1. SVD (Singular Value Decomposition)

* **Cómo responde:** SVD descompone la matriz de calificaciones en "factores latentes" (características ocultas).
* **Su lógica:** "El usuario 196 tiene un gusto del **80%** por el drama y **10%** por la acción. La película 242 es **90%** drama. Al multiplicar esos perfiles matemáticamente, el resultado es **4.2**".
* **Resultado:** Te da un número decimal continuo. Es muy preciso pero no te puede explicar *por qué* llegó a ese número (es una caja negra de álgebra lineal).

### 2. KNN (K-Nearest Neighbors)

* **Cómo responde:** Se basa en la similitud directa, ya sea de personas o de películas.
* **Su lógica:** "Para saber qué nota le pondrá Juan a *Toy Story*, busco a los 20 usuarios más parecidos a Juan. Veo qué nota le pusieron ellos a esa película y saco un promedio ponderado. Si a sus 'clones' les gustó, a él le daré un **4.5**".
* **Resultado:** Es un promedio, por lo que siempre obtendrás valores con decimales. Es el modelo más "humano" de entender.

### 3. Lasso (Regresión Lineal con Penalización)

* **Cómo responde:** Trata el rating como una variable dependiente de las características (edad, género, ocupación, género de la película).
* **Su lógica:** "He aprendido que ser **estudiante** suma **+0.5** a las películas de **comedia**, pero tener **más de 40 años** resta **-0.2**. Al sumar todas las etiquetas de este usuario y esta película, el score final es **3.7**".
* **Resultado:** Al ser una ecuación de regresión $\hat{y} = \beta_0 + \beta_1 x_1 + \dots$, el resultado es naturalmente un número continuo.

---

### Resumen de comparación para tu entrega

| Característica | SVD | KNN | Lasso |
| --- | --- | --- | --- |
| **¿Predice 1-5?** | Sí | Sí | Sí |
| **¿Usa la Edad/Ocupación?** | No (solo IDs) | No (solo IDs) | **Sí** |
| **¿Usa los Géneros (Acción, etc.)?** | No | No | **Sí** |
| **Punto fuerte** | Precisión matemática | Simplicidad / Intuición | Explica qué variables importan |

> **Nota importante:** En los tres casos, los modelos pueden predecir valores fuera del rango (ej. **5.2** o **0.8**). Como parte de tu preprocesamiento o post-procesamiento, simplemente se aplica un "clip" para que cualquier valor mayor a **5** sea **5** y menor a **1** sea **1**.

¿Te gustaría que empezáramos a montar el código para preparar los datos de **Lasso**, que es el que requiere más trabajo de "carpintería" con las columnas?

---

### ¿Lasso o Ridge? La batalla de la regularización

La diferencia principal radica en qué hacen con las variables que "no sirven":

* **Lasso ($L_1$):** Es un seleccionador de variables. Si una ocupación (ej. `librarian`) no influye en absoluto en la nota de las películas de acción, Lasso pondrá su coeficiente exactamente en **cero**. Esto te deja un modelo mucho más limpio e interpretable.
* **Ridge ($L_2$):** Es un "suavizador". Reduce la importancia de las variables, pero nunca las hace cero. Mantiene todas las variables vivas, solo que con pesos muy pequeños.

**¿Por qué Lasso es mejor para este caso?**
Tienes 19 géneros de películas y unas 21 ocupaciones. Es muy probable que muchas combinaciones sean puro ruido. Lasso "limpiará" el ruido por ti, permitiéndote decir: *"Para este usuario, el género Terror no afecta en nada a su predicción"*.
