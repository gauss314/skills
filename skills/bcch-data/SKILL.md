---
name: bcch-data
description: "API oficial del Banco Central de Chile (Base de Datos Estadísticos): 85.000+ series macro y financieras — tipo de cambio, TPM, IPC, Imacec, cuentas nacionales, agregados monetarios. Credenciales gratuitas."
license: MIT
metadata:
  category: finanzas, api, chile, banco-central, macroeconomia
  language: es
  source: https://si3.bcentral.cl/Siete/es/Siete/API
---

# BCCh Data — Base de Datos Estadísticos del Banco Central de Chile

API **oficial y gratuita** del Banco Central de Chile sobre su Base de Datos Estadísticos (BDE): **85.000+ series temporales** de tipo de cambio, tasas, inflación, actividad (Imacec), cuentas nacionales, agregados monetarios, balanza de pagos y mercado laboral. Es la fuente de origen de los indicadores chilenos — el equivalente chileno de FRED.

**Base URL:** `https://si3.bcentral.cl/SieteRestWS/SieteRestWS.ashx`
**Documentación oficial:** [si3.bcentral.cl/Siete/es/Siete/API](https://si3.bcentral.cl/Siete/es/Siete/API)
**Catálogo navegable de series:** [si3.bcentral.cl/Siete](https://si3.bcentral.cl/Siete/es/Siete/Canasta)

---

## Autenticación

### Obtener credenciales (GRATIS)

1. Registrarse en https://si3.bcentral.cl/Siete/es/Siete/API (crear cuenta de usuario BDE)
2. Las credenciales son **usuario y contraseña** (no un token) y se pasan como query params
3. **No requiere tarjeta de crédito**

```python
import os
USER = os.getenv("BCCH_USER")
PASS = os.getenv("BCCH_PASS")
```

**⚠️ NUNCA hardcodear credenciales en código compartido/commits.** Sin credenciales válidas responde `{"Codigo": -5, "Descripcion": "Invalid username or password"}`.

---

## Funciones de la API

Todas via `GET` sobre la base URL, con `user`, `pass` y `function`:

| # | `function` | Uso |
|---|-----------|-----|
| 1 | `GetSeries` | Observaciones de una serie entre fechas (`timeseries`, `firstdate`, `lastdate`) |
| 2 | `SearchSeries` | Buscar series del catálogo por frecuencia (`frequency=DAILY/MONTHLY/QUARTERLY/ANNUAL`) |

### Parámetros de `GetSeries`

| Parámetro | Formato | Ejemplo |
|-----------|---------|---------|
| `timeseries` | Código de serie BDE | `F073.TCO.PRE.Z.D` |
| `firstdate` | `yyyy-mm-dd` | `2025-01-01` |
| `lastdate` | `yyyy-mm-dd` | `2025-12-31` |

### Series de uso frecuente

> Verificar el código exacto en el [catálogo BDE](https://si3.bcentral.cl/Siete/es/Siete/Canasta) — el catálogo es la fuente de verdad y permite copiar el código de cada serie.

| Código | Serie | Frecuencia |
|--------|-------|------------|
| `F073.TCO.PRE.Z.D` | Dólar observado | Diaria |
| `F073.UFF.PRE.Z.D` | Unidad de Fomento (UF) | Diaria |
| `F022.TPM.TIN.D001.NO.Z.D` | Tasa de Política Monetaria (TPM) | Diaria |
| `F074.IPC.VAR.Z.Z.C.M` | IPC, variación mensual | Mensual |
| `F032.IMC.IND.Z.Z.EP18.Z.Z.0.M` | Imacec | Mensual |

---

## Ejemplos

### Dólar observado del último mes

```python
import requests, os

BASE = "https://si3.bcentral.cl/SieteRestWS/SieteRestWS.ashx"
params = {
    "user": os.getenv("BCCH_USER"),
    "pass": os.getenv("BCCH_PASS"),
    "function": "GetSeries",
    "timeseries": "F073.TCO.PRE.Z.D",
    "firstdate": "2026-07-01",
    "lastdate": "2026-07-31",
}
data = requests.get(BASE, params=params, timeout=15).json()
obs = data["Series"]["Obs"]   # [{'indexDateString': '01-07-2026', 'value': '952.31', 'statusCode': 'OK'}, ...]
```

### A DataFrame

```python
import pandas as pd

df = pd.DataFrame(obs)
df["fecha"] = pd.to_datetime(df["indexDateString"], format="%d-%m-%Y")
df["valor"] = pd.to_numeric(df["value"], errors="coerce")   # días sin dato vienen como 'NaN'
df = df.dropna(subset=["valor"]).set_index("fecha")[["valor"]]
```

### Buscar series del catálogo por frecuencia

```python
params = {"user": USER, "pass": PASS, "function": "SearchSeries", "frequency": "DAILY"}
catalogo = requests.get(BASE, params=params, timeout=30).json()["SeriesInfos"]
# cada item trae seriesId, título en español/inglés, frecuencia, fechas de cobertura
```

### Alternativa con paquete Python

Existe el paquete [`bcchapi`](https://pypi.org/project/bcchapi/) (del propio BCCh) que envuelve esta API con interfaz pandas: `pip install bcchapi`.

---

## Rate limits y buenas prácticas

- Sin límite público documentado; el servicio es compartido — cachear localmente e intercalar pausas en descargas masivas.
- **Gotcha #1:** las respuestas de `GetSeries` traen `value` como **string** y usan `'NaN'` para días sin observación (feriados/fines de semana en series diarias).
- **Gotcha #2:** las fechas de observación vienen `dd-mm-yyyy` (`indexDateString`), pero los parámetros de consulta van `yyyy-mm-dd`.
- **Gotcha #3:** el `statusCode` por observación (`OK`/otros) permite filtrar datos provisorios.
- Los códigos de serie son estables pero el catálogo evoluciona: ante duda, confirmar en la BDE web antes de automatizar.

---

## Casos de uso con agentes

- *"Descarga la TPM y el IPC de los últimos 5 años y grafica la tasa real"*
- *"Trae el dólar observado 2020-2026 y calcula la depreciación anualizada del peso"*
- *"Compara el Imacec contra la variación del IPC para el informe del directorio"*
