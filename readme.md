# 🧊 Trabajo Práctico Integrador – Data Engineering  
## Extracción FULL e INCREMENTAL + Capa Bronze & Silver (Delta Lake)

Proyecto de **Ingeniería de Datos** que implementa un flujo completo de **ingesta, transformación y almacenamiento** utilizando una arquitectura **Data Lakehouse** sobre **MinIO (S3)** y **Delta Lake**, consumiendo datos reales de la API **Open-Meteo**.

---

## 📡 Fuentes de Datos (API Open-Meteo)

Se consumen **dos endpoints**:

1. **Weather Forecast (`/v1/forecast`)**  
   Datos meteorológicos horarios.
2. **Air Quality (`/v1/air-quality`)**  
   Datos horarios de calidad del aire.

---

## 🥉 Parte 1 – Capa Bronze (Ingesta)

### 🔵 Extracción FULL
- Descarga completa del dataset diario.
- Modo de escritura: **overwrite**
- Sin particiones.
- Genera un snapshot limpio y completo.

**Rutas:**
- `bronze/full/weather/`
- `bronze/full/air_quality/`

---

### 🟢 Extracción INCREMENTAL
- Descarga solo los datos del día actual.
- Modo de escritura: **append**
- Particionado por columna `date`.
- Permite construir historiales diarios.

**Rutas:**
- `bronze/incremental/weather/`
- `bronze/incremental/air_quality/`

---

### 📂 Estructura generada en Bronze
```text
bronze/
├── full/
│   ├── weather/
│   └── air_quality/
└── incremental/
    ├── weather/date=YYYY-MM-DD/
    └── air_quality/date=YYYY-MM-DD/
```

## 🥈 Parte 2 – Capa Silver (Transformación y Agregación)

En esta etapa se procesan los datos horarios provenientes de la **capa Bronze** para generar datasets **limpios, estandarizados y enriquecidos** en la **capa Silver**, utilizando **pandas**, **Delta Lake** y **MinIO (S3)**.  
Se trabajan datos de **Weather** y **Air Quality**.

---

### 🔧 Lectura desde Bronze
- Carga de datasets horarios:
  - `weather_hourly`
  - `air_quality_hourly`
- Si la lectura falla, el pipeline continúa con DataFrames vacíos para evitar interrupciones.

---

### 🧹 Limpieza y Calidad de Datos
- Columnas numéricas: imputación de valores nulos con la **mediana**.
- Columnas categóricas: imputación con `'unknown'`.
- Eliminación de registros duplicados:
  - Weather: `time + station_id`
  - Air Quality: `time + location`
- No se eliminan filas completas innecesariamente.

---

### 🕒 Transformaciones
- Conversión de la columna `time` a tipo **datetime**.
- Creación de columnas auxiliares:
  - `extract_date`
  - `extract_hour`
- Optimización de columnas categóricas para mejorar rendimiento.
- Estandarización de nombres (Weather):
  - `temperature` → `temp_c`
  - `humidity` → `rel_humidity_pct`

---

### ➕ Enriquecimiento de Datos (Weather)
Se generan variables derivadas para análisis avanzado:
- `temp_above_30`
- `temp_f`
- `dew_point_c_est`
- `rolling_mean_temp_3h`
- `heat_index_c`
- `wind_kmh`
- `temp_category`
- `period_day`
- `weekday`

---

### 📊 Agregación Diaria
- **Weather**
  - Promedios, máximos y mínimos diarios
  - Velocidad de viento promedio
  - Horas con temperatura mayor a 30 °C
  - Horas con precipitación
- **Air Quality**
  - Promedios diarios de PM10, PM2.5, CO, O3, NO2 y SO2

---

### 🪙 Datasets generados en Silver
- Silver horario Weather
- Silver horario Air Quality
- Silver diario Weather
- Silver diario Air Quality

> Escritura en **Delta Lake** con modo **overwrite**, garantizando consistencia y versiones limpias.

---

### 🔐 Seguridad
- Credenciales gestionadas mediante variables de entorno o archivo `.env`.
- No se exponen claves ni información sensible en el código.

    ├── weather/date=YYYY-MM-DD/
    └── air_quality/date=YYYY-MM-DD/
