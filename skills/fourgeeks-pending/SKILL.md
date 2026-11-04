---
name: fourgeeks-pending
description: >-
  Consulta las tareas academicas pendientes del estudiante en 4Geeks mediante
  la API oficial BreatheCode. Muestra su titulo, tipo y estado real.
  No inventa resultados ni fechas limite.
allowed-tools:
  - exec
---

# FourGeeks Pending

## Proposito

Listar las tareas academicas del estudiante autenticado que aun no han sido
completadas (`task_status=PENDING`), mostrando su titulo, tipo, estado de
revision y metadatos disponibles.

## Disparadores

- "?Que tareas tengo pendientes?"
- "Muestrame mis tareas sin entregar"
- "?Que me falta por hacer en 4Geeks?"

## Endpoint

```
GET /v1/assignment/user/me/task?task_status=PENDING
```

**Fuente:** `breatheco-de/apiv2` — `urls.py` linea 84 (`user/me/task`),
`urls.py` raiz monta el prefijo `v1/assignment/`, y `views.py` linea 28-30
acepta el filtro `task_status` como query parameter.

## Campos expuestos por la API (según `TaskGETSerializer`)

| Campo             | Tipo         | Proposito                           |
|-------------------|--------------|-------------------------------------|
| `id`              | integer      | Identificador unico de la tarea     |
| `title`           | string       | Titulo de la tarea                  |
| `task_status`     | string       | `PENDING` o `DONE`                  |
| `revision_status` | string       | `PENDING`, `APPROVED`, `REJECTED`, `IGNORED` |
| `task_type`       | string       | `PROJECT`, `QUIZ`, `LESSON`, `EXERCISE` |
| `associated_slug` | string       | Slug del recurso asociado           |
| `github_url`      | string|null  | URL del repositorio GitHub          |
| `opened_at`       | datetime|null| Fecha de apertura de la tarea       |
| `description`     | string       | Descripcion textual                 |

## Reglas de interpretacion

| Condicion                                          | Interpretacion                              |
|----------------------------------------------------|---------------------------------------------|
| `task_status=PENDING` y sin `opened_at`            | Tarea sin abrir, pendiente                  |
| `task_status=PENDING` con `opened_at` presente     | Tarea abierta pero sin completar            |
| `task_status=DONE` con `revision_status=PENDING`   | Entregada sin revision (no es pendiente)    |
| `task_status=DONE` con `revision_status=REJECTED`  | Entregada pero rechazada (no es pendiente)  |
| `task_status=DONE` con `revision_status=APPROVED`  | Completada y aprobada (no es pendiente)     |

Esta skill solo consulta tareas con `task_status=PENDING`. No confluye
tareas entregadas y rechazadas con tareas pendientes de entrega.

El modelo `Task` (en `models.py`) no tiene campo `due_date` ni `priority`.
No se deben inventar fechas limite ni niveles de prioridad.

## Procedimiento

### Paso 1: Consultar la API

Ejecutar mediante exec:

```sh
python3 - <<'AUTH_PY'
import os
import subprocess

url = "https://breathecode.herokuapp.com/v1/assignment/user/me/task?task_status=PENDING"
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
                print("TAREAS_PENDIENTES:", len(data))
            else:
                print("JSON_VALIDO:", isinstance(data, dict))
                print("FORMATO:", type(data).__name__)
        except json.JSONDecodeError:
            print("ERROR: JSON_INVALIDO")

    if result.returncode != 0:
        print("AUTH_ERROR: fallo de transporte")
AUTH_PY
```

### Paso 2: Interpretar el codigo HTTP

| Codigo | Significado | Respuesta |
|--------|-------------|-----------|
| `200`  | Autenticado, tareas pendientes disponibles | Procesar JSON |
| `401`  | Token rechazado o ausente | Informar bloqueo de autenticacion |
| `403`  | Token sin permisos | Informar acceso prohibido |
| `404`  | Ruta no encontrada | Informar error de ruta |
| Otro / error conexion | No se pudo consultar | NO COMPROBADA |

### Paso 3: Procesar la respuesta (si HTTP 200)

El Paso 1 ya procesa el JSON mediante json.loads(body) y muestra la cantidad de tareas pendientes cuando la respuesta es una lista.

Usar exclusivamente los datos realmente recuperados. Si la respuesta es un objeto JSON, inspeccionar su estructura antes de contar registros. No inventar tareas, estados ni fechas.

### Paso 4: Responder

En espanol, sin exponer credenciales ni cabeceras de autorizacion.

## Verificacion

La skill es funcional cuando:
1. Con token valido y tareas pendientes, lista sus titulos, tipos y estados.
2. Con token valido y sin tareas pendientes, responde "No tienes tareas pendientes".
3. Con token invalido, informa el codigo HTTP y que no se pudo autenticar.
4. Con error de red, informa que no se pudo comprobar.