---
name: fourgeeks-projects
description: >-
  Consulta los proyectos y tareas asignados al estudiante en 4Geeks mediante
  la API oficial BreatheCode. Muestra su estado real de finalizacion y
  revision. Distingue tareas academicas de proyectos finales.
allowed-tools:
  - exec
---

# FourGeeks Projects

## Proposito

Consultar las asignaciones academicas del estudiante autenticado en 4Geeks
a traves de la API BreatheCode, mostrando el estado real de cada tarea
y proyecto final.

## Disparadores

- "?Que proyectos tengo asignados?"
- "?Cual es el estado de mis proyectos?"
- "Muestrame mis proyectos finales"
- "Muestrame mis tareas academicas"

## Endpoints oficiales

Fuente: https://github.com/breatheco-de/apiv2

El `urls.py` raiz monta `v1/assignment/` como prefijo base para las rutas
de `breathecode.assignments.urls`.

| Endpoint | Proposito |
|----------|-----------|
| `GET v1/assignment/user/me/task` | Tareas academicas del usuario |
| `GET v1/assignment/user/me/final_project` | Proyectos finales del usuario |

## Campos expuestos en la respuesta JSON

### Tareas (`TaskGETSerializer` en serializers.py:79)

- `id` — identificador numerico
- `title` — titulo de la tarea
- `task_status` — PENDING | DONE (finalizacion)
- `revision_status` — PENDING | APPROVED | REJECTED | IGNORED (revision)
- `task_type` — PROJECT | QUIZ | LESSON | EXERCISE (tipo)
- `associated_slug` — slug del recurso
- `description` — descripcion textual
- `github_url` — URL del repositorio GitHub
- `live_url` — URL desplegada
- `opened_at` — fecha de apertura
- `delivered_at` — fecha de entrega
- `reviewed_at` — fecha de revision
- `created_at` — fecha de creacion
- `updated_at` — fecha de actualizacion

### Proyectos finales (`FinalProjectGETSerializer` en serializers.py:680)

- `id` — identificador numerico
- `name` — nombre del proyecto
- `one_line_desc` — descripcion breve
- `description` — descripcion completa
- `project_status` — PENDING | DONE (finalizacion)
- `revision_status` — PENDING | APPROVED | REJECTED | IGNORED (revision)
- `visibility_status` — PRIVATE | UNLISTED | PUBLIC (visibilidad)
- `repo_url` — URL del repositorio
- `public_url` — URL desplegada
- `created_at` — fecha de creacion
- `updated_at` — fecha de actualizacion

## Procedimiento

### Paso 1: Consulta autenticada de referencia

El siguiente bloque demuestra el mecanismo de autenticación validado. Debe adaptarse a los endpoints de tareas y proyectos antes de ejecutar la skill.

```sh
python3 - <<'AUTH_PY'
import os
import subprocess

url = "https://breathecode.herokuapp.com/v1/assignment/user/me/task"
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

    # Consulta adicional: proyectos finales
    fp_url = "https://breathecode.herokuapp.com/v1/assignment/user/me/final_project"
    fp_result = subprocess.run(["curl", "-sS", "--max-time", "20", "-H", "Authorization: Token " + token, "-w", "\\nHTTP_CODE:%{http_code}", fp_url], capture_output=True, text=True, timeout=25)
    fp_body, fp_sep, fp_status = fp_result.stdout.rpartition("HTTP_CODE:")
    print("PROYECTOS_HTTP:", fp_status.strip() if fp_sep else "NO_DETECTADO")
    if fp_result.returncode == 0 and fp_sep and fp_status.strip() == "200":
        try:
            fp_data = json.loads(fp_body)
            print("PROYECTOS_JSON_VALIDO:", isinstance(fp_data, (list, dict)))
            print("PROYECTOS_REGISTROS:", len(fp_data) if isinstance(fp_data, list) else "FORMATO_OBJETO")
        except json.JSONDecodeError:
            print("PROYECTOS_JSON_INVALIDO")
    print("CURL_EXIT:", result.returncode)
    import json
    body, sep, status = result.stdout.rpartition("HTTP_CODE:")
    print("HTTP_CODE:", status.strip() if sep else "NO_DETECTADO")
    if result.returncode == 0 and sep and status.strip() == "200":
        try:
            data = json.loads(body)
            print("JSON_VALIDO:", isinstance(data, (list, dict)))
            print("REGISTROS:", len(data) if isinstance(data, list) else "FORMATO_OBJETO")
        except json.JSONDecodeError:
            print("ERROR: JSON_INVALIDO")

    if result.returncode != 0:
        print("AUTH_ERROR: fallo de transporte")
AUTH_PY
```

### Paso 2: Interpretar codigos HTTP

| Codigo | Significado | Accion |
|--------|-------------|--------|
| `200`  | Autenticado, datos disponibles | Procesar JSON y presentar resumen |
| `401`  | Token rechazado o ausente | Informar que la autenticacion fallo |
| `403`  | Token sin permisos | Informar acceso prohibido |
| Otro / error conexion | No se pudo consultar | Informar error de conexion |

### Paso 3: Procesar y responder

Usar exclusivamente el JSON validado en el Paso 1. Mostrar títulos, estados y enlaces solo cuando estén presentes en los datos reales. No inventar registros ni confundir tareas con proyectos finales.

## Verificacion

La skill es funcional cuando:
1. Con token valido y tareas asignadas, responde con datos reales de la API.
2. Con token valido y sin tareas, responde que no hay asignaciones.
3. Con token invalido, informa el codigo HTTP y que no se pudo autenticar.
4. Con error de red, informa que no se pudo comprobar.