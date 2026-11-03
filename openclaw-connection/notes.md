# NEXUS - Integración de OpenClaw con Telegram, Google Docs y Google Calendar

## 1. Objetivo

Configurar un agente personal de inteligencia artificial llamado NEXUS utilizando OpenClaw, conectarlo con Telegram e integrar Google Docs y Google Calendar mediante Zapier MCP.

## 2. Infraestructura

- Sistema operativo del servidor: Ubuntu 22.04.5 LTS.
- Entorno de ejecución: VPS remoto.
- Versión de OpenClaw: 2026.10.9.
- Agente: NEXUS.
- Canal de comunicación: Telegram.
- Integraciones externas: Zapier MCP, Google Docs y Google Calendar.
- Autenticación MCP: OAuth.
- Comunicación MCP: HTTP.

## 3. Configuración de Telegram

Se creó el bot NexusMatias023Bot mediante BotFather.

El canal Telegram se configuró en OpenClaw y se habilitó el acceso mediante un token almacenado en el sistema de secretos.

Se autorizó la cuenta del administrador mediante el mecanismo de emparejamiento de OpenClaw.

La conexión se verificó enviando un mensaje desde Telegram y recibiendo una respuesta del agente NEXUS.

## 4. Configuración de Zapier MCP

Se creó y configuró un servidor MCP en Zapier.

Se registraron las siguientes acciones:

- Google Docs: Create Document From Text.
- Google Calendar: Create Detailed Event.

El servidor MCP se agregó directamente a OpenClaw utilizando su interfaz nativa de administración MCP.

La autenticación OAuth se completó mediante un túnel SSH temporal para recibir el callback en el VPS.

La conexión se comprobó con el comando:

openclaw mcp probe

Resultado verificado:

- 17 herramientas MCP detectadas.
- 0 diagnósticos reportados.

## 5. Pruebas individuales

### Google Docs

NEXUS recibió una solicitud desde Telegram y creó el documento:

Prueba NEXUS - 4Geeks

Se abrió el documento en Google Docs y se verificó que contenía el texto solicitado.

### Google Calendar

NEXUS creó el evento:

Revisión documento NEXUS - 4Geeks

- Fecha: 4 de noviembre de 2026.
- Horario: 15:00 a 15:30.
- Zona horaria: America/Santiago.

Se verificó visualmente el evento en Google Calendar.

## 6. Prueba integral

Se envió una única solicitud a NEXUS mediante Telegram para realizar dos operaciones consecutivas.

Primero, el agente creó el documento:

Prueba Integral NEXUS - 4Geeks

Después, creó el evento:

Revisión Integral NEXUS - 4Geeks

- Fecha: 5 de noviembre de 2026.
- Horario: 16:00 a 16:30.
- Zona horaria: America/Santiago.

El evento incluyó un enlace al documento generado durante la misma ejecución.

Se verificaron ambos recursos directamente en Google Docs y Google Calendar.

Finalmente, NEXUS confirmó la ejecución de las tareas mediante Telegram.

## 7. Problemas encontrados y soluciones

- Token de Telegram rechazado: se actualizó la credencial y se recargaron los secretos.
- Acceso inicial denegado en Telegram: se aprobó la solicitud de emparejamiento.
- Callback OAuth inaccesible desde Linux Mint: se utilizó un túnel SSH local hacia el VPS.
- Verificación de herramientas MCP: se utilizó la salida JSON de OpenClaw para identificar las 17 herramientas disponibles.

## 8. Resultado

Se verificó correctamente el flujo:

Telegram -> NEXUS -> OpenClaw -> Zapier MCP -> Google Docs y Google Calendar -> Telegram

El agente pudo crear documentos, programar eventos y confirmar los resultados desde Telegram.

## 9. Seguridad

Las credenciales de Telegram y OAuth no se incluyen en esta entrega.

Se utilizaron mecanismos de almacenamiento de secretos y autenticación OAuth.

Las evidencias se prepararon procurando no exponer tokens, claves de API ni otros datos sensibles.
EOF

echo "✅ notes.md creado correctamente"
