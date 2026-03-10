# 🗳️ Resultados Electorales Colombia 2026
**Autor: Nicolás Cardona — [@cardonanl](https://github.com/cardonanl)**

Notebooks de Python para descargar y analizar los resultados electorales de Colombia 2026 directamente desde los datos abiertos de la Registraduría Nacional del Estado Civil, a nivel de municipio.

> **English version below** ↓

---

## ¿Qué hace este proyecto?

Extrae de forma automatizada los votos por candidato, partido y municipio para las elecciones de **Senado**, **Cámara de Representantes**, **Consultas Interpartidistas** y **CITREP**, a partir del API público de la Registraduría.

## Notebooks incluidos

| Archivo | Descripción |
|---|---|
| `VotacionColombia2026_V4_Nacional.ipynb` | Descarga nacional con checkpoint automático por departamento |
| `VotacionColombia2026_V3.ipynb` | Descarga por territorio específico: departamento o municipio |
| `ExplorarNomenclator2026.ipynb` | Explorador del nomenclator: busca municipios, departamentos y circunscripciones por nombre o código |

## Requisitos

- Python 3.8+
- `requests`, `pandas`

Instalación rápida:
```bash
pip install requests pandas
```

Los notebooks están diseñados para correr en **Google Colab** sin ninguna configuración adicional.

## Uso paso a paso

### 1. Clona el repositorio
```bash
git clone https://github.com/cardonanl/elecciones-colombia-2026.git
```

### 2. (Opcional) Descarga el nomenclator localmente
Para evitar descargarlo cada vez, guarda una copia local:
```python
import requests, json
url = "https://resultados.registraduria.gov.co/json/nomenclator.json"
headers = {"User-Agent": "Mozilla/5.0", "Referer": "https://resultados.registraduria.gov.co/"}
data = requests.get(url, headers=headers).json()
with open("nomenclator_2026.json", "w", encoding="utf-8") as f:
    json.dump(data, f, ensure_ascii=False)
```

### 3. Configura la celda 1 del notebook principal

| Parámetro | Opciones | Descripción |
|---|---|---|
| `TIPO_ELECCION` | `'SE'`, `'CA'`, `'CN'`, `'CT'` | Tipo de elección |
| `SCOPE` | `'nacional'`, `'departamento'`, `'municipio'` | Alcance de la descarga |
| `CODIGO_TERRITORIO` | ej: `'3100'`, `'3100001'` | Código del territorio (ignorado si SCOPE es `'nacional'`) |
| `USAR_NOMENCLATOR_LOCAL` | `True` / `False` | Usar archivo local o descargar |

### 4. Ejecuta las celdas en orden

Para una **descarga nacional**, simplemente configura `SCOPE = 'nacional'` y ejecuta. Los resultados se guardan por departamento en la carpeta `resultados_SE_2026/`. Si la sesión se interrumpe, vuelve a ejecutar — los departamentos ya descargados se saltan automáticamente.

### 5. Consolida (solo para descarga nacional)

Ejecuta la **Celda 7** para unir todos los archivos parciales en un único CSV nacional.

---

## Estructura del JSON de la Registraduría

### Nomenclator (`/json/nomenclator.json`)

Archivo maestro con la jerarquía territorial. Estructura principal:

```json
{
  "ver": 1,
  "y": "2026",
  "elec": [ { "i": 0, "elec": 1, "sigla": "SE", "n": "SENADO" }, ... ],
  "amb": [
    {
      "elec": 1,
      "ambitos": [
        {
          "i": 176,
          "n": "CALI",
          "co": "3100001",
          "s": "CALI",
          "l": 3,
          "p": [{ "l": 2, "p": [25] }],
          "h": []
        }
      ]
    }
  ],
  "partidos": [ { "codpar": "92", "nombre": "PACTO HISTÓRICO", ... } ]
}
```

Campos clave de cada ámbito:

| Campo | Descripción |
|---|---|
| `i` | ID interno del nomenclator |
| `n` | Nombre del territorio |
| `co` | Código territorial (4 dígitos = departamento, 7 dígitos = municipio) |
| `l` | Nivel: 1=País, 2=Departamento, 3=Municipio |
| `p` | Referencias al padre en la jerarquía |
| `h` | IDs de los hijos directos |

> ⚠️ **Cambio 2026:** El campo de código pasó de `'c'` a `'co'`, y los códigos de departamento ya no son los códigos DANE de 2 dígitos sino códigos propios de 4 dígitos (ej: `3100` = Valle del Cauca).

### Resultados por municipio (`/json/ACT/{tipo}/{codigo}.json`)

```json
{
  "camaras": [
    {
      "partotabla": [
        {
          "act": {
            "codpar": "92",
            "cantotabla": [
              { "codcan": "001", "vot": 1523, "ncan": "NOMBRE CANDIDATO" }
            ]
          }
        }
      ]
    }
  ]
}
```

---

## Limitaciones conocidas

- **Nivel de granularidad:** El nomenclator 2026 solo llega hasta nivel de municipio (nivel 3). No hay datos de puesto de votación en el nomenclator público.
- **Tiempo de descarga:** Una descarga nacional completa puede tomar entre 1 y 3 horas dependiendo de la velocidad de la conexión y la carga del servidor.
- **Bloqueo 403:** El servidor de la Registraduría bloquea requests sin headers de navegador. Los notebooks ya incluyen los headers necesarios, pero el servidor puede imponer límites adicionales de tasa.
- **Datos en tiempo real:** Los resultados se actualizan progresivamente durante la noche electoral. Los datos descargados reflejan el estado del servidor en el momento de la consulta.
- **Códigos de partido:** El mapeo de `codpar` a nombre de partido puede estar incompleto. Los partidos no mapeados aparecen como `'Desconocido'`.
- **Estructura del JSON:** La Registraduría puede cambiar la estructura del API sin previo aviso. Si la descarga falla, usa la función `inspeccionar_json_municipio()` incluida en el notebook para diagnosticar.

---

## Contribuciones

Las contribuciones son bienvenidas. Algunas formas de ayudar:

- **Actualizar `MAPEO_PARTIDOS`** con nuevos códigos que aparezcan en los datos.
- **Reportar cambios en la estructura del API** abriendo un issue.
- **Agregar análisis** en notebooks separados (visualizaciones, comparaciones históricas, etc.).

Para contribuir: haz un fork, crea una rama con tu cambio y abre un Pull Request.

---
---

# 🗳️ Colombia 2026 Electoral Results
**Author: Nicolás Cardona — [@cardonanl](https://github.com/cardonanl)**

Python notebooks to download and analyze Colombia's 2026 electoral results directly from the open data published by the Registraduría Nacional del Estado Civil, at the municipality level.

---

## What does this project do?

It automatically extracts votes by candidate, party, and municipality for the **Senate**, **House of Representatives**, **Inter-party Consultations**, and **CITREP** elections, using the Registraduría's public API.

## Included notebooks

| File | Description |
|---|---|
| `VotacionColombia2026_V4_Nacional.ipynb` | National download with automatic checkpoint per department |
| `VotacionColombia2026_V3.ipynb` | Targeted download by specific department or municipality |
| `ExplorarNomenclator2026.ipynb` | Nomenclator explorer: search municipalities, departments, and constituencies by name or code |

## Requirements

- Python 3.8+
- `requests`, `pandas`

Quick install:
```bash
pip install requests pandas
```

Both notebooks are designed to run on **Google Colab** with no additional setup.

## Step-by-step usage

### 1. Clone the repository
```bash
git clone https://github.com/cardonanl/elecciones-colombia-2026.git
```

### 2. (Optional) Download the nomenclator locally
To avoid downloading it on every run, save a local copy:
```python
import requests, json
url = "https://resultados.registraduria.gov.co/json/nomenclator.json"
headers = {"User-Agent": "Mozilla/5.0", "Referer": "https://resultados.registraduria.gov.co/"}
data = requests.get(url, headers=headers).json()
with open("nomenclator_2026.json", "w", encoding="utf-8") as f:
    json.dump(data, f, ensure_ascii=False)
```

### 3. Configure cell 1 of the main notebook

| Parameter | Options | Description |
|---|---|---|
| `TIPO_ELECCION` | `'SE'`, `'CA'`, `'CN'`, `'CT'` | Election type |
| `SCOPE` | `'nacional'`, `'departamento'`, `'municipio'` | Download scope |
| `CODIGO_TERRITORIO` | e.g. `'3100'`, `'3100001'` | Territory code (ignored if SCOPE is `'nacional'`) |
| `USAR_NOMENCLATOR_LOCAL` | `True` / `False` | Use local file or download fresh |

### 4. Run cells in order

For a **national download**, set `SCOPE = 'nacional'` and run. Results are saved per department in the `resultados_SE_2026/` folder. If the session is interrupted, simply re-run — already downloaded departments are skipped automatically.

### 5. Consolidate (national download only)

Run **Cell 7** to merge all partial files into a single national CSV.

---

## Registraduría JSON structure

### Nomenclator (`/json/nomenclator.json`)

Master file with the territorial hierarchy. Main structure:

```json
{
  "ver": 1,
  "y": "2026",
  "elec": [ { "i": 0, "elec": 1, "sigla": "SE", "n": "SENADO" }, ... ],
  "amb": [
    {
      "elec": 1,
      "ambitos": [
        {
          "i": 176,
          "n": "CALI",
          "co": "3100001",
          "s": "CALI",
          "l": 3,
          "p": [{ "l": 2, "p": [25] }],
          "h": []
        }
      ]
    }
  ],
  "partidos": [ { "codpar": "92", "nombre": "PACTO HISTÓRICO", ... } ]
}
```

Key fields per territory entry:

| Field | Description |
|---|---|
| `i` | Internal nomenclator ID |
| `n` | Territory name |
| `co` | Territory code (4 digits = department, 7 digits = municipality) |
| `l` | Level: 1=Country, 2=Department, 3=Municipality |
| `p` | Parent references in the hierarchy |
| `h` | Direct children IDs |

> ⚠️ **2026 change:** The code field changed from `'c'` to `'co'`, and department codes are no longer the standard 2-digit DANE codes — they are now 4-digit internal codes (e.g. `3100` = Valle del Cauca).

### Results by municipality (`/json/ACT/{type}/{code}.json`)

```json
{
  "camaras": [
    {
      "partotabla": [
        {
          "act": {
            "codpar": "92",
            "cantotabla": [
              { "codcan": "001", "vot": 1523, "ncan": "CANDIDATE NAME" }
            ]
          }
        }
      ]
    }
  ]
}
```

---

## Known limitations

- **Granularity level:** The 2026 nomenclator only goes down to municipality level (level 3). Polling station data is not available in the public nomenclator.
- **Download time:** A full national download can take between 1 and 3 hours depending on connection speed and server load.
- **403 blocking:** The Registraduría's server blocks requests without browser-like headers. The notebooks include the necessary headers, but the server may impose additional rate limits.
- **Real-time data:** Results are updated progressively on election night. Downloaded data reflects the server state at the time of the request.
- **Party codes:** The `codpar` to party name mapping may be incomplete. Unmapped parties appear as `'Desconocido'`.
- **JSON structure:** The Registraduría may change the API structure without notice. If the download fails, use the `inspeccionar_json_municipio()` utility function included in the notebook to diagnose.

---

## Contributing

Contributions are welcome. Some ways to help:

- **Update `MAPEO_PARTIDOS`** with new party codes that appear in the data.
- **Report API structure changes** by opening an issue.
- **Add analysis notebooks** (visualizations, historical comparisons, etc.).

To contribute: fork the repo, create a branch with your change, and open a Pull Request.

---

*Datos obtenidos del portal público de la Registraduría Nacional del Estado Civil de Colombia — [resultados.registraduria.gov.co](https://resultados.registraduria.gov.co)*
