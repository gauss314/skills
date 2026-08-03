---
name: mindicador
description: "API gratuita SIN key con los principales indicadores económicos de Chile: UF, UTM, dólar observado, euro, IPC, TPM, Imacec, cobre, desempleo y bitcoin."
license: MIT
metadata:
  category: finanzas, api, chile, indicadores, macroeconomia
  language: es
  source: https://mindicador.cl/
---

# Mindicador — Indicadores económicos de Chile (sin API key)

API **gratuita y sin registro** que agrega los principales indicadores económicos diarios de Chile desde fuentes oficiales (Banco Central de Chile y CMF). Es el punto de partida de fricción cero para trabajar datos chilenos: cero auth, JSON limpio, CORS abierto.

**Base URL:** `https://mindicador.cl/api`
**Documentación oficial:** [mindicador.cl](https://mindicador.cl/)

> Para series oficiales de origen y mayor profundidad histórica/catálogo, ver las skills hermanas [`bcch-data`](../bcch-data/) (Banco Central, 85.000+ series) y [`cmf-data`](../cmf-data/) (CMF).

---

## Autenticación

**No requiere.** Sin API key, sin registro, sin tarjeta.

---

## Endpoints

| # | Endpoint | Uso |
|---|----------|-----|
| 1 | `GET /api` | Todos los indicadores con su último valor del día |
| 2 | `GET /api/{indicador}` | Serie del año en curso de un indicador |
| 3 | `GET /api/{indicador}/{dd-mm-yyyy}` | Valor de un indicador en una fecha específica |
| 4 | `GET /api/{indicador}/{yyyy}` | Serie completa de un año calendario |

### Indicadores disponibles

| Código | Indicador | Unidad | Periodicidad |
|--------|-----------|--------|--------------|
| `uf` | Unidad de Fomento (UF) | CLP | Diaria |
| `ivp` | Índice de Valor Promedio | CLP | Diaria |
| `dolar` | Dólar observado | CLP | Diaria (hábil) |
| `dolar_intercambio` | Dólar acuerdo | CLP | Diaria |
| `euro` | Euro | CLP | Diaria (hábil) |
| `ipc` | IPC (variación mensual) | % | Mensual |
| `utm` | Unidad Tributaria Mensual | CLP | Mensual |
| `imacec` | Imacec (variación) | % | Mensual |
| `tpm` | Tasa de Política Monetaria | % | Diaria |
| `libra_cobre` | Libra de cobre | USD | Diaria (hábil) |
| `tasa_desempleo` | Tasa de desempleo | % | Mensual |
| `bitcoin` | Bitcoin | USD | Diaria |

---

## Ejemplos

### Todos los indicadores de hoy

```python
import requests

data = requests.get("https://mindicador.cl/api", timeout=10).json()
print(data["uf"]["valor"], data["dolar"]["valor"], data["tpm"]["valor"])
```

### Serie histórica de un año (dólar 2025)

```python
import pandas as pd
import requests

r = requests.get("https://mindicador.cl/api/dolar/2025", timeout=15).json()
df = pd.DataFrame(r["serie"])            # columnas: fecha, valor
df["fecha"] = pd.to_datetime(df["fecha"])
df = df.set_index("fecha").sort_index()
```

### UF de una fecha específica

```python
r = requests.get("https://mindicador.cl/api/uf/02-08-2026", timeout=10).json()
uf = r["serie"][0]["valor"] if r["serie"] else None
```

### Respuesta típica (`/api/{indicador}`)

```json
{
  "codigo": "uf",
  "nombre": "Unidad de fomento (UF)",
  "unidad_medida": "Pesos",
  "serie": [ {"fecha": "2026-08-02T04:00:00.000Z", "valor": 40844.79} ]
}
```

---

## Rate limits y buenas prácticas

- Sin límite documentado, pero es un servicio comunitario: **cachear** (la UF cambia una vez al día) e intercalar pausas en descargas masivas.
- Fechas en formato `dd-mm-yyyy` en la URL; las respuestas traen ISO-8601 UTC.
- Los días no hábiles no traen valor para indicadores cambiarios (`dolar`, `euro`): manejar series con huecos.
- Para uso productivo/auditable preferir la fuente oficial (`bcch-data` / `cmf-data`); mindicador es ideal para prototipos, dashboards y agentes.

---

## Casos de uso con agentes

- *"¿Cuánto está la UF hoy y cuánto ha variado el dólar en el mes?"*
- *"Descarga el dólar observado 2024-2025 y calcula la volatilidad mensual"*
- *"Convierte este contrato de 2.500 UF a pesos con la UF de la fecha de firma"*
