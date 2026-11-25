# Informe técnico: diagnóstico inicial de calidad de datos

**Evidencia:** GA1-220501094-01-AA1-EV06
**Aprendiz:** Nicolás Muñoz Hernández · Ficha 3223873 · CTMA – SENA, Antioquia

## 1. Nombre del proyecto y pregunta analítica

**Proyecto:** Análisis y Predicción de Cancelación de Suscripciones (Churn Rate) en Plataformas Digitales.

**Pregunta analítica:** ¿Qué patrones de interacción, frecuencia de uso y nivel de soporte técnico en los primeros 30 días permiten predecir el riesgo de cancelación?

## 2. Descripción de los datasets

**Dataset 1 – Suscripción (SaaS, sintético)** · `customer_subscription_churn_usage_patterns.csv`
Contiene el plan contratado, la tarifa mensual, las horas semanales de uso, los tickets de soporte, los fallos de pago, la antigüedad, los días desde el último acceso y la variable objetivo `churn` (Yes/No). Se relaciona con el problema porque permite explorar uso, soporte y pagos frente a la cancelación.

**Dataset 2 – Telco (telecomunicaciones)** · `churn.csv`
Contiene el tipo de plan, los minutos, llamadas y cargos por franja (diurna, vespertina, nocturna e internacional), las llamadas a servicio al cliente y la variable objetivo `churn` (True/False). Se relaciona con el problema porque permite analizar si el uso y las llamadas a soporte se asocian a la cancelación.

**Corrección respecto al informe EV03:** allí se indicó que el Dataset 1 tenía 10.000 registros y variables como edad, ubicación, CSAT y NPS. El archivo real tiene **2.800 registros y 10 columnas**, y esas variables no existen.

## 3. Resultados de la inspección estructural

| Dataset | Filas | Columnas | Tipos de datos | Observaciones |
|---|---|---|---|---|
| df_suscripcion | 2.800 | 10 | 6 int, 1 float, 3 texto (`signup_date`, `plan_type`, `churn`) | Sin nulos ni duplicados. Fechas guardadas como texto |
| df_telco | 4.250 | 21 | 7 int, 8 float, 3 bool, 3 texto (`customer_id`, `state`, `area_code`) | Sin nulos ni duplicados. Tres columnas booleanas, entre ellas `churn` |

Rangos observados en el Dataset 1: uso semanal 0,5–25 h, tickets 0–8, fallos de pago 0–5, antigüedad 1–36 meses, último acceso 0–60 días, fechas de registro entre 2023-01-01 y 2024-12-31.

## 4. Valores faltantes, tipos de datos e inconsistencias

**Valores faltantes:** ninguno de los dos datasets presenta valores nulos (0 en todas las columnas).

**Duplicados:** 0 filas duplicadas en ambos datasets. `user_id` y `customer_id` son únicos.

**Tipos de datos:**
- `signup_date` está como texto y debe convertirse a fecha (se convierte sin errores).
- `churn` es texto (Yes/No) en el Dataset 1 y booleano (True/False) en el Dataset 2.

**Posibles inconsistencias:**
- En el Dataset 1, `tenure_months` casi no tiene correlación con la antigüedad calculada desde `signup_date` (≈ -0,01 con fecha de referencia supuesta), lo que sugiere generación independiente de las variables.
- En el Dataset 2, hay 2 registros con 0 minutos y 0 llamadas diurnas pero con uso normal en las demás franjas.

## 5. Principales hallazgos de calidad por dataset

**Dataset 1 – Suscripción**
- Variable objetivo desbalanceada hacia la cancelación: 57,3 % "Yes" y 42,7 % "No", algo poco realista en un negocio real.
- `monthly_fee` depende exactamente de `plan_type` (Basic = 199, Standard = 399, Premium = 699), por lo que es redundante.
- Datos sintéticos con rangos cerrados y distribuciones muy regulares; pueden no reflejar el comportamiento de clientes reales.
- No contiene variables de los primeros 30 días; solo `tenure_months`.

**Dataset 2 – Telco**
- Variable objetivo desbalanceada: 14,1 % True y 85,9 % False.
- El cargo por minuto es prácticamente constante en cada franja (diurna ≈ 0,17; vespertina ≈ 0,085; nocturna ≈ 0,045; internacional ≈ 0,27), por lo que cada par minutos/cargo es redundante.
- `area_code` tiene solo 3 valores para 51 estados, con poco valor geográfico.
- `customer_id` es un identificador y no debe usarse como variable predictora.

**Entre ambos datasets**
- No comparten llave y sus dominios son distintos, por lo que no pueden unirse.
- Las etiquetas de churn tienen formatos diferentes (Yes/No frente a True/False).

## 6. Acciones de limpieza y preparación propuestas

| Dataset | Problema | Acción propuesta |
|---|---|---|
| Suscripción | Fechas como texto | Convertir con `pd.to_datetime` |
| Suscripción | `churn` como texto | Mapear Yes→1 y No→0 |
| Suscripción | Desbalance hacia "Yes" | Documentar y evaluar con métricas adecuadas |
| Suscripción | `tenure_months` vs `signup_date` | Verificar y priorizar una de las dos variables |
| Suscripción | `monthly_fee` redundante | Conservar solo una de las dos variables |
| Suscripción | Sin variables de 30 días | Ajustar el alcance de la pregunta analítica |
| Telco | Desbalance de clases | Muestreo estratificado y métricas como F1 o recall |
| Telco | Cargos redundantes con minutos | Elegir minutos o cargos por franja |
| Telco | 2 registros con 0 minutos diurnos | Revisar y decidir si se tratan como faltantes |
| Telco | `area_code` poco informativo | Tratar como categoría o descartar |
| Telco | `customer_id` | Excluir del modelado |
| Ambos | Etiquetas de churn distintas | Normalizar a 0/1 |

## 7. Conclusiones

Ambos datasets se cargan correctamente, no tienen valores nulos ni duplicados y sus rangos numéricos son plausibles, por lo que son **aptos para continuar con la fase de preparación**. Los problemas encontrados son de tipo de dato, redundancia entre variables, desbalance de clases y limitaciones de origen, ya que el Dataset 1 es sintético.

La pregunta analítica, centrada en los primeros 30 días, **no puede responderse con exactitud** con estos datos, porque ninguno trae información por ventana de 30 días. Se recomienda ajustar el alcance de la pregunta a las variables disponibles (uso, tickets de soporte, fallos de pago y llamadas a servicio al cliente).

Por último, se detectó una discrepancia entre la documentación previa (EV03) y los datos reales, lo que confirma la importancia de validar los archivos antes de modelar.
