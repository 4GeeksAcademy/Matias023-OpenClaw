# SKILLS_DESIGN.md — Diseño de habilidades de NEXUS

## Objetivo

Diseñar dos skills personalizadas de OpenClaw antes de implementarlas.
Ambas utilizarán exclusivamente herramientas previamente configuradas.
Este documento debe registrarse en Git antes de crear las skills.

## Skill 1 — nexus-weekly-planner

### ¿Qué hace?

Transforma objetivos semanales en un plan priorizado, crea un documento
de planificación y puede programar un compromiso en Google Calendar.

### ¿Qué input necesita?

- Objetivos y actividades proporcionados por el usuario.
- Semana o intervalo de fechas.
- Prioridades y fechas límite conocidas.
- Disponibilidad horaria proporcionada por el usuario.
- Autorización explícita para crear documentos y eventos.

No inventa disponibilidad, fechas límite ni compromisos.
Solicita los datos imprescindibles cuando falten.

### ¿Cómo es un buen output?

- Plan semanal estructurado por prioridades.
- Documento creado mediante Google Docs.
- Evento creado mediante Google Calendar cuando se solicite.
- Fechas y zona horaria America/Santiago verificadas.
- Enlaces o identificadores de los recursos creados.
- Confirmación clara de operaciones exitosas y fallidas.

### Prueba prevista

Solicitar un plan con tres objetivos, fechas definidas y un compromiso.
Comprobar que el documento contiene los objetivos y que el evento
aparece en Google Calendar con el horario solicitado.

## Skill 2 — nexus-meeting-notes

### ¿Qué hace?

Convierte notas de una reunión en una minuta profesional y crea
un documento organizado mediante Google Docs.

### ¿Qué input necesita?

- Notas o transcripción proporcionada por el usuario.
- Nombre de la reunión.
- Fecha, cuando esté disponible.
- Participantes, cuando estén identificados.
- Autorización para crear el documento.

No inventa responsables, acuerdos, fechas ni decisiones.
Identifica explícitamente la información que no fue proporcionada.

### ¿Cómo es un buen output?

- Resumen de los temas tratados.
- Decisiones respaldadas por las notas originales.
- Acciones pendientes y responsables solo cuando consten.
- Preguntas abiertas y datos faltantes.
- Documento creado mediante Google Docs.
- Enlace o identificador verificable del documento.

### Prueba prevista

Proporcionar notas reales con decisiones y tareas identificables.
Comprobar que el documento conserva la información original,
no agrega acuerdos inexistentes y distingue datos faltantes.

## Herramientas previstas

- Telegram: recepción de solicitudes y comunicación de resultados.
- Zapier MCP: ejecución de acciones disponibles.
- Google Docs: Create Document From Text.
- Google Calendar: Create Detailed Event.

La disponibilidad de cada acción se verificará antes de utilizarla.

## Seguridad y restricciones

- No instalar nuevos servicios, API ni flujos OAuth.
- No exponer credenciales o información privada.
- Evitar crear recursos duplicados ante reintentos.
- Confirmar cualquier operación que no esté autorizada.
- No declarar éxito sin comprobar el resultado de la herramienta.

## Orden obligatorio de trabajo

1. Crear y revisar este documento de diseño.
2. Registrar el diseño mediante un commit independiente.
3. Implementar las dos skills de OpenClaw.
4. Ejecutar pruebas reales y registrar evidencias.
5. Publicar la entrega en el repositorio correspondiente.
