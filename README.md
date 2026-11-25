# Churn en plataformas digitales: preparación y estructuración de datos

Proyecto individual de ciencia de datos (SENA, Tecnólogo en Análisis y Desarrollo de Software, ficha 3223873) sobre el **análisis y la predicción de la cancelación de suscripciones (churn)**.

**Aprendiz:** Nicolás Muñoz Hernández · CTMA – SENA, Regional Antioquia

## Pregunta analítica

¿Qué patrones de interacción, frecuencia de uso y nivel de soporte técnico en los primeros 30 días permiten predecir el riesgo de cancelación?

> Ningún dataset trae una ventana de 30 días (lo comprobó el diagnóstico de calidad), así que el análisis usa el comportamiento acumulado: uso, inicios de sesión, tickets y llamadas de soporte, y fallos de pago.

## Fuentes de datos

| Dataset | Archivo | Filas | Columnas | Fuente |
|---|---|---|---|---|
| Suscripción (SaaS, sintético) | `data/raw/customer_subscription_churn_usage_patterns.csv` | 2.800 | 10 | Google Dataset Search |
| Telco (telecomunicaciones) | `data/raw/churn.csv` | 4.250 | 21 | Kaggle / BigQuery |

Los archivos originales se conservan **sin modificar** en `data/raw/`.

## Estructura del proyecto

```
proyecto-churn/
├── README.md
├── .gitignore
├── requirements.txt
├── data/
│   ├── raw/                  # datasets originales
│   └── processed/            # datasets limpios y estructurados (generados)
├── notebooks/
│   ├── data_quality.ipynb        # diagnóstico de calidad (actividad anterior)
│   └── data_preparation.ipynb    # limpieza, análisis y estructuración
└── reports/
    └── data_quality.md           # informe del diagnóstico (actividad anterior)
```

## Cómo ejecutarlo (Windows)

```
git clone https://github.com/nicolasmunosh/proyecto-churn.git
cd proyecto-churn
python -m venv .venv
.venv\Scripts\activate.bat
python -m pip install -r requirements.txt
```

Después abre `notebooks/data_preparation.ipynb` en VS Code, elige el kernel `.venv` y ejecuta **Run All**. El cuaderno lee `data/raw/` y escribe en `data/processed/`. Requiere `pyarrow` para guardar Parquet (incluido en `requirements.txt`).

## Transformaciones realizadas

**Limpieza y estandarización (Lección 2)**

| Tema | Resultado | Decisión |
|---|---|---|
| Duplicados | 0 exactos, 0 por identificador, 0 ignorando el identificador | No se elimina nada; la función queda en el pipeline |
| Nulos | 0 (ni vacíos ni `NA`) | No se imputa |
| Ceros sospechosos | 2 clientes de Telco con 0 minutos diurnos | Se conservan y se marcan con `zero_day_usage` |
| Tipos | `signup_date` texto, `churn` en dos formatos | `signup_date` a fecha; `churn` a 0/1 en ambos; `plan_type` categoría ordenada; enteros reducidos |
| Texto | `area_code` como `area_code_510`, textos sin normalizar | Se deja el número; `state` en mayúsculas |

Se usan funciones reutilizables encadenadas con `pipe()`, un registro de cambios por paso y validaciones con `assert`.

**Análisis (Lección 3)**

Filtros (booleanos, `isin`, `between`, `query`), ordenamiento, `groupby`/`agg`, tablas dinámicas y seis indicadores de tasa de churn:

| Indicador | Dataset | Variable |
|---|---|---|
| IND-1 | Suscripción | Uso semanal |
| IND-2 | Suscripción | Días sin iniciar sesión |
| IND-3 | Suscripción | Tickets de soporte |
| IND-4 | Suscripción | Fallos de pago |
| IND-5 | Telco | Llamadas a servicio al cliente |
| IND-6 | Telco | Uso diurno |

**Estructuración (Lección 4)**

- Los dos datasets **no se combinan**: no comparten una clave (`user_id` es un entero y `customer_id` un UUID), no se solapan y son negocios distintos. Unirlos sería una combinación artificial.
- Cada dataset se estructura por separado: variables derivadas (`total_minutes`, `total_calls`, `total_charge`), categorías, banderas de riesgo y eliminación de columnas redundantes (`monthly_fee`, que depende del plan, y los cargos por franja, que son una tarifa fija por minuto).
- Validaciones de claves, cantidades y coherencia de las variables derivadas con los datos originales.

## Archivos generados en `data/processed/`

| Archivo | Contenido |
|---|---|
| `subscription_features.parquet` / `.csv` | Dataset de suscripción estructurado (2.800 filas, 19 columnas) |
| `telco_features.parquet` / `.csv` | Dataset Telco estructurado (4.250 filas, 28 columnas) |
| `churn_indicators.csv` | Tabla con los indicadores de churn por grupo |

Parquet conserva los tipos (fechas, categorías); el CSV sirve para consulta rápida.

## Limitaciones

- No hay ventana de 30 días en los datos.
- El dataset de suscripción es sintético, y en él `tenure_months` no es coherente con `signup_date`.
- Los indicadores muestran asociación, no causalidad, y los umbrales de riesgo se eligieron observando los mismos datos.

## Historial de versiones

El proyecto sigue el flujo `main` ← `develop` ← ramas `feature/...`. Esta etapa se desarrolló en `feature/project-data-preparation`. El diagnóstico de calidad de la actividad anterior se conserva en `notebooks/data_quality.ipynb` y `reports/data_quality.md`.
