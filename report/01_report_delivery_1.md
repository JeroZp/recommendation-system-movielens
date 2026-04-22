# Sistema de Recomendación con MovieLens 100K

# Entrega 1 — Problema, Datos, EDA y Baseline

## 1. Problema

### Formulación

El objetivo de este proyecto es:

> **Predecir el rating (escala 1–5) que un usuario asignaría a una película que aún no ha visto.**

Esto corresponde a un problema de:

* **Aprendizaje supervisado**
* **Regresión** (variable objetivo numérica discreta)

Formalmente:

* Entrada:
  $$x_i = (\text{usuario}, \text{película}, \text{contexto})$$
* Salida:
  $$y_i \in {1, 2, 3, 4, 5}$$

### Motivación

Este problema es representativo de los sistemas de recomendación utilizados en plataformas como Netflix o Amazon, donde:

* No todos los usuarios interactúan con todos los ítems
* Es necesario **inferir preferencias faltantes**
* La calidad de la predicción impacta directamente la experiencia del usuario

## 2. ¿Por qué este dataset es adecuado?

El dataset **MovieLens 100K** es adecuado porque:

* Contiene **interacciones reales usuario–ítem**
* Incluye:

  * Ratings explícitos (1–5)
  * Información de usuarios (edad, género, ocupación)
  * Información de películas (géneros, año)
* Tiene estructura típica de sistemas de recomendación:

  * Matriz usuario–película **altamente dispersa**
* Es un estándar académico ampliamente utilizado

Además, cumple criterios del curso:

* Permite construir un baseline claro
* Permite comparar múltiples enfoques (colaborativo, contenido, híbrido)
* Es manejable computacionalmente

## 3. Análisis Exploratorio de Datos (EDA)

### 3.1 Calidad de los datos

Hallazgos principales:

* **Ratings**

  * Sin valores nulos
  * Sin duplicados
  * Rango válido: 1–5

* **Users**

  * Sin valores faltantes

* **Movies**

  * `video_release_date`: completamente nula → eliminada
  * `imdb_url`: sin valor predictivo → eliminada
  * `release_date`: 1 valor faltante → imputado con la mediana

Esto indica que el dataset tiene **alta calidad estructural**.

### 3.2 Distribución de ratings

Hallazgos:

* Media ≈ **3.5**
* Mediana = **4**
* Alta concentración en ratings **3 y 4**

**Interpretación:**

* Existe un **sesgo de positividad**
* Un modelo simple basado en la media ya captura parte del comportamiento

### 3.3 Actividad de usuarios y películas

* Distribuciones con **cola larga**
* Pocos usuarios generan muchas interacciones
* Pocas películas concentran la mayoría de ratings

**Sparsity:**

* ~ **93.7% de la matriz está vacía**

**Implicaciones:**

* Problema típico de recomendación
* Dificultad estructural:

  * **Cold start**
  * Baja cobertura de datos

### 3.4 Demografía de usuarios

Hallazgos:

* ~75% hombres
* Edad:

  * Media ≈ 34
  * Mediana ≈ 31
* Alta proporción de estudiantes

**Implicaciones:**

* Dataset **no balanceado demográficamente**
* El modelo aprenderá principalmente:

  * Preferencias de hombres jóvenes
* Riesgo de sesgo en recomendaciones

### 3.5 Popularidad de películas

* Alta concentración de ratings en:

  * Películas populares
  * Estrenos de los años 90

**Interpretación:**

* Existe un sesgo temporal
* Popularidad influye fuertemente en los datos observados

## 4. Definición del problema de modelado

* **Tipo de tarea:** Regresión
* **Variable objetivo:** `rating`
* **Features posibles:**

  * Usuario: edad, género, ocupación
  * Película: géneros, año
  * Interacción: user_id, item_id

## 5. Métrica

Se utilizan:

* **RMSE (Root Mean Squared Error)**
* **MAE (Mean Absolute Error)**

### Justificación

* El problema es de **predicción numérica**
* RMSE penaliza más los errores grandes → útil cuando:

  * Fallar mucho es peor que fallar poco
* MAE es más robusto → interpretación directa

## 6. Baseline

### Definición

El baseline es:

> **Predecir el rating promedio global del dataset para todas las observaciones**

Formalmente:
$$
\hat{y} = \bar{y}
$$

### Justificación

* Es el modelo más simple posible
* No utiliza información de usuario ni ítem
* Sirve como referencia mínima

Dado el EDA:

* La media ≈ 3.5
* Los ratings están concentrados alrededor de este valor

→ Este baseline ya captura una parte importante del comportamiento

### Interpretación del baseline

* Si un modelo más complejo **no mejora este baseline**, entonces:

  * No está aprendiendo patrones útiles
* Si mejora significativamente:

  * Existe señal explotable en los datos

## 7. ¿Qué tan difícil parece el problema?

El problema presenta varias dificultades:

### 1. Alta sparsity (~93.7%)

* Muy pocas interacciones observadas
* Difícil generalizar

### 2. Cola larga

* Muchos usuarios con pocos datos
* Muchas películas con pocas observaciones

### 3. Sesgos en los datos

* Demográfico (género, edad)
* Popularidad
* Temporal

### 4. Naturaleza subjetiva

* El rating es altamente dependiente del usuario

### Conclusión de dificultad

* **No es un problema trivial**

* Pero tampoco es extremadamente complejo:

* El baseline (media) ya captura parte del patrón

* Hay estructura explotable:

  * preferencias por género
  * popularidad
  * comportamiento de usuarios

## 8. Primeras conclusiones

1. El dataset es de alta calidad y adecuado para el problema
2. Existe una estructura clara en los datos:

   * sesgo positivo en ratings
   * sparsity alta
   * cola larga
3. El baseline basado en la media es fuerte debido a la distribución
4. El problema requiere modelos que:

   * manejen sparsity
   * capturen interacciones usuario–ítem
5. Hay riesgos importantes:

   * sesgo demográfico
   * falta de representatividad
