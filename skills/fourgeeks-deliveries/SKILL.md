---
name: fourgeeks-deliveries
description: >-
  Consulta las tareas academicas ya entregadas en 4Geeks mediante la API oficial
  BreatheCode y muestra sus evidencias de entrega (repositorio GitHub y enlace
  de despliegue). No inventa enlaces, fechas ni calificaciones.
allowed-tools:
  - exec
---

# FourGeeks Deliveries

## Proposito

Listar las tareas academicas que el estudiante ya entrego, mostrando sus
evidencias reales de entrega: repositorio de GitHub y URL de despliegue,
cuando la API las proporcione.

## Disparadores

- "?Que tareas he entregado en 4Geeks?"
- "Muestrame mis entregas y sus repositorios"
- "?Donde estan los enlaces de mis entregas?"

## Endpoint

```
GET https://breathecode.herokuapp.com/v1/assignment/user/me/task?task_status=DONE
```

**Fuentes oficiales (`breatheco-de/apiv2`):**
- Ruta: `assignments/urls.py` linea 84 — `path("user/me/task", TaskMeView...)`,
  prefijo raiz `v1/assignment/`
- Filtros: `assignments/views.py`, `TaskMeView.get()` acepta `task_status`
  y `revision_status` como query params separados por coma
- Serializador del listado: `TaskUserSmallSerializer` en `assignments/serializers.py`
- Modelo y choices: `assignments/models.py` lineas 75-121

## Campos expuestos por la API

| Campo             | Uso en esta skill                                  |
|-------------------|----------------------------------------------------|
| `id`              | Identificador unico de la tarea                    |
| `title`           | Titulo de la tarea                                 |
| `task_type`       | `PROJECT`, `QUIZ`, `LESSON`, `EXERCISE`            |
| `task_status`     | `PENDING` o `DONE` (filtrado a `DONE`)             |
| `revision_status` | `PENDING`, `APPROVED`, `REJECTED`, `IGNORED`       |
| `delivered_at`    | Fecha de entrega; puede ser `null`                 |
| `github_url`      | URL del repositorio; puede ser `null`              |
| `live_url`        | URL de despliegue; puede ser `null`                |
| `associated_slug` | Slug del recurso                                   |

## Entregada vs. aprobada

| Concepto | Condicion | Significado |
|----------|-----------|-------------|
| Entregada | `task_status == "DONE"` | El estudiante envio la tarea |
| Aprobada | `task_status == "DONE"` **y** `revision_status == "APPROVED"` | Ademas fue revisada y aceptada |
| Entregada sin revisar | `task_status == "DONE"` **y** `revision_status == "PENDING"` | Enviada, esperando revision |
| Rechazada | `task_status == "DONE"` **y** `revision_status == "REJECTED"` | Enviada, no aceptada |

Esta skill lista **entregas** (`task_status=DONE`) y reporta el estado de
revision como dato secundario. No afirma que una tarea este aprobada solo
por estar entregada.

## Procedimiento

### Paso 1: Consultar la API

Ejecutar mediante exec:

```sh
python3 - <<'AUTH_PY'
import os
import subprocess

url = "https://breathecode.herokuapp.com/v1/assignment/user/me/task?task_status=DONE"
token = os.environ.get("FOURGEEKS_TOKEN", "")

if not token:
    print("AUTH_ERROR: FOURGEEKS_TOKEN no disponible")
else:
    result = subprocess.run(
        [
            "curl",
            "--silent",
            "--show-error",

            "--write-out", "HTTP_CODE:%{http_code}\\n",
            "--max-time", "20",
            "-H", "Authorization: Token " + token,
            url,
        ],
        capture_output=True,
        text=True,
        timeout=25,
    )

    print("CURL_EXIT:", result.returncode)
    import json
    body, sep, status = result.stdout.rpartition("HTTP_CODE:")
    print("HTTP_CODE:", status.strip() if sep else "NO_DETECTADO")
    if result.returncode == 0 and sep and status.strip() == "200":
        try:
            data = json.loads(body)
            if isinstance(data, list):
                print("JSON_VALIDO: True")
                grupos = {estado: [] for estado in ("PENDING", "APPROVED", "REJECTED", "IGNORED")}
                desconocidos = 0
                for tarea in data:
                    if not isinstance(tarea, dict) or tarea.get("task_status") != "DONE":
                        continue
                    estado = tarea.get("revision_status")
                    if estado in grupos:
                        grupos[estado].append(tarea)
                    else:
                        desconocidos += 1
                print("ENTREGAS_RECUPERADAS:", len(data))
                print("CON_GITHUB_URL:", sum(bool(t.get("github_url")) for t in data if isinstance(t, dict)))
                print("CON_LIVE_URL:", sum(bool(t.get("live_url")) for t in data if isinstance(t, dict)))
                for estado, tareas in grupos.items():
                    print("REVISION_" + estado + ":", len(tareas))
                    for tarea in tareas[:5]:
                        print("TAREA:", tarea.get("title"), "| FECHA:", tarea.get("delivered_at"), "| GITHUB:", tarea.get("github_url"), "| LIVE:", tarea.get("live_url"))
                print("REVISION_DESCONOCIDA:", desconocidos)
            else:
                print("JSON_VALIDO:", isinstance(data, dict))
                print("FORMATO:", type(data).__name__)
        except json.JSONDecodeError:
            print("ERROR: JSON_INVALIDO")

    if result.returncode != 0:
        print("AUTH_ERROR: fallo de transporte")
AUTH_PY
```

### Paso 2: Verificar conexion y codigo HTTP

| Situacion | Significado | Respuesta |
|-----------|-------------|-----------|
| `EXIT_CODE != 0` | Fallo de DNS/TLS/conexion | NO COMPROBADA |
| `200` | Datos disponibles | Procesar JSON |
| `401` | Token rechazado o ausente | Informar bloqueo de autenticacion |
| `403` | Token sin permisos | Informar acceso prohibido |
| `404` | Ruta no encontrada | Informar error de ruta |
| Otro codigo | No contemplado | NO COMPROBADA |

`000` no es un codigo HTTP valido. Ante duda, informar el bloqueo sin
inventar resultados.

### Paso 3: Procesar la respuesta

El Paso 1 procesa directamente el JSON recibido, cuenta enlaces GitHub y Live y presenta hasta cinco ejemplos por estado. No ejecutar procesos adicionales ni depender de RESPONSE_JSON.

### Paso 4: Responder

En espanol, una entrada por tarea entregada. Si `github_url`, `live_url` o
`delivered_at` son `null` o cadena vacia, indicar explicitamente que el dato
no esta disponible; nunca rellenar con suposiciones ni enlaces inventados.

## Manejo de casos limites

- **Respuesta vacia (`[]`):** informar "No hay tareas entregadas registradas".
- **JSON malformado:** informar error de formato; no reintentar con datos inventados.
- **Tareas con `task_status=PENDING` en la respuesta:** ignorarlas; no son entregas.
- **Campos `null`:** mostrar "no disponible" en lugar de omitir el dato en silencio.
- **Error de conexion:** responder NO COMPROBADA indicando el fallo.

## Verificacion

La skill es funcional cuando:
1. Con token valido y entregas, lista titulo, tipo, fecha, GitHub y despliegue.
2. Con token valido y sin entregas, informa que no hay tareas entregadas.
3. Con campos `null`, indica "no disponible" sin inventar valores.
4. Con token invalido o error de conexion, informa el bloqueo sin inventar datos.
