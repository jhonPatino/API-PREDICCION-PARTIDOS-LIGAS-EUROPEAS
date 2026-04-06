# Football Match Predictor — API REST con FastAPI

## Descripcion

Este proyecto despliega un modelo de clasificacion de machine learning como servicio web accesible via API REST. El modelo predice el resultado de partidos de futbol europeo (victoria local, empate o victoria visitante) a partir del historial estadistico de los equipos involucrados.

El sistema esta implementado en Python usando FastAPI para la capa de API, scikit-learn para el modelo de clasificacion, y ngrok para exponer el servicio publicamente desde Google Colab. Incluye logging persistente en archivo y monitoreo basico de metricas en memoria.

---

## Dataset

**Archivo:** `todas_las_ligas_2025.csv`

Contiene 1.266 partidos de la temporada 2025 distribuidos en cinco ligas europeas:

| Codigo | Liga |
|--------|------|
| PL | Premier League (Inglaterra) |
| PD | La Liga (Espana) |
| BL1 | Bundesliga (Alemania) |
| SA | Serie A (Italia) |
| FL1 | Ligue 1 (Francia) |

**Columnas disponibles:** `fecha`, `equipo_local`, `equipo_visitante`, `goles_local`, `goles_visitante`, `goles_totales`, `tiros_local`, `tiros_visitante`, `liga`, `temporada`.

Nota: las columnas `tiros_local` y `tiros_visitante` estan completamente vacias en el dataset y no se utilizan en el modelo.

---

## Modelo

**Algoritmo:** RandomForestClassifier (scikit-learn)

**Variable objetivo:** resultado del partido con tres clases posibles:
- `local` — gana el equipo de casa
- `empate` — ambos equipos marcan los mismos goles
- `visitante` — gana el equipo visitante

**Features:** Como el dataset no incluye estadisticas previas de los equipos, se construyen 13 features historicas acumuladas por equipo usando unicamente partidos anteriores al partido en cuestion, evitando data leakage:

| Feature | Descripcion |
|---------|-------------|
| `liga_enc` | Liga codificada numericamente |
| `local_jugados` | Partidos jugados por el equipo local hasta esa fecha |
| `local_win_rate` | Tasa de victorias historica del equipo local |
| `local_draw_rate` | Tasa de empates historica del equipo local |
| `local_avg_gf` | Promedio de goles a favor del equipo local |
| `local_avg_gc` | Promedio de goles en contra del equipo local |
| `visita_jugados` | Partidos jugados por el equipo visitante |
| `visita_win_rate` | Tasa de victorias del equipo visitante |
| `visita_draw_rate` | Tasa de empates del equipo visitante |
| `visita_avg_gf` | Promedio de goles a favor del equipo visitante |
| `visita_avg_gc` | Promedio de goles en contra del equipo visitante |
| `diff_win_rate` | Diferencia de tasa de victorias entre local y visitante |
| `diff_avg_gf` | Diferencia de promedio de goles a favor |

**Accuracy en test:** 44% (el baseline de azar para 3 clases es 33%).

Los partidos con menos de 3 encuentros previos registrados para alguno de los dos equipos se excluyen del entrenamiento, ya que las features historicas no son representativas con tan pocos datos.

---

## Estructura del proyecto

```
.
├── FastAPI_Ligas2025_REST.ipynb   # Notebook principal (Google Colab)
├── todas_las_ligas_2025.csv       # Dataset de partidos
├── README.md                      # Este archivo
```

Los siguientes archivos se generan al ejecutar el notebook:

```
├── api.py                         # Codigo de la API FastAPI (generado en Celda 4)
├── model.pkl                      # Modelo entrenado y metadatos (generado en Celda 3)
├── app.log                        # Log de eventos del servidor
├── dashboard_monitoreo.png        # Grafico de metricas (generado en Celda 9)
```

---

## Requisitos

- Cuenta de Google para usar Google Colab
- Cuenta gratuita en [ngrok.com](https://ngrok.com) para obtener el authtoken y exponer el servidor publicamente

Las dependencias de Python se instalan automaticamente en la Celda 1 del notebook:

```
fastapi
uvicorn
pyngrok
nest-asyncio
scikit-learn
```

---

## Instrucciones de uso

### 1. Abrir el notebook en Google Colab

Sube el archivo `FastAPI_Ligas2025_REST.ipynb` a Google Colab desde [colab.research.google.com](https://colab.research.google.com).

### 2. Subir el dataset

En el panel lateral de Colab (icono de carpeta), sube el archivo `todas_las_ligas_2025.csv` a la sesion. Tambien puedes hacerlo desde una celda:

```python
from google.colab import files
files.upload()  # selecciona todas_las_ligas_2025.csv
```

### 3. Instalar dependencias

Ejecuta la **Celda 1**. Instala todas las librerias necesarias con pip.

### 4. Preparar el dataset y construir features

Ejecuta la **Celda 2**. Esta celda:
- Carga y ordena el dataset por fecha
- Crea la variable objetivo (`resultado`)
- Construye las 13 features historicas por equipo sin data leakage
- Muestra graficos de distribucion de resultados por liga

### 5. Entrenar el modelo

Ejecuta la **Celda 3**. Esta celda:
- Divide los datos en entrenamiento (80%) y prueba (20%)
- Entrena el RandomForestClassifier
- Imprime el reporte de clasificacion y la matriz de confusion
- Guarda el modelo y sus metadatos en `model.pkl`

### 6. Generar la API

Ejecuta la **Celda 4**. Genera el archivo `api.py` con todos los endpoints y la logica de logging y monitoreo.

### 7. Configurar ngrok y lanzar el servidor

Antes de ejecutar la **Celda 5**, reemplaza el valor de `NGROK_AUTH_TOKEN` con tu token personal:

```python
NGROK_AUTH_TOKEN = "tu_token_aqui"
```

El token se obtiene en: [dashboard.ngrok.com/get-started/your-authtoken](https://dashboard.ngrok.com/get-started/your-authtoken)

Al ejecutar la celda aparecera en pantalla la URL publica del servidor, por ejemplo:

```
URL PUBLICA : https://abcd1234.ngrok-free.app
Swagger UI  : https://abcd1234.ngrok-free.app/docs
Health      : https://abcd1234.ngrok-free.app/health
```

### 8. Probar la API

Ejecuta las **Celdas 6 y 7** para verificar el funcionamiento con cURL y con Python requests. Tambien puedes abrir la URL de Swagger UI en el navegador para probar los endpoints de forma interactiva.

### 9. Monitoreo

Ejecuta las **Celdas 8 y 9** para consultar las metricas del servicio, revisar el archivo de log y generar el dashboard visual.

---

## Endpoints

| Metodo | Ruta | Descripcion |
|--------|------|-------------|
| GET | `/` | Informacion general de la API |
| GET | `/health` | Estado del servicio y del modelo |
| GET | `/metrics` | Metricas de uso: requests, predicciones, errores, latencias |
| GET | `/model-info` | Parametros e informacion del modelo desplegado |
| GET | `/equipos` | Lista de todos los equipos con historial disponible |
| GET | `/equipo/{nombre}` | Historial estadistico de un equipo especifico |
| POST | `/predict` | Predice el resultado de un partido |
| GET | `/docs` | Documentacion interactiva Swagger UI |

### Ejemplo de solicitud a /predict

```bash
curl -X POST https://<tu-url-ngrok>/predict \
  -H "Content-Type: application/json" \
  -d '{
    "equipo_local": "Liverpool FC",
    "equipo_visitante": "Arsenal FC",
    "liga": "PL"
  }'
```

### Ejemplo de respuesta

```json
{
  "equipo_local": "Liverpool FC",
  "equipo_visitante": "Arsenal FC",
  "liga": "PL",
  "prediccion": "local",
  "probabilidades": {
    "empate": 0.2341,
    "local": 0.5102,
    "visitante": 0.2557
  },
  "confianza": 0.5102,
  "local_historial": {
    "jugados": 38,
    "win_rate": 0.632,
    "avg_gf": 2.18
  },
  "visitante_historial": {
    "jugados": 38,
    "win_rate": 0.526,
    "avg_gf": 1.92
  },
  "model_accuracy": 0.44,
  "inference_time_ms": 12.4
}
```

---

## Logging

Cada solicitud al servidor queda registrada en el archivo `app.log` con el siguiente formato:

```
2025-04-05 14:32:10 | INFO     | -> POST /predict
2025-04-05 14:32:10 | INFO     | Prediccion: Liverpool FC vs Arsenal FC | PL
2025-04-05 14:32:10 | INFO     | Resultado: local | confianza=0.51 | 12.40 ms
2025-04-05 14:32:10 | INFO     | <- POST /predict status=200 | 13.21 ms
```

Los errores de validacion (HTTP 422) y los errores internos (HTTP 500) tambien quedan registrados con nivel `ERROR` o `CRITICAL`.

---

## Limitaciones

- El modelo no dispone de datos de tiros, posesion ni informacion de plantillas, lo que limita su capacidad predictiva.
- Los equipos que no aparecen en el historial del dataset reciben features con valor cero, lo que puede reducir la precision de la prediccion para esos casos.
- Las metricas de monitoreo se almacenan en memoria y se pierden al reiniciar el servidor.
- El servicio en Google Colab tiene una duracion maxima de sesion de aproximadamente 12 horas en cuentas gratuitas.
