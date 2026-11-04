---
name: fourgeeks-progress
description: >-
  Calcula un resumen del progreso academico del estudiante en 4Geeks usando
  datos reales de la API BreatheCode. Cuenta tareas pendientes, entregadas,
  aprobadas y rechazadas. No inventa calificaciones ni porcentajes oficiales.
allowed-tools:
  - exec
---

# FourGeeks Progress

## Proposito

Resumir el progreso academico del estudiante autenticado calculando metricas
verificables a partir de sus tareas reales en BreatheCode.

## Disparadores

- "?Como va mi progreso en 4Geeks?"
- "Dame un resumen de mi avance"
- "?Cuantas tareas llevo aprobadas?"

## Endpoint

```
GET https://breathecode.herokuapp.com/v1/assignment/user/me/task
```

**Fuente:** `breatheco-de/apiv2` — `assignments/urls.py` linea 84
(`path("user/me/task", TaskMeView...)`), prefijo raiz `v1/assignment/`.

No existe un endpoint oficial de "progreso". Las metricas se calculan a
partir de la lista de tareas del usuario.

## Campos utilizados (segun `TaskUserSmallSerializer`)

| Campo             | Uso en las metricas                                  |
|-------------------|------------------------------------------------------|
| `id`              | Identificador unico de la tarea                      |
| `title`           | Titulo mostrado en el resumen                        |
| `task_status`     | `PENDING` o `DONE`                                   |
| `revision_status` | `PENDING`, `APPROVED`, `REJECTED`, `IGNORED`         |
| `task_type`       | `PROJECT`, `QUIZ`, `LESSON`, `EXERCISE`              |
| `delivered_at`    | Fecha de entrega (si aplica)                         |
| `associated_slug` | Slug del recurso                                     |

## Definicion de metricas

| Metrica | Condicion exacta |
|---------|------------------|
| Total de tareas | `len(todas_las_tareas)` recuperadas |
| Pendientes | `task_status == "PENDING"` |
| Entregadas | `task_status == "DONE"` |
| Aprobadas | `task_status == "DONE"` **y** `revision_status == "APPROVED"` |
| Rechazadas | `task_status == "DONE"` **y** `revision_status == "REJECTED"` |
| Entregadas sin revisar | `task_status == "DONE"` **y** `revision_status == "PENDING"` |

## Formulas

- Porcentaje de entrega = `(entregadas / total) * 100`
- Porcentaje de aprobacion = `(aprobadas / total) * 100`

**Condiciones obligatorias:**
- Si `total == 0`, NO dividir. Informar: "No hay datos suficientes para
  calcular porcentajes".
- Si la paginacion no permitio recuperar todas las tareas, NO presentar
  porcentajes como resultado completo; informar la cantidad recuperada y
  que el total podria ser mayor.
- Estos porcentajes NO son el avance oficial del programa academico de
  4Geeks. Son indicadores calculados sobre las tareas recuperadas de la API.

## Paginacion

`TaskMeView` usa `APIViewExtensions` con `paginate=True`, y los parametros
son `limit` y `offset` (`api_view_extensions.py` lineas 12-13).

Procedimiento: iniciar con `offset=0`, usar `limit=100`, incrementar
`offset` por cada pagina hasta recibir una pagina vacia o menor que `limit`.
Maximo de seguridad: 50 paginas (5000 tareas). Si se alcanza el tope,
informar que la recuperacion fue parcial.

## Procedimiento

### Paso 1: Recuperar todas las tareas

Ejecutar mediante `exec` del Gateway:

```sh
python3 - <<'PY'
import json, subprocess, os

BASE = "https://breathecode.herokuapp.com/v1/assignment/user/me/task"
LIMIT = 100
MAX_PAGES = 50
tareas, offset, paginas = [], 0, 0
error = None

while paginas < MAX_PAGES:
    url = f"{BASE}?limit={LIMIT}&offset={offset}"
    p = subprocess.run(
        ["curl", "--silent", "--show-error", "--max-time", "20",
         "-H", "Authorization: Token " + os.environ.get("FOURGEEKS_TOKEN", ""),
         "-w", "\n%{http_code}", url],
        capture_output=True, text=True,
    )
    if p.returncode != 0:
        error = f"curl exit {p.returncode}"
        break
    cuerpo, _, codigo = p.stdout.rpartition("\n")
    if codigo.strip() != "200":
        error = f"HTTP {codigo.strip()}"
        break
    try:
        lote = json.loads(cuerpo or "[]")
    except json.JSONDecodeError:
        error = "JSON invalido"
        break
    if not lote:
        break
    tareas.extend(lote)
    if len(lote) < LIMIT:
        break
    offset += LIMIT
    paginas += 1

if error:
    print("ERROR:" + error)
else:
    print("TOTAL:" + str(len(tareas)))
    print("PARCIAL:" + ("si" if paginas >= MAX_PAGES else "no"))
    print(json.dumps(tareas))
PY
```

El token se pasa via `os.environ` desde la variable inyectada por OpenClaw;
nunca se imprime ni se escribe en archivos.

### Paso 2: Interpretar el resultado

| Salida | Significado | Respuesta |
|--------|-------------|-----------|
| `ERROR:HTTP 200` no aparece y `TOTAL:n` | Datos obtenidos | Calcular metricas |
| `ERROR:HTTP 401` | Token rechazado o ausente | Informar bloqueo de autenticacion |
| `ERROR:HTTP 403` | Sin permisos | Informar acceso prohibido |
| `ERROR:HTTP 404` | Ruta no encontrada | Informar error de ruta |
| `ERROR:curl ...` | Fallo de conexion/DNS/TLS | NO COMPROBADA |

### Paso 3: Calcular y presentar

Con la lista de tareas, contar segun las condiciones de la tabla de metricas.
Presentar en espanol: total, pendientes, entregadas, aprobadas, rechazadas,
entregadas sin revisar, y los dos porcentajes (o la advertencia si `total == 0`
o si la recuperacion fue parcial).

Nunca exponer el token ni la cabecera Authorization.

## Verificacion

La skill es funcional cuando:
1. Con token valido y tareas, muestra conteos correctos y porcentajes.
2. Con token valido y cero tareas, informa que no hay datos suficientes.
3. Con token invalido, informa el codigo HTTP sin inventar metricas.
4. Con mas de una pagina, recupera todas antes de calcular.
