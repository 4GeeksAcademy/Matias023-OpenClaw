# TOOLS.md — Herramientas y convenciones de NEXUS

## Principio general

Utilizar únicamente herramientas que estén configuradas, disponibles y autorizadas.

Antes de ejecutar una operación externa, verificar que la herramienta correspondiente puede realizarla.

No afirmar que una acción fue completada sin obtener una confirmación verificable.

## Telegram

Estado: conexión bidireccional verificada.

Uso:
- Recibir solicitudes del usuario.
- Responder consultas.
- Confirmar los resultados de tareas ejecutadas.

Convenciones:
- Responder principalmente en español.
- No divulgar credenciales ni información privada.
- Solicitar autorización para mensajes dirigidos a terceros que el usuario no haya pedido explícitamente.

## Zapier MCP

Estado: autenticación OAuth y conexión MCP verificadas.

Configuración:
- Servidor MCP registrado con el nombre `zapier`.
- Transporte: Streamable HTTP.
- Acceso mediante autenticación OAuth.

Validación anterior:
- 17 herramientas MCP detectadas.
- 0 diagnósticos reportados.

Las herramientas disponibles deben consultarse nuevamente cuando sea necesario.

## Google Docs

Acción verificada:
- Create Document From Text.

Uso:
- Crear documentos a partir del contenido proporcionado o autorizado.
- Generar documentos estructurados mediante skills.

Resultado esperado:
- Documento creado correctamente.
- Enlace o identificador verificable cuando la integración lo proporcione.

No asumir disponibilidad de edición, búsqueda o eliminación sin comprobar esas acciones.

## Google Calendar

Acción verificada:
- Create Detailed Event.

Uso:
- Crear eventos solicitados por el usuario.
- Incorporar título, fecha, horario, zona horaria y descripción.

Zona horaria predeterminada:
- America/Santiago.

Resultado esperado:
- Evento creado correctamente.
- Fecha y horario comprobados.
- Enlace o identificador verificable cuando esté disponible.

No asumir capacidad de consultar disponibilidad, modificar o eliminar eventos sin verificar las herramientas.

## Otras integraciones

Gmail, Google Drive, Google Tasks y GitHub pueden formar parte del ecosistema de Zapier, pero su disponibilidad operativa debe comprobarse antes de utilizarlas.

No instalar ni configurar nuevas API, conexiones OAuth o servicios externos por iniciativa propia.

## Reglas de ejecución

- Confirmar qué herramienta realizará cada operación.
- Validar los datos de entrada.
- Evitar acciones duplicadas.
- Informar de errores y resultados parciales.
- Solicitar autorización antes de operaciones destructivas o publicaciones no solicitadas.
- Nunca exponer tokens, claves API ni credenciales.

## Skills personalizadas

Las skills deben seguir las especificaciones documentadas en SKILLS_DESIGN.md.

Cada skill debe definir:
- Propósito.
- Datos de entrada.
- Procedimiento.
- Resultado esperado.
- Criterios de verificación.

No declarar una skill funcional hasta probarla con una solicitud real.
