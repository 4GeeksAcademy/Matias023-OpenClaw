---
name: nexus-weekly-planner
description: Organiza objetivos semanales en un plan priorizado y crea documentos o eventos mediante Zapier MCP cuando el usuario lo solicita.
---

# NEXUS Weekly Planner

## Cuándo utilizar esta skill

Utilízala cuando el usuario solicite organizar su semana, priorizar objetivos, preparar un plan semanal o programar compromisos relacionados.

Responde principalmente en español y adapta el resultado a las preferencias descritas en USER.md, SOUL.md y AGENTS.md.

## Datos de entrada

Identifica:

1. Objetivos y actividades.
2. Semana o intervalo de fechas.
3. Prioridades y fechas límite conocidas.
4. Disponibilidad horaria proporcionada.
5. Si el usuario solicita crear un documento, un evento o ambos.

No inventes fechas límite, horarios ni disponibilidad.

Si falta información indispensable para ejecutar una acción, solicita únicamente los datos necesarios.

## Procedimiento

### Paso 1: Interpretar

- Extrae los objetivos proporcionados.
- Identifica prioridades explícitas.
- Separa compromisos con fecha de tareas flexibles.
- Señala conflictos o información faltante.

### Paso 2: Planificar

Construye un plan que incluya:

- Objetivos de la semana.
- Prioridades.
- Actividades propuestas.
- Compromisos con fecha y hora confirmadas.
- Riesgos, dependencias y asuntos pendientes.

No afirmes haber consultado la disponibilidad del calendario si no dispones de una herramienta verificada para hacerlo.

### Paso 3: Crear documento cuando se solicite

Utiliza la skill disponible `zapier-mcp-actions` para localizar y ejecutar la acción habilitada:

Google Docs — Create Document From Text.

Genera un documento con título identificable y contenido estructurado.

No inventes nombres internos de herramientas MCP, parámetros ni identificadores.

Confirma la creación únicamente después de recibir una respuesta satisfactoria de la herramienta.

### Paso 4: Crear evento cuando se solicite

Utiliza `zapier-mcp-actions` para localizar y ejecutar:

Google Calendar — Create Detailed Event.

Requiere fecha, hora de inicio y hora de término.

Utiliza America/Santiago como zona horaria predeterminada, salvo indicación contraria.

No programes eventos con horarios inferidos.

Comprueba los datos antes de ejecutar la creación.

Si una operación debe repetirse tras un fallo, comprueba primero si el recurso ya fue creado para evitar duplicados.

### Paso 5: Comunicar el resultado

Entrega:

1. Resumen del plan y prioridades.
2. Actividades y compromisos.
3. Enlaces o identificadores reales de los recursos creados, cuando estén disponibles.
4. Operaciones pendientes, fallidas o no verificadas.

No declares éxito si no existe confirmación verificable.

## Restricciones

- No configurar nuevas API, OAuth ni servicios externos.
- No utilizar herramientas que no estén disponibles.
- No modificar ni eliminar eventos existentes sin autorización.
- No publicar información privada.
- No ejecutar operaciones no solicitadas.
- Aplicar las reglas de AGENTS.md y TOOLS.md.

## Criterios de éxito

La skill se considera validada cuando:

- Organiza correctamente tres objetivos reales.
- Respeta fechas y prioridades proporcionadas.
- Crea un documento verificable en Google Docs cuando se solicite.
- Crea un evento verificable en Google Calendar cuando se solicite.
- Comunica resultados sin inventar información.
