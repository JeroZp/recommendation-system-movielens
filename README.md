# Sistema de Recomendación de Películas — MovieLens 100K

**Curso:** Aprendizaje de Máquina Aplicado — EAFIT  
**Profesor:** Marco Teran  
**Dataset:** [MovieLens 100K](https://grouplens.org/datasets/movielens/100k/)  

## Pregunta central
¿Es posible predecir el rating que un usuario le daría a una película que no ha visto?

## Estructura del repositorio
recommendation-system-movielens/
├── data/
│   ├── raw/          # Datos originales sin modificar
│   └── processed/    # Datos limpios y transformados
├── notebooks/        # Jupyter Notebooks del proyecto
├── figures/          # Gráficas exportadas
├── report/           # Reporte PDF
├── poster/           # Póster visual
├── src/              # Módulos Python opcionales
├── requirements.txt
└── README.md

## Cómo reproducir
```bash
# 1. Clonar el repositorio
git clone https://github.com/JeroZp/recommendation-system-movielens.git
cd recommendation-system-movielens

# 2. Crear y activar entorno virtual
python -m venv venv

# Windows PowerShell
venv\Scripts\Activate.ps1

# Linux/macOS
source venv/bin/activate
# 3. Instalar dependencias
pip install -r requirements.txt

# 4. Descargar el dataset manualmente desde:
# https://files.grouplens.org/datasets/movielens/ml-100k.zip
# y descomprimirlo en data/raw/

# 5. Abrir el notebook
jupyter notebook notebooks/
```
