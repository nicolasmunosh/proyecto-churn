# Diagnóstico de calidad de datos: Churn en plataformas digitales

Evidencia **GA1-220501094-01-AA1-EV06** – Diagnosticar la calidad de datos: preparación, exploración y documentación.

**Aprendiz:** Nicolás Muñoz Hernández · **Ficha:** 3223873
**Programa:** Tecnólogo en Análisis y Desarrollo de Software · CTMA – SENA, Regional Antioquia

## Descripción del proyecto

Proyecto de ciencia de datos sobre el **análisis y predicción de la cancelación de suscripciones (churn)** en plataformas digitales. Este repositorio contiene el **diagnóstico inicial de calidad** de los dos datasets seleccionados, realizado con Python y Pandas.

## Propósito

Identificar la estructura, los tipos de variables, los valores faltantes y las posibles inconsistencias de los datasets, y documentar las acciones de limpieza y preparación que deberían aplicarse antes del análisis.

## Pregunta analítica

¿Qué patrones de interacción, frecuencia de uso y nivel de soporte técnico en los primeros 30 días permiten predecir el riesgo de cancelación?

## Datasets utilizados

| Dataset                       | Archivo                                                   | Filas | Columnas | Fuente                |
| ----------------------------- | --------------------------------------------------------- | ----- | -------- | --------------------- |
| Suscripción (SaaS, sintético) | `data/raw/customer_subscription_churn_usage_patterns.csv` | 2.800 | 10       | Google Dataset Search |
| Telco (telecomunicaciones)    | `data/raw/churn.csv`                                      | 4.250 | 21       | Kaggle / BigQuery     |

> **Nota:** el informe EV03 indicaba 10.000 registros para el dataset de suscripción. Al cargar el archivo real se comprobó que tiene 2.800 filas. Este repositorio usa las cifras reales.

## Estructura del repositorio

```
proyecto-churn/
├── README.md
├── .gitignore
├── requirements.txt
├── data/raw/
│   ├── customer_subscription_churn_usage_patterns.csv
│   └── churn.csv
├── notebooks/
│   └── diagnostico_calidad.ipynb
└── reports/
    └── informe_diagnostico.md
```

## Cómo reproducir el diagnóstico

Requisitos: Python 3.x, Git y Jupyter (o VS Code con la extensión de Jupyter).

```bash
git clone https://github.com/nicolasmunosh/proyecto-churn.git
cd proyecto-churn

python -m venv .venv
.venv\Scripts\activate          # Windows
# source .venv/bin/activate     # Linux / macOS

pip install -r requirements.txt
```

Luego abre `notebooks/diagnostico_calidad.ipynb`, selecciona el kernel del `.venv` y ejecuta todas las celdas (**Run All**). Las rutas de los datos son relativas (`../data/raw/`), por lo que el notebook debe ejecutarse desde la carpeta `notebooks/`.

## Resultados

El análisis completo está en [`reports/informe_diagnostico.md`](reports/informe_diagnostico.md).
