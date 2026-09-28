# n8n Agents IA, tools and MCP server
Workflow Agentes de IA, Tools y MCP Servers

# Workflow Agent IA

Asistente de IA en n8n que administra correo electrónico y calendario de forma conversacional. El agente recibe mensajes por chat, razona con un modelo de lenguaje y ejecuta acciones sobre Gmail y Google Calendar a través de herramientas MCP, siguiendo reglas de productividad predefinidas.

## Descripción

Este workflow implementa un **AI Agent** que actúa como asistente personal para gestión de correo y agenda. Prioriza la organización, responde de forma corta y directa, y confirma siempre antes de crear, modificar o eliminar información.

## Arquitectura

```
When chat message received  ──►  AI Agent  ──►  respuesta
                                    │
        ┌───────────────┬──────────┼──────────────┬─────────────────┐
        │               │          │              │                 │
   Chat Model        Memory      Tool:         Tool:             Tool:
 (Gemini/Anthropic) (Buffer)   Pokemons      MCP Gmail        MCP Calendar
```

## Componentes

| Nodo | Tipo | Función |
|------|------|---------|
| **When chat message received** | Chat Trigger | Punto de entrada; recibe los mensajes del usuario. |
| **AI Agent** | LangChain Agent | Orquesta el razonamiento y decide qué herramienta usar. |
| **Google Gemini Chat Model** | Modelo de lenguaje | Modelo activo que impulsa al agente. |
| **Anthropic Chat Model** | Modelo de lenguaje | Modelo alternativo (Claude) disponible. |
| **Simple Memory** | Memoria (buffer window) | Mantiene contexto de las últimas 20 interacciones. |
| **Obtener_Pokemons** | Tool Workflow | Consulta información de Pokémon (sub-workflow). |
| **MCP Gmail Client** | MCP Client Tool | Acceso al correo: leer, redactar, clasificar y enviar. |
| **MCP Calendar Client** | MCP Client Tool | Acceso al calendario: crear, modificar, cancelar y consultar eventos. |

## Reglas del agente

### Calendario
- Horario laboral: **8am a 4pm, lunes a viernes**.
- No agenda fuera del horario laboral ni en fines de semana.
- Verifica disponibilidad antes de agendar (sin traslapes).
- Confirma siempre antes de crear, modificar o cancelar eventos.
- Recordatorios 15 minutos antes de cada evento.
- Resumen diario al inicio del día y resumen semanal los lunes.

### Correo electrónico
- Lectura en texto plano (sin HTML).
- Crea un borrador en HTML y lo devuelve para confirmar antes de enviar.
- Los envíos usan formato HTML (negritas, títulos, estructura apropiada).
- Clasifica por prioridad: Alta > Media > Baja.
- Ignora promociones, spam y notificaciones irrelevantes.
- Confirma antes de eliminar o archivar.
- Propone respuestas rápidas para correos de alta prioridad.
- Firma siempre como **John Peñaloza**.

## Requisitos

- Instancia de **n8n** (probado en n8n Cloud).
- Credenciales configuradas para:
  - Modelo de lenguaje: **Google Gemini** (activo) y/o **Anthropic Claude**.
  - **MCP Gmail** y **MCP Calendar** (servidores MCP propios expuestos vía webhook).
- Sub-workflow **Pokemon Finder Tool** disponible en la instancia.

## Instalación

1. Importa el archivo JSON del workflow en tu instancia de n8n (**Workflows → Import from File**).
2. Configura las credenciales de cada nodo (modelo de lenguaje y clientes MCP).
3. Ajusta las URLs de los endpoints MCP a las de tu propia instancia.
4. Verifica que el sub-workflow de la herramienta Pokémon exista.
5. Activa el workflow.

## Uso

Abre el chat del Chat Trigger y escribe peticiones en lenguaje natural, por ejemplo:
- "Resume mis correos importantes de hoy."
- "Agenda una reunión el martes a las 10am."
- "Redacta un correo de seguimiento para el cliente X."

## Notas

- Actualmente el modelo conectado al agente es **Google Gemini**; el nodo de Anthropic está presente pero sin conexión activa al agente.
- Los endpoints MCP y las credenciales son específicos de la instancia original y deben reemplazarse al desplegar en otro entorno.
