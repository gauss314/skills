---
name: cmf-data
description: "API oficial de la CMF de Chile (ex SBIF): UF, UTM, dólar, euro, IPC, TMC y TIP con histórico, más el perímetro de instituciones fiscalizadas. API key gratuita."
license: MIT
metadata:
  category: finanzas, api, chile, cmf, indicadores, regulador
  language: es
  source: https://api.cmfchile.cl/
---

# CMF Data — API oficial de la Comisión para el Mercado Financiero (Chile)

API **oficial y gratuita** del regulador financiero chileno (CMF, ex SBIF). Entrega los indicadores financieros de curso legal con **histórico profundo** (UF desde 1977) y valores **futuros ya publicados** (la UF se publica hacia adelante), además de tasas reguladas (TMC/TIP) que no están en los agregadores.

**Base URL:** `https://api.cmfchile.cl/api-sbifv3/recursos_api`
**Documentación oficial:** [api.cmfchile.cl](https://api.cmfchile.cl/)

---

## Autenticación

### Obtener API Key (GRATIS)

1. Ir a: https://api.cmfchile.cl/index.html → "Solicitar API Key"
2. Registrarse con email (se recibe la key al correo)
3. **No requiere tarjeta de crédito**

### Usar la API Key

Se pasa como query param en cada request, junto con `formato=json` (por defecto responde XML):

```python
import os
API_KEY = os.getenv("CMF_API_KEY")
params = {"apikey": API_KEY, "formato": "json"}
```

**⚠️ NUNCA hardcodear la API key en código compartido/commits.**

Sin key la API responde `{"CodigoHTTP": 422, "Mensaje": "API key no ha sido suministrada"}`.

---

## Endpoints principales

Patrón general por indicador (`uf`, `utm`, `dolar`, `euro`, `ipc`, `tmc`, `tip`):

| # | Endpoint | Uso |
|---|----------|-----|
| 1 | `GET /{indicador}` | Valor vigente (hoy / período actual) |
| 2 | `GET /{indicador}/{yyyy}` | Serie de un año completo |
| 3 | `GET /{indicador}/{yyyy}/{mm}` | Serie de un mes |
| 4 | `GET /{indicador}/{yyyy}/{mm}/dias/{dd}` | Valor de un día específico |
| 5 | `GET /{indicador}/posteriores/{yyyy}/{mm}` | Valores futuros ya publicados (útil en UF) |
| 6 | `GET /{indicador}/anteriores/{yyyy}/{mm}` | Valores anteriores a un período |
| 7 | `GET /{indicador}/periodo/{yyyy}/{mm}/{yyyy2}/{mm2}` | Rango entre dos períodos |

### Indicadores y claves de respuesta

| Recurso | Contenido | Clave JSON | Desde |
|---------|-----------|-----------|-------|
| `uf` | Unidad de Fomento diaria | `UFs` | 1977 |
| `utm` | Unidad Tributaria Mensual | `UTMs` | 1990 |
| `dolar` | Dólar observado | `Dolares` | 1984 |
| `euro` | Euro | `Euros` | 1999 |
| `ipc` | IPC variación mensual | `IPCs` | 1928 |
| `tmc` | Tasa Máxima Convencional (por tramo) | `TMCs` | 1993 |
| `tip` | Tasas de interés promedio captación | `TIPs` | — |

### Instituciones fiscalizadas

| Endpoint | Uso |
|----------|-----|
| `GET /instituciones` | Perímetro regulado: bancos y entidades vigentes con su código |

---

## Ejemplos

### UF de hoy

```python
import requests, os

BASE = "https://api.cmfchile.cl/api-sbifv3/recursos_api"
params = {"apikey": os.getenv("CMF_API_KEY"), "formato": "json"}

uf_hoy = requests.get(f"{BASE}/uf", params=params, timeout=10).json()["UFs"][0]
# {'Valor': '40.844,79', 'Fecha': '2026-08-02'}
```

### Serie anual a DataFrame (dólar 2025)

```python
import pandas as pd

data = requests.get(f"{BASE}/dolar/2025", params=params, timeout=15).json()["Dolares"]
df = pd.DataFrame(data)
# ⚠️ Los valores vienen como string con formato chileno: "952,31"
df["Valor"] = df["Valor"].str.replace(".", "", regex=False).str.replace(",", ".", regex=False).astype(float)
df["Fecha"] = pd.to_datetime(df["Fecha"])
```

### UF futura ya publicada (proyectar un contrato)

```python
futuras = requests.get(f"{BASE}/uf/posteriores/2026/08", params=params, timeout=10).json()["UFs"]
```

---

## Rate limits y buenas prácticas

- Sin límite público documentado; uso razonable con caché local (los indicadores cambian a lo más una vez al día).
- **Gotcha #1:** sin `formato=json` responde **XML**.
- **Gotcha #2:** los montos vienen como **string con separadores chilenos** (`"40.844,79"`) — convertir antes de calcular.
- **Gotcha #3:** `dolar`/`euro` solo tienen valor en días hábiles bancarios.
- La TMC es el insumo legal para validar tasas de crédito en Chile (Ley 18.010) — dato que ningún agregador global entrega.

---

## Casos de uso con agentes

- *"Valida que la tasa de este crédito de consumo no supere la TMC vigente del tramo"*
- *"Trae la UF de la fecha de cada factura y reajusta esta cartera a pesos de hoy"*
- *"Serie del dólar observado 2020-2025 en un DataFrame para calcular exposición cambiaria"*
