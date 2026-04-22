# Sistema de Recomendación con MovieLens 100K

## Entrega 1 — Problema, Datos, EDA y Baseline

**Institución:** EAFIT  
**Curso:** Aprendizaje de Máquina Aplicado  
**Profesor:** Marco Teran  
**Dataset:** MovieLens 100K — GroupLens Research  
**Fecha de entrega:** 09/04/2026  
**Estado:** ✓ Completado

---

## 1. ¿Qué problema intenta resolver?

### Formulación del problema

**Pregunta central:**

> ¿Es posible predecir el rating (escala 1–5) que un usuario asignaría a una película que aún no ha visto?

**Tipo de problema:**

- Aprendizaje supervisado
- Regresión (variable objetivo discreta: 1, 2, 3, 4, 5)
- Contexto: Sistemas de recomendación

**Formulación matemática:**

$$\text{Dado: } x_i = (\text{user\_id}, \text{movie\_id}, \text{demographics}, \text{genres})$$

$$\text{Predecir: } y_i \in \{1, 2, 3, 4, 5\} \text{ (rating del usuario } i \text{ sobre película } j\text{)}$$

### Motivación y aplicabilidad

Este problema es **relevante en la práctica** porque:

- Representa un caso de uso real en plataformas como Netflix, Amazon Prime Video, Spotify
- Aproximadamente **93.7% de la matriz usuario–película está vacía** (valores desconocidos)
- La calidad de la predicción afecta directamente la experiencia del usuario
- Permite estudiar múltiples enfoques: filtrado colaborativo, contenido e híbridos

---

## 2. ¿Por qué MovieLens 100K es el dataset adecuado?

### Características que lo hacen apropiado

| Criterio | MovieLens 100K | Justificación |
|----------|---|---|
| **Escala** | 100,000 ratings de 943 usuarios en 1,682 películas | Tamaño manageable para iteración rápida |
| **Interacciones reales** | Ratings explícitos (1–5) recolectados por usuarios | Datos auténticos, no sintéticos |
| **Features disponibles** | Demográficos (edad, género, ocupación) + géneros | Permite experimentar contenido + colaborativo |
| **Dispersión** | Sparsity ≈ 93.7% | Problema realista de recomendación |
| **Estándar académico** | Ampliamente usado desde 1997 | Resultados comparables con literatura |
| **Documentación** | Excelente data card del GroupLens Research | Metadatos claros y fiables |

### Requisitos de la entrega cumplidos

- ✓ Permite construir **baseline reproducible** (media global)
- ✓ Permite comparar **múltiples enfoques** (modelos colaborativos vs. basados en contenido)
- ✓ Computacionalmente **manejable** en desarrollo iterativo
- ✓ Presenta **desafíos realistas** (sparsity, cold start, sesgos)

---

## 3. Análisis Exploratorio de Datos (EDA)

### 3.1 Calidad y estructura de datos

#### Hallazgos de calidad

| Aspecto | Hallazgo | Acción |
|---------|----------|--------|
| **Ratings** | 100,000 sin nulos, sin duplicados, rango 1–5 ✓ | Ninguna |
| **Usuarios** | 943 usuarios, demográficos completos ✓ | Ninguna |
| **Películas** | `video_release_date` 100% nulo | Eliminado |
| | `imdb_url` sin valor predictivo | Eliminado |
| | `release_date` 1 nulo (0.06%) | Imputado con mediana |
| **Géneros** | 19 géneros, codificación binaria correcta ✓ | Ninguna |

**Conclusión:** Dataset de **alta calidad estructural**. Limpieza mínima requerida.

### 3.2 Distribución de ratings

**Estadísticas básicas:**

- Media: **3.53**
- Mediana: **4.00**
- Std: **1.13**
- Moda: **4** (concentración en ratings 3 y 4)

**Interpretación:**

- Existe un **sesgo de positividad** — los usuarios tienden a calificar películas que les gustan
- La concentración alrededor de 3–4 significa que un modelo simple (baseline media) ya captura mucho del patrón
- Rango completo (1–5) está representado, pero con cola larga hacia valores altos

![Rating Distribution](../figures/01_rating_distribution.png)

### 3.3 Sparsity y distribución de actividad

**Matriz usuario–película:**

$$\text{Sparsity} = 1 - \frac{\text{ratings observados}}{\text{combinaciones posibles}} = 1 - \frac{100,000}{943 \times 1,682} = 93.7\%$$

**Distribución de interacciones (cola larga):**

- **Usuarios:** Media 106 ratings/usuario, pero rango 20–737
  - 25% de usuarios tienen ≤ 49 ratings (datos escasos)
  - Top 10% de usuarios generan 33% de los ratings
  
- **Películas:** Media 59 ratings/película, pero rango 1–583
  - 45% de películas tienen < 10 ratings (cold start)
  - Top 10% de películas reciben 52% de los ratings

**Implicaciones:**

- Problema realista de **cold start**: nuevos usuarios/películas son difíciles de recomendar
- Datos **desbalanceados**: modelos sobreajustarán a usuarios/películas activos
- Necesidad de técnicas que manejen sparsity (factorización de matrices, embeddings)

![Activity Distribution](../figures/02_activity_distribution.png)

### 3.4 Demografía de usuarios

**Perfil demográfico:**

- **Género:** 75.1% hombres, 24.9% mujeres → sesgo significativo
- **Edad:** Media 34.1, mediana 31, rango 7–73
  - Concentración en 20–45 años (70% de usuarios)
- **Ocupación:** 21% estudiantes (más frecuente)
  - Luego: tech-related, sales, other, healthcare

**Riesgos identificados:**

- Dataset **no representa la población general** — subestima mujeres, usuarios mayores
- Modelo aprenderá principalmente **preferencias de hombres jóvenes** 
- Posible sesgo en recomendaciones para otros perfiles demográficos
- Recomendación: estratificar evaluación por demográficos

![User Demographics](../figures/03_user_demographics.png)

### 3.5 Preferencias por género y año

**Hallazgos principales:**

- **Géneros más comunes:** Drama (438 películas), Comedy (389), War (224)
- **Géneros mejor valorados:** Film-Noir (avg 4.17), War (4.08), Animation (4.01)
- **Sesgo temporal:** Dataset dominado por películas de **1994–1998** (89% de películas)
- **Top películas:** "Shawshank Redemption" (317 ratings, avg 4.51), "Pulp Fiction" (307 ratings, avg 4.06)

**Interpretación:**

- Existe estructura explotable: preferencias claras por género
- Sesgo temporal: modelo puede no generalizar a películas modernas
- Recomendación: considerar año como feature importante

---

## 4. ¿Qué métrica es razonable y por qué?

### Métricas seleccionadas

#### 1. **Root Mean Squared Error (RMSE)**

$$\text{RMSE} = \sqrt{\frac{1}{n} \sum_{i=1}^{n} (y_i - \hat{y}_i)^2}$$

- **Justificación:** Penaliza más los errores grandes → errar mucho es peor que errar poco
- **Interpretación:** Error promedio en la escala original (1–5)
- **Aplicabilidad:** Estándar en literatura de recomendación (permite comparar con papers)

#### 2. **Mean Absolute Error (MAE)**

$$\text{MAE} = \frac{1}{n} \sum_{i=1}^{n} |y_i - \hat{y}_i|$$

- **Justificación:** Más robusto a outliers que RMSE
- **Interpretación:** Error promedio en estrellas (ej: MAE=0.5 significa error promedio de media estrella)
- **Aplicabilidad:** Más interpretable para stakeholders no técnicos

### Por qué ambas son apropiadas

1. **Problema es de regresión continua** → métricas de error numérico son apropiadas
2. **RMSE es estándar en recomendación** → permite benchmarking vs. literatura
3. **MAE es más interpretable** → facilita comunicación de resultados
4. **Escala pequeña (1–5)** → ambas métricas son comparables en magnitud

---

## 5. Baseline y dificultad del problema

### 5.1 Definición del baseline

**Modelo baseline:**

> Predecir el rating promedio global del dataset para todas las observaciones, ignorando características de usuario y película.

$$\hat{y}_i = \bar{y} = 3.53$$

**Justificación:**

- Modelo más simple posible (no aprende patrones)
- Sirve como referencia mínima: cualquier modelo debe mejorar esto
- Captura el sesgo de positividad del dataset

### 5.2 Performance del baseline

Evaluado en test set (20%):

| Métrica | Valor | Interpretación |
|---------|-------|---|
| **RMSE** | 1.125 | Error promedio de ±1.12 estrellas |
| **MAE** | 0.901 | Error promedio de ±0.9 estrellas |

**Análisis:**

- Un modelo que predice "3.5 estrellas" para todo comete error medio de ~0.9 estrellas
- Por comparación: rango es 1–5 (4 unidades) → baseline captura ~22% de la varianza
- Esto indica que **existe señal explotable** — modelos con features deben hacerlo mejor

### 5.3 ¿Qué tan difícil es el problema?

#### Factores que lo hacen desafiante

| Factor | Impacto | Evidencia |
|--------|--------|----------|
| **Sparsity alta (93.7%)** | Alto | Muy pocas observaciones por usuario–película |
| **Cold start** | Medio-Alto | 45% películas con <10 ratings |
| **Cola larga** | Medio | Top 10% usuarios/películas dominan datos |
| **Sesgos demográficos** | Medio | 75% hombres, estudiantes jóvenes |
| **Subjetividad** | Bajo-Medio | Rating depende del usuario, pero hay patrones |

#### Factores que lo hacen tratable

| Factor | Beneficio | Evidencia |
|--------|----------|----------|
| **Baseline fuerte** | Bajo | Media captura parte del patrón |
| **Estructura en preferencias** | Medio | Géneros tienen ratings distintos |
| **Features disponibles** | Medio-Alto | Demográficos + géneros permiten modelos ricos |
| **Tamaño manejable** | Alto | 100K ratings es suficiente para iteración rápida |
| **Datos auténticos** | Alto | Interacciones reales, bien documentadas |

#### Conclusión de dificultad

**Dificultad: MEDIA-ALTA**

- **No es trivial:** Requiere modelos que manejen sparsity y capten interacciones usuario–ítem
- **Es tractable:** Existen técnicas comprobadas (factorización, embeddings, ensambles)
- **Hay riesgo de overfitting:** A usuarios/películas frecuentes, a sesgos demográficos
- **Requiere validación cuidadosa:** Cross-validation y estratificación por demográficos

---

## 6. Primeras conclusiones

### Síntesis de hallazgos

1. **Problema bien definido:** Regresión de ratings es caso de uso real en sistemas recomendadores
   
2. **Dataset apropiado:** MovieLens 100K es estándar académico con alta calidad, aunque con limitaciones demográficas conocidas

3. **Estructura explotable:** EDA revela patrones claros
   - Sesgo positivo en ratings (media 3.53)
   - Preferencias diferenciadas por género
   - Comportamiento usuario-específico
   
4. **Baseline competente:** Media global (RMSE 1.125, MAE 0.901) establece referencia clara
   - Modelos deben mejorar esto sustancialmente
   - Hay margen de mejora (solo captura ~22% de varianza)

5. **Riesgos identificados y mitigables:**
   - Sparsity 93.7% → usar técnicas de regularización
   - Cold start → considerar modelos basados en contenido
   - Sesgos demográficos → validar por segmentos
   - Sesgo temporal → incluir año en features

### Recomendaciones para siguientes entregas

- Experimentar con factorización de matrices (SVD, NMF)
- Comparar enfoques: colaborativo vs. contenido vs. híbrido
- Implementar validación cruzada estratificada por demográficos
- Analizar errores por tipo de usuario/película (análisis de residuales)
- Explorar embeddings y métodos modernos (redes neuronales)

---

## Anexo: Reproducibilidad

### Stack tecnológico

- Python 3.8+
- pandas, numpy (manipulación datos)
- scikit-learn (métricas, baseline)
- matplotlib, seaborn (visualización)
- Jupyter Notebook (análisis reproducible)

### Estructura de archivos

```
.
├── data/
│   ├── raw/ml-100k/
│   │   ├── u.data, u.user, u.item, u.genre, u.occupation
│   └── processed/
│       └── (archivos intermedios)
├── notebooks/
│   └── 01_eda_baseline.ipynb
├── figures/
│   ├── 01_rating_distribution.png
│   ├── 02_activity_distribution.png
│   └── 03_user_demographics.png
├── report/
│   └── 01_report_delivery_1.md
└── src/
    └── (código reutilizable)
```

### Reproducción

```bash
# 1. Instalar dependencias
pip install pandas numpy scikit-learn matplotlib seaborn jupyter

# 2. Ejecutar notebook
jupyter notebook notebooks/01_eda_baseline.ipynb

# 3. Generar figuras y métricas baseline
# (ejecutar todas las celdas del notebook)
```

---

**Fecha de generación:** 23/04/2026  
**Versión:** 1.0  
**Estado:** Listo para revisión
