---
name: nexus-meeting-notes
description: Convierte apuntes o transcripciones de reuniones en minutas estructuradas y crea documentos en Google Docs mediante Zapier MCP cuando el usuario lo solicita.
---

# NEXUS Meeting Notes

## Propósito

Transformar notas desordenadas de reuniones en minutas claras,
profesionales y verificables.

Utilizar esta skill cuando el usuario solicite:
- Preparar una minuta.
- Organizar apuntes de una reunión.
- Identificar acuerdos, decisiones y tareas pendientes.
- Crear un documento de reunión en Google Docs.

Aplicar las preferencias y restricciones de USER.md, SOUL.md,
AGENTS.md y TOOLS.md.

## Datos de entrada

Identificar:

1. Nombre o tema de la reunión.
2. Notas, apuntes o transcripción proporcionada.
3. Fecha de la reunión, cuando esté disponible.
4. Participantes identificados.
5. Decisiones y acuerdos explícitos.
6. Acciones pendientes y responsables mencionados.
7. Solicitud o autorización para crear el documento.

No inventar participantes, fechas, acuerdos ni responsables.

Si falta información, indicarlo claramente.

## Procedimiento

### Paso 1: Analizar las notas

- Identificar los asuntos tratados.
- Separar hechos de opiniones.
- Extraer decisiones explícitas.
- Identificar tareas pendientes.
- Reconocer responsables y plazos cuando estén presentes.
- Detectar preguntas abiertas.

No transformar propuestas en decisiones confirmadas.

### Paso 2: Organizar la minuta

Construir un documento con la siguiente estructura:

# Minuta de reunión

## Información general

- Reunión:
- Fecha:
- Participantes:

Utilizar "No informado" cuando corresponda.

## Resumen ejecutivo

Sintetizar los principales asuntos tratados.

## Temas tratados

Organizar los temas en secciones claras.

## Decisiones confirmadas

Incluir únicamente decisiones respaldadas por las notas.

## Acciones pendientes

Para cada acción, identificar:
- Tarea.
- Responsable, si consta.
- Fecha límite, si consta.
- Estado, si consta.

## Preguntas abiertas

Registrar asuntos pendientes de aclaración.

## Información faltante

Indicar los datos necesarios para completar la minuta.

### Paso 3: Verificar fidelidad

Antes de generar el documento:

- Comprobar que las decisiones aparecen en las notas originales.
- Evitar agregar responsabilidades inexistentes.
- No inventar fechas ni compromisos.
- Mantener el significado de las intervenciones.
- Diferenciar claramente hechos de información pendiente.

### Paso 4: Crear documento

Cuando el usuario solicite crear o guardar la minuta:

Utilizar la skill disponible `zapier-mcp-actions`.

Localizar y ejecutar la acción habilitada:

Google Docs — Create Document From Text.

Proporcionar un título identificable y el contenido completo
de la minuta.

No inventar nombres internos de herramientas, parámetros,
identificadores ni enlaces.

Confirmar la creación solamente cuando la herramienta
devuelva un resultado satisfactorio.

Ante un error o resultado ambiguo, comprobar primero si
el documento existe antes de reintentar, para evitar duplicados.

### Paso 5: Comunicar resultados

Responder principalmente en español.

Entregar:

1. Resumen breve del contenido generado.
2. Decisiones y tareas identificadas.
3. Enlace o identificador real del documento, si está disponible.
4. Información faltante.
5. Errores o acciones pendientes de verificación.

No afirmar que un documento fue creado sin confirmación.

## Restricciones

- Utilizar únicamente integraciones existentes.
- No configurar nuevas API ni conexiones OAuth.
- No divulgar información privada.
- No enviar documentos a terceros sin autorización.
- No inventar decisiones, responsables ni plazos.
- No ejecutar acciones no solicitadas.
- Respetar las reglas permanentes de AGENTS.md.

## Criterios de éxito

La skill se considera validada cuando:

- Procesa notas reales de una reunión.
- Produce una minuta estructurada.
- Conserva fielmente las decisiones originales.
- Identifica tareas y responsables sin inventarlos.
- Distingue los datos ausentes.
- Crea un documento verificable en Google Docs cuando se solicite.
- Comunica correctamente el resultado de la operación.
