# 🛵 Rappi Plus – Análisis de Rentabilidad, Embudo de Conversión y Test A/B

Proyecto de analítica de datos end-to-end sobre el programa de suscripción **Rappi Plus / Rappi Pro**, que combina limpieza y modelado de datos en Python, análisis exploratorio y estadístico en Jupyter Notebook, consultas SQL sobre eventos de usuario, y un **Dashboard interactivo en Power BI** con **modelado relacional en esquema estrella**.

---

## 📌 Objetivo del proyecto

Evaluar la salud del negocio de Rappi Plus a partir de tres frentes analíticos:

1. **Rentabilidad del negocio**: ingresos, costos, descuentos y gasto en marketing por país, categoría de producto y canal de adquisición.
2. **Embudo de conversión (Funnel Analysis)**: identificar en qué etapa del recorrido del usuario (`first_visit` → `select_item` → `add_to_cart` → `begin_checkout` → `add_payment_info` → `purchase`) se concentran las mayores pérdidas.
3. **Retención por cohortes**: medir qué tan bien la plataforma retiene usuarios en ventanas de 7, 14 y 21 días desde su registro.
4. **Test A/B**: determinar si un cambio en la UI del checkout impacta de forma estadísticamente significativa la tasa de conversión y la duración de la sesión.

---

## 🗂️ Fuentes de datos

| Dataset | Descripción | Registros |
|---|---|---|
| `rappiplus_orders_raw.csv` | Pedidos: usuario, país, dispositivo, fuente de referencia, producto, cantidad, precios, descuentos y monto total | 25,100 |
| `rappiplus_catalog.csv` | Catálogo de productos: costo unitario y proveedor por producto | 7 |
| `rappiplus_marketing_spend.csv` | Gasto de marketing por fecha, país, campaña y canal | 1,620 |
| Base de datos SQL (PostgreSQL) | Eventos de usuario (`first_visit`, `select_item`, `add_to_cart`, `begin_checkout`, `add_payment_info`, `purchase`) para el análisis de funnel, cohortes y test A/B | — |

---

## 🔎 Contenido del notebook (`Rappi_Plus_Analysis.ipynb`)

### 1. Preparación de datos
- Importación y exploración inicial de `orders`, `catalog` y `marketing` con `pandas`.
- Tratamiento documentado de valores nulos en columnas numéricas (`cantidad`, `precio_unitario`, `monto_descuento`): se decidió **no eliminar ni imputar** estos registros para preservar transacciones válidas y aprovechar que DAX en Power BI ignora automáticamente los `BLANK` en agregaciones.
- Exportación de datasets limpios (`orders_clean.csv`, `catalog_clean.csv`, `marketing_clean.csv`) listos para el modelo en Power BI.

### 2. Análisis de rentabilidad
- Cálculo de ingresos, costos y margen a partir del cruce `orders` × `catalog`.
- Comparación del gasto en marketing contra los ingresos generados por país/canal.

### 3. Funnel de conversión (SQL)
- Consultas SQL contra la base de eventos para obtener usuarios únicos por etapa.
- **Hallazgo principal:** el mayor cuello de botella está entre `begin_checkout` y `add_payment_info`, con una caída de **13.29%** (1,042 usuarios), asociada a fricción en la pasarela de pago.
- Una vez capturado el método de pago, la conversión a compra es casi perfecta (**99.84%**).
- Validación cruzada de usuarios que llegan a `add_to_cart` sin pasar por `select_item` (compras directas desde listados/carruseles) y viceversa.

### 4. Retención por cohortes
- Cálculo de tasas de retención periódica en ventanas de 7, 14 y 21 días por cohorte mensual.
- Identificación de repuntes en la Semana 2 en cohortes como 2025-03 y 2025-04, interpretados como reactivación de usuarios inactivos en su primera semana.

### 5. Test estadístico A/B (cambio de UI en checkout)
- **Hipótesis principal (conversión):** Z-test de proporciones → Z = 0.8133, p-value = 0.4161 → **no se rechaza H₀** (sin diferencia significativa entre control 15.69% y tratamiento 16.29%).
- **Hipótesis secundaria (duración de sesión):** t-Student → t = 1.4454, p-value = 0.1484 → **no se rechaza H₀** (161.04 s control vs 158.70 s tratamiento, estadísticamente equivalentes).
- **Conclusión:** el cambio de UI evaluado no generó un impacto estadísticamente significativo en conversión ni en duración de sesión, dentro de un nivel de significancia α = 0.05.

---

## 📊 Dashboard en Power BI

A partir de los datasets limpios exportados desde el notebook, se construyó un **modelo de datos relacional en esquema estrella (star schema)** en Power BI:

- **Tabla de hechos:** pedidos/transacciones (`orders_clean`), con métricas de cantidad, precio, descuento y monto total.
- **Tablas de dimensión:** producto/catálogo (`catalog_clean`), marketing/campañas (`marketing_clean`), y dimensiones descriptivas derivadas (país, dispositivo, fuente de referencia, fecha).
- Medidas DAX para ingresos, margen, gasto en marketing, tasas de conversión del funnel y retención por cohorte.
- Visualizaciones interactivas con segmentadores por país, canal y periodo, que permiten explorar los mismos hallazgos documentados en el notebook de forma dinámica.

> 📁 *Agrega aquí el enlace o archivo `.pbix` del dashboard, y una captura de pantalla si deseas mostrarlo visualmente en este README.*

---

## 🛠️ Tecnologías utilizadas

- **Python:** `pandas`, `numpy`, `scipy.stats`, `statsmodels` (`proportions_ztest`), `sqlalchemy`
- **SQL:** PostgreSQL (consultas de eventos de usuario para funnel, cohortes y test A/B)
- **Power BI:** modelado relacional en esquema estrella, DAX, dashboard interactivo

---

## 📁 Estructura del repositorio

```
├── Rappi_Plus_Analysis.ipynb   # Notebook con limpieza, EDA, SQL y análisis estadístico
├── orders_clean.csv            # Dataset de pedidos limpio (exportado del notebook)
├── catalog_clean.csv           # Dataset de catálogo limpio (exportado del notebook)
├── marketing_clean.csv         # Dataset de marketing limpio (exportado del notebook)
├── dashboard/                  # Archivo .pbix del dashboard de Power BI
└── README.md
```

---

## 🚀 Cómo reproducir el análisis

```bash
# Clonar el repositorio
git clone <URL_DEL_REPOSITORIO>
cd <NOMBRE_DEL_REPO>

# Instalar dependencias
pip install pandas numpy scipy statsmodels sqlalchemy psycopg2-binary jupyter

# Ejecutar el notebook
jupyter notebook Rappi_Plus_Analysis.ipynb
```

> ⚠️ El notebook incluye una conexión a una base de datos PostgreSQL provista para fines educativos (credenciales de solo lectura). Para reproducir la sección de funnel/cohortes/A-B testing se requiere acceso a esa base o una fuente de eventos equivalente.

---

## 👤 Autor

**Jair García Zamorano**
Biólogo y limnólogo (M.Sc., UNAM/ICMyL) en transición hacia analítica de datos.
Portafolio: [github.com/jairgyz97-lab](https://github.com/jairgyz97-lab)
