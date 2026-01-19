# Credit-Collection-Risk-Modeling
Este repositorio contiene el desarrollo completo de un modelo de riesgo de cobranza utilizando técnicas de Machine Learning supervisado, con un enfoque temporal y por tramos de mora, siguiendo buenas prácticas del sector financiero. 


El objetivo es **predecir la probabilidad de deterioro del cliente (malos)** en los meses siguientes, a partir de información sociodemográfica, económica, de producto y comportamiento histórico.

---

## Definición del problema

**Variable objetivo (Y_malo):**

- **Buenos (0):**
  - Mantienen o mejoran su tramo de mora
  - No presentan castigo ni reestructuración en el mes siguiente

- **Malos (1):**
  - Incrementan su tramo de mora
  - Permanecen con mora > 120 días
  - Son reestructurados o castigados en el mes siguiente

El modelo predice:

> **P(Y_malo = 1 | información histórica hasta el mes t)**

---

## Fuentes de información

Se integran múltiples tablas operativas:

- Información sociodemográfica
- Información económica
- Información de producto
- Comportamiento mensual de cartera
- Pagos
- Gestiones de cobranza
- Reestructuraciones
- Castigos

La llave de integración es:
folade y negocio


Cada fila representa un **cliente-producto-mes**.

---

## Enfoque temporal (muy importante)

- El dataset tiene múltiples observaciones por cliente.
- Se respeta el orden cronológico:
  - Variables explicativas en `t`
  - Variable objetivo definida en `t+1`
- El split para entrenar y probar el modelo **NO es aleatorio**, se hace por **meses completos**:
  - 70% historia → entrenamiento
  - 30% futuro → test (out-of-time)

Esto evita data leakage y replica un entorno productivo real.

---

## Segmentación por tramo de mora

Se entrena **un modelo independiente por tramo**, debido a que el comportamiento del cliente cambia radicalmente según su nivel de mora:

| Tramo | Días de mora |
|------|--------------|
| T1_0_30 | 0–30 |
| T2_31_60 | 31–60 |
| T3_61_120 | 61–120 |
| T4_120+ | >120 |

Esto mejora interpretabilidad y estabilidad del score.

---

## Feature Engineering (comportamiento temporal)

Se crean variables de comportamiento dinámico:

- Lags de mora (`mora_t_1`, `mora_t_2`)
- Cambios de mora (`delta_mora`)
- Meses consecutivos en mora
- Máximo y promedio de mora en ventanas móviles (3 meses)
- Intensidad de pagos
- Actividad de gestión y contactabilidad

Estas variables permiten capturar **tendencia y aceleración del deterioro**, no solo el nivel actual.

---

## Modelo

Se utiliza **LightGBM** por su:

- Robustez con variables heterogéneas
- Buen desempeño en datasets grandes
- Capacidad para capturar no linealidades

Configuración clave:

- `class_weight='balanced'` para manejar desbalance
- Validación temporal
- Evaluación por AUC y KS

---

## Métricas de evaluación

Para cada tramo se reporta:

- **AUC**: capacidad de ranking del modelo
- **KS**: máxima separación entre buenos y malos
- **Matriz de confusión**
- **Reporte de clasificación**

### Resultados resumidos:

| Tramo | AUC | KS |
|-----|-----|----|
| T1_0_30 | 0.906 | 0.676 |
| T2_31_60 | 0.675 | 0.235 |
| T3_61_120 | 0.689 | 0.284 |
| T4_120+ | 0.723 | 0.333 |

Tramos intermedios presentan menor AUC debido a mayor incertidumbre natural del comportamiento del cliente.

---

## Threshold y KS

- El **AUC no depende del threshold**
- El **threshold óptimo** se obtiene maximizando KS
- Se utiliza únicamente cuando se requiere **decisión binaria**
- Para análisis y priorización, se trabaja con **scores continuos**

---

## Conclusión

Este proyecto reproduce un **pipeline real de modelación de riesgo de cobranza**, considerando:

- Panel de datos
- Dependencia temporal
- Segmentación por riesgo
- Métricas estándar del sector financiero

Es un enfoque **productivo, interpretable y escalable**.

---

## Contacto

Andrea Rodríguez  
Data Scientist | Risk & Credit Analytics
