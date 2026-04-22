# 🎬 Sistema de Recomendación de Películas — MovieLens 100K

**Curso:** Aprendizaje de Máquina Aplicado — EAFIT  
**Profesor:** Marco Terán  
**Dataset:** [MovieLens 100K](https://grouplens.org/datasets/movielens/100k/)

---

## ❓ Pregunta central
¿Es posible predecir el rating que un usuario le daría a una película que no ha visto?

## Stack tecnológico

- **Python** 3.8+
- **Data manipulation:** pandas, numpy
- **ML:** scikit-learn (métricas, baseline, cross-validation)
- **Visualization:** matplotlib, seaborn
- **Reproducible:** Jupyter Notebook

## 📁 Estructura del repositorio

```bash
recommendation-system-movielens/
│
├── data/
│   ├── raw/          # Datos originales sin modificar
│   ├── processed/    # Datos limpios y transformados
│   └── README.md     # Data Card con detalles del dataset
│
├── notebooks/        # Jupyter Notebooks del proyecto
│   └── 01_eda_baseline.ipynb    # Análisis reproducible, 1ra entrega
├── figures/          # Gráficas exportadas
├── report/           # Reporte PDF
│   ├── 01_report_delivery_1.md  # Reporte editable, 1ra entrega 
│   └── 01_report_delivery_1.pdf # Reporte definitivo, 1ra entrega 
├── poster/           # Póster visual
├── src/              # Módulos Python (opcional)
│
├── requirements.txt
└── README.md
```

---

## ⚙️ Cómo reproducir el proyecto

### 1. Clonar el repositorio
```bash
git clone https://github.com/JeroZp/recommendation-system-movielens.git
cd recommendation-system-movielens
```

### 2. Crear entorno virtual
```bash
python -m venv venv
```

### 3. Activar entorno

**Windows (PowerShell):**
```bash
venv\Scripts\Activate.ps1
```

**Linux / macOS:**
```bash
source venv/bin/activate
```

### 4. Instalar dependencias
```bash
pip install -r requirements.txt
```

### 5. Descargar dataset

Descargar desde:  
👉 https://files.grouplens.org/datasets/movielens/ml-100k.zip  

Y descomprimir en:
```
data/raw/
```

### 6. Ejecutar notebooks
```bash
jupyter notebook notebooks/
```

---

## 📌 Notas
- Asegúrate de tener Python 3.10+ instalado.
