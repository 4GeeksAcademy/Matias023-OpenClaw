---
name: fourgeeks-auth
description: >-
  Verifica si la sesion del estudiante en 4Geeks esta autenticada consultando
  la API oficial BreatheCode. Responde SI / NO / NO COMPROBADA segun el
  codigo HTTP real. No inventa resultados de autenticacion.
allowed-tools:
  - exec
---

# FourGeeks Auth

## Proposito

Verificar si el usuario tiene una sesion activa en 4Geeks consultando
el endpoint `/v1/auth/user/me` de la API BreatheCode.

## Disparadores

- "?Estoy autenticado en 4Geeks?"
- "Verifica mi sesion de 4Geeks"
- Cualquier pregunta que involucre verificar la autenticacion en 4Geeks

## Procedimiento

### Paso 1: Consultar la API

Ejecutar mediante `exec` del Gateway:

```sh
python3 - <<'AUTH_PY'
import os
import subprocess

url = "https://breathecode.herokuapp.com/v1/auth/user/me"
token = os.environ.get("FOURGEEKS_TOKEN", "")

if not token:
    print("AUTH_ERROR: FOURGEEKS_TOKEN no disponible")
else:
    result = subprocess.run(
        [
            "curl",
            "--silent",
            "--show-error",
            "--output", "/dev/null",
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
    print(result.stdout.strip())

    if result.returncode != 0:
        print("AUTH_ERROR: fallo de transporte")
AUTH_PY
```

OpenClaw inyecta $FOURGEEKS_TOKEN como variable de entorno opaca
en los comandos ejecutados via exec en el Gateway. El shell resuelve
la variable al ejecutar curl, y el proxy puede sustituirla durante
la solicitud HTTPS saliente al dominio autorizado.

### Paso 2: Interpretar el codigo HTTP

| Codigo  | Significado                   | Respuesta        |
|---------|-------------------------------|------------------|
| `200`   | Token valido, sesion activa   | **SI**           |
| `401`   | Token rechazado o ausente     | **NO**           |
| `403`   | Token presente sin permisos   | **NO**           |
| Otro    | Codigo HTTP no contemplado    | **NO COMPROBADA** |

Si el comando falla con codigo de error distinto de 0, la conexion
no se completo y la respuesta debe ser **NO COMPROBADA**.

### Paso 3: Responder

En espanol, sin exponer credenciales ni cabeceras de autorizacion.

## Verificacion

La skill es funcional cuando:
1. La API responde HTTP 200 y se responde SI.
2. Con un token invalido, la API responde 401 y se responde NO.
3. Con un error de red simulado, se responde NO COMPROBADA.