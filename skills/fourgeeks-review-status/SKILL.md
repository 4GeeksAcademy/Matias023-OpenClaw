---
name: fourgeeks-review-status
description: >-
  Consulta el estado de revision de las tareas academicas entregadas en 4Geeks
  mediante la API oficial BreatheCode. Distingue entregadas sin revisar,
  aprobadas y rechazadas. No inventa tareas, calificaciones ni fechas.
allowed-tools:
  - exec
---

# FourGeeks Review Status

## Proposito

Mostrar el estado de revision de las tareas academicas que el estudiante ya
entrego, diferenciando las pendientes de revision de las aprobadas y las
rechazadas.

## Disparadores

- "?Como van mis revisiones en 4Geeks?"
- "?Que tareas tengo pendientes de revision?"
- "?Que tareas me aprobaron y cuales me rechazaron?"

## Endpoint

```
GET https://breathecode.herokuapp.com/v1/assignment/user/me/task?task_status=DONE
```

**Fuentes oficiales (`breatheco-de/apiv2`):**
- Ruta: `assignments/urls.py` linea 84 — `path("user/me/task", TaskMeView...)`, prefijo raiz `v1/assignment/`
- Filtros: `assignments/views.py` — `TaskMeView.get()` acepta `task_status` y `revision_status` como query params separados por coma
- Serializador del listado: `TaskUserSmallSerializer` en `assignments/serializers.py`
- Modelo y choices: `assignments/models.py` lineas 75-88 (`TaskStatus`, `RevisionStatus`)

## Campos expuestos por la API

| Campo             | Uso en esta skill                        |
|-------------------|------------------------------------------|
| `id`              | Identificador unico de la tarea          |
| `title`           | Titulo mostrado en el resumen            |
| `task_type`       | `PROJECT`, `QUIZ`, `LESSON`, `EXERCISE`  |
| `task_status`     | `PENDING` o `DONE`                       |
| `revision_status` | `PENDING`, `APPROVED`, `REJECTED`, `IGNORED` |
| `delivered_at`    | Fecha de entrega (puede ser `null`)      |
| `reviewed_at`     | Fecha de revision (puede ser `null`)     |
| `associated_slug` | Slug del recurso                         |

## Reglas de clasificacion

Se consultan solo tareas con `task_status=DONE` (ya entregadas) y se agrupan
por `revision_status`:

| Grupo | Condicion exacta |
|-------|------------------|
| Entregadas pendientes de revision | `task_status == "DONE"` **y** `revision_status == "PENDING"` |
| Aprobadas | `task_status == "DONE"` **y** `revision_status == "APPROVED"` |
| Rechazadas | `task_status == "DONE"` **y** `revision_status == "REJECTED"` |
| Revision ignorada | `task_status == "DONE"` **y** `revision_status == "IGNORED"` |

**Distincion critica:** una tarea con `task_status=PENDING` NO fue entregada
todavia. No debe aparecer en este informe ni clasificarse como "pendiente de
revision". Solo `task_status=DONE` con `revision_status=PENDING` es una entrega
esperando revision.

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
                for estado, tareas in grupos.items():
                    print("REVISION_" + estado + ":", len(tareas))
                    for tarea in tareas:
                        print("TAREA:", tarea.get("title"), "| FECHA:", tarea.get("delivered_at"))
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

Nunca interpretar `000` como codigo HTTP valido. Ante duda, informar el
bloqueo sin inventar resultados.

### Paso 3: Procesar la respuesta

La clasificación se ejecuta directamente en el Paso 1 sobre el JSON recibido. No utilizar RESPONSE_JSON ni procesos separados.

### Paso 4: Responder

En espanol, agrupando por estado de revision y sin exponer credenciales ni
cabeceras. Si `delivered_at` es `null`, indicar "sin fecha de entrega"
en lugar de inventar una fecha.

## Manejo de casos limites

- **Respuesta vacia (`[]`):** informar "No hay tareas entregadas registradas".
- **JSON malformado:** informar error de formato, sin reintentar con datos inventados.
- **Tareas con `task_status=PENDING` en la respuesta:** ignorarlas; no son entregas.
- **`revision_status` desconocido:** listarlo aparte como valor no reconocido, sin reclasificarlo.

## Verificacion

La skill es funcional cuando:
1. Con token valido y entregas, agrupa correctamente por estado de revision.
2. Con token valido y sin entregas, informa que no hay tareas entregadas.
3. Con token invalido, informa el codigo HTTP sin inventar tareas.
4. Con error de conexion, responde NO COMPROBADA.
