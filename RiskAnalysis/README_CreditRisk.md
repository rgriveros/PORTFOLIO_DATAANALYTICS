# 💳 Credit Risk Analytics — Detección de Morosidad y Alertas Tempranas

![Python](https://img.shields.io/badge/Python-3.10+-blue?logo=python&logoColor=white)
![SQL](https://img.shields.io/badge/SQL-SQLite-lightblue?logo=sqlite&logoColor=white)
![Pandas](https://img.shields.io/badge/Pandas-2.0+-darkblue?logo=pandas&logoColor=white)
![Seaborn](https://img.shields.io/badge/Seaborn-Visualization-teal)
![ISO31000](https://img.shields.io/badge/Marco-ISO%2031000-orange)
![Status](https://img.shields.io/badge/Status-Completado-brightgreen)

---

## 📋 Descripción del Proyecto

Simulación de una **auditoría de cartera crediticia** para una financiera de consumo con 150.000 clientes. El Gerente de Riesgo solicitó un análisis exploratorio para identificar patrones de morosidad, segmentar clientes por nivel de riesgo y proponer criterios de alerta temprana basados en datos reales.

El análisis combina técnicas estadísticas (segmentación por DebtRatio, análisis etario) con criterios de gestión de riesgos del marco **ISO 31000**, traduciendo hallazgos técnicos en decisiones de negocio accionables.

---

## 🎯 Objetivos

- Calcular y contextualizar la **tasa de morosidad global** de la cartera
- Segmentar clientes por nivel de riesgo usando el **ratio deuda/ingreso**
- Identificar los **grupos etarios** con mayor probabilidad de incumplimiento
- Detectar las **combinaciones críticas** de edad + segmento mediante heatmap
- Proponer un **criterio de aceptación/rechazo** alineado a ISO 31000

---

## 📦 Dataset

**Give Me Some Credit — Kaggle**
- Fuente: [Kaggle — brycecf](https://www.kaggle.com/datasets/brycecf/give-me-some-credit-dataset)
- Registros: 150.000 clientes
- Variables clave: `SeriousDlqin2yrs`, `DebtRatio`, `MonthlyIncome`, `age`, `NumberOfDependents`

| Variable | Descripción |
|---|---|
| `SeriousDlqin2yrs` | Variable objetivo: morosidad grave en los próximos 2 años (1=Sí, 0=No) |
| `DebtRatio` | Ratio deuda/ingreso mensual |
| `MonthlyIncome` | Ingreso mensual del cliente |
| `age` | Edad del cliente |
| `RevolvingUtilizationOfUnsecuredLines` | Uso de líneas de crédito rotativas |
| `NumberOfDependents` | Cantidad de dependientes económicos |

---

## 🛠️ Tecnologías utilizadas

| Herramienta | Uso |
|---|---|
| **Python 3.10+** | Análisis, limpieza y visualización |
| **SQLite (`:memory:`)** | Base de datos relacional en RAM |
| **SQL** | Queries de agregación y análisis segmentado |
| **Pandas** | Transformación, imputación y agrupación |
| **Seaborn / Matplotlib** | Visualizaciones estadísticas y heatmap |
| **NumPy** | Operaciones numéricas |
| **ISO 31000** | Marco de referencia para criterios de riesgo |

---

## 🔍 Metodología

### 1. Limpieza e Imputación con criterio de negocio

Los nulos en `MonthlyIncome` (29.731 registros) se imputaron con la **mediana segmentada por grupo etario** — no con la mediana global — porque el ingreso varía naturalmente a lo largo del ciclo de vida del cliente. Usar la mediana global distorsionaría el perfil financiero de cada segmento.

```python
df['age_group'] = pd.cut(df['age'], bins=[0,30,45,60,120],
                          labels=['<30','30-45','45-60','>60'])

df['MonthlyIncome'] = df.groupby('age_group', observed=False)['MonthlyIncome'].transform(
    lambda x: x.fillna(x.median())
)
df['NumberOfDependents'] = df['NumberOfDependents'].fillna(0)
```

### 2. Segmentación de Riesgo por DebtRatio

| Segmento | DebtRatio | Tasa de Morosidad |
|---|---|---|
| Bajo | < 0.3 | 5.87% |
| Medio | 0.3 – 0.6 | 6.89% |
| Alto | 0.6 – 1.0 | 10.65% |
| Crítico | > 1.0 | 6.49% |

> **Hallazgo metodológico:** El segmento "Crítico" no tiene la mayor tasa de morosidad. Esto sugiere que clientes con DebtRatio extremadamente alto pueden corresponder a perfiles de alto patrimonio con capacidad de pago real, no necesariamente clientes en riesgo.

### 3. Análisis SQL por tramos de edad

```sql
SELECT
    age_group,
    COUNT(*) AS total_clientes,
    ROUND(AVG(SeriousDlqin2yrs) * 100, 2) AS tasa_morosidad_pct,
    ROUND(AVG(MonthlyIncome), 2) AS ingreso_promedio,
    ROUND(AVG(DebtRatio), 2) AS ratio_deuda_ingreso
FROM creditos
GROUP BY age_group
ORDER BY tasa_morosidad_pct DESC
```

### 4. Heatmap de Alerta Temprana

Cruce de `age_group` × `segmento_riesgo` para identificar combinaciones de mayor riesgo conjunto — el indicador más accionable para el equipo de Riesgo.

---

## 📊 Hallazgos Principales

| Hallazgo | Detalle |
|---|---|
| **Tasa de morosidad global** | 6.68% — línea base de referencia |
| **Grupo etario de mayor riesgo** | < 30 años → 11.56% de morosidad |
| **Combinación crítica #1** | 30-45 años + Riesgo Alto → **12.55%** |
| **Combinación crítica #2** | < 30 años + Riesgo Crítico → **12.33%** |
| **Mejor predictor simple** | DebtRatio segmento "Alto" duplica la tasa base |

---

## 💡 Criterio de Aceptación/Rechazo — Marco ISO 31000

| Zona | Criterio | Acción recomendada |
|---|---|---|
| ✅ **Aceptación automática** | DebtRatio < 0.3 AND age_group ≠ '<30' | Aprobar crédito |
| ⚠️ **Revisión manual** | DebtRatio 0.3–1.0 OR age_group '<30' | Analista revisa historial completo |
| 🔴 **Rechazo / garantía** | DebtRatio > 1.0 AND age_group '<30' o '30-45' | Rechazar o exigir garantía adicional |

**Principio aplicado:** El umbral de intervención es la tasa de morosidad global (6.68%). Todo segmento que **duplique esa tasa** (> 13%) requiere intervención activa antes del otorgamiento.

---

## 📁 Estructura del Repositorio

```
RiskAnalysis/
│
├── GiveMeSomeCredit.ipynb     # Notebook principal con análisis completo
├── README.md                  # Este archivo
└── assets/                    # Capturas de visualizaciones (opcional)
```

---

## ▶️ Cómo reproducir el análisis

1. Clonar el repositorio:
```bash
git clone https://github.com/RGRIVEROS-PORTFOLIO/PORTFOLIO_ANALYTICS.git
cd RiskAnalysis
```

2. Instalar dependencias:
```bash
pip install pandas numpy matplotlib seaborn kagglehub
```

3. Ejecutar el notebook completo con `Restart & Run All`.
El dataset se descarga automáticamente via API de Kaggle en la primera celda.

---

## 👤 Autor

**Rodolfo Gabriel Riveros Lobos**
Data Analyst Junior | Background en Calidad ISO 9001, ISO 31000 & Customer Experience

[![LinkedIn](https://img.shields.io/badge/LinkedIn-Conectar-blue?logo=linkedin)](https://www.linkedin.com/in/tu-perfil)
[![GitHub](https://img.shields.io/badge/GitHub-Portfolio-black?logo=github)](https://github.com/RGRIVEROS-PORTFOLIO/PORTFOLIO_ANALYTICS)

---

## 📂 Otros proyectos del portfolio

| Proyecto | Descripción | Stack |
|---|---|---|
| [Restaurant Reviews](../restaurant-reviews-analysis) | NLP básico y análisis de satisfacción | Python, WordCloud |
| [Superstore Retail](../retail-sales-analysis) | Ventas, RFM, Pareto, Executive Summary | Python, Pandas |
| [Olist E-Commerce](../olist-ecommerce-analysis) | SQL + Python + mapa coroplético | SQL, GeoPandas |
| [HR Nómina Analytics](../hr-nomina-analytics) | Auditoría salarial, Z-Score, Window Functions | SQL, NumPy |
| **Credit Risk Analytics** | Morosidad, segmentación de riesgo, ISO 31000 | SQL, Python, Seaborn |

---

*Proyecto desarrollado como parte del proceso de formación en Data Analytics aplicado a escenarios reales del sector financiero.*
