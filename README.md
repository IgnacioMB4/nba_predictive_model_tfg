# NBA Betting Predictive Model

Modelo predictivo de resultados de la NBA con evaluación económica frente a la línea de las casas de apuestas.

Trabajo Fin de Grado en Ingeniería Informática, ETSI Informáticos, Universidad Politécnica de Madrid.
Autor: **Ignacio Marina Baselga** ([@IgnacioMB4](https://github.com/IgnacioMB4)).

## Descripción

Este proyecto implementa un *pipeline* completo que va desde la descarga de datos oficiales de la NBA hasta la simulación económica de estrategias de apuesta. Sobre los partidos de las temporadas 2006-07 a 2024-25 se entrena un modelo XGBoost regularizado, con ingeniería de *features* a nivel de equipo y de jugador (agregación ponderada por minutos sobre los jugadores convocados), se interpreta con SHAP y se compara con la línea de cierre de las casas de apuestas.

## Estructura del proyecto

```
.
├── api_nba.ipynb              # Descarga por lotes desde la NBA API oficial
├── datasets_analysis.ipynb    # Análisis y validación de datasets públicos de Kaggle
├── dataset.ipynb              # Construcción de la tabla master e ingeniería de features
├── model.ipynb                # Comparación de algoritmos, tuning, entrenamiento y SHAP
├── bet.ipynb                  # Procesamiento de cuotas y simulación de estrategias
├── datasets/                  # CSV crudos, datasets intermedios y master.csv
├── requirements.txt           # Dependencias Python
└── README.md                  # Este fichero
```

## Requisitos

- Python 3.10 o superior.
- 16 GB de RAM recomendados para el cálculo de *rolling* sobre la tabla de jugadores.
- Conexión a internet para la descarga inicial de datos vía `nba_api`.

## Instalación

```bash
git clone https://github.com/IgnacioMB4/nba_betting_predictive_model.git
cd nba_betting_predictive_model
python -m venv venv
source venv/bin/activate      # en Windows: venv\Scripts\activate
pip install -r requirements.txt
```

## Datos

El proyecto utiliza dos fuentes externas.

### 1. Estadísticas oficiales NBA

Se descargan directamente desde la API oficial mediante el paquete [`nba_api`](https://github.com/swar/nba_api). No requiere autenticación. El notebook `api_nba.ipynb` genera los seis CSV crudos (equipo y jugador × *base* / *advanced* / *misc*) en la carpeta `datasets/`. La descarga completa cubre las temporadas 2006-07 a 2024-25 y es el paso más lento del *pipeline* por la limitación de peticiones de la API.

### 2. Cuotas históricas del mercado

Se utiliza el *dataset* público [**NBA Betting Data (October 2007 to June 2025)**](https://www.kaggle.com/datasets/cviaxmiwnptr/nba-betting-data-october-2007-to-june-2024) disponible en Kaggle bajo licencia CC0. Contiene las cuotas *moneyline*, *spread* y total de puntos de todos los partidos NBA en el intervalo indicado. En este proyecto se utiliza el *moneyline* y solo el periodo con cobertura completa (2008-2022).

Tres formas de descargarlo:

- **Desde el navegador**. Iniciar sesión en Kaggle y pulsar «Download» en la página del *dataset*. Extraer los CSV en `datasets/`.
- **Con la CLI de Kaggle**:
```bash
  kaggle datasets download -d cviaxmiwnptr/nba-betting-data-october-2007-to-june-2024
  unzip nba-betting-data-october-2007-to-june-2024.zip -d datasets/
```
  Requiere generar el fichero `kaggle.json` desde la configuración del perfil.
- **Con `kagglehub` desde Python**:
```python
  import kagglehub
  path = kagglehub.dataset_download(
      "cviaxmiwnptr/nba-betting-data-october-2007-to-june-2024"
  )
  print(path)
```

## Uso

Los notebooks están pensados para ejecutarse en este orden.

1. **`api_nba.ipynb`** genera los seis CSV crudos en `datasets/` a partir de la NBA API.
2. **`datasets_analysis.ipynb`** analiza los *datasets* públicos de Kaggle utilizados en la fase de exploración, con los cuatro modelos comparados y el chequeo de *leakage*.
3. **`dataset.ipynb`** parte de los CSV crudos y produce `master.csv` con las 168 *features* del modelo final.
4. **`model.ipynb`** carga `master.csv`, compara los cuatro algoritmos (LogReg, Random Forest, XGBoost, HistGradientBoosting), entrena el modelo final y produce el *ranking* SHAP.
5. **`bet.ipynb`** cruza las predicciones con las cuotas históricas y simula las estrategias de apuesta.

## Configuración del modelo final

```python
XGBClassifier(
    n_estimators = 300,
    max_depth = 2,
    learning_rate = 0.08,
    min_child_weight = 15,
    gamma = 0.5,
    reg_lambda = 10,
    eval_metric = 'logloss',
    random_state = 42
)
```

Las *features* de entrada son todas las columnas que empiezan por `diff_` en la tabla `master`, más `bubble`, `importance_max` e `importance_diff`. El total son 168 *features*. La variable objetivo es `home_team_won`.

## Reproducibilidad

Todos los generadores aleatorios usan `random_state = 42`. La ejecución completa del *pipeline* desde los CSV crudos hasta las gráficas finales tarda entre 15 y 30 minutos en un equipo doméstico con 16 GB de RAM. El paso más costoso es el cálculo de *rolling* y *season prior* sobre la tabla de jugadores (477.193 filas).

## Memoria del TFG

La memoria completa del trabajo, con la revisión del estado del arte, el análisis, el diseño, la implementación y la evaluación económica, se encuentra en el propio repositorio o disponible bajo petición al autor.

## Licencia

Código publicado con fines académicos. Uso libre para investigación y aprendizaje.

## Contacto

Ignacio Marina Baselga — [@IgnacioMB4](https://github.com/IgnacioMB4)
