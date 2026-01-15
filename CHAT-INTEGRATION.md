# Documentacion de Integracion de Chats - Ibon Palacio

## Resumen Ejecutivo

Todos los chats de las 4 landings del ecosistema Ibon Palacio estan conectados al mismo workflow de n8n (`ibon-web`), garantizando una experiencia de IA unificada.

**Webhook URL:** `https://flow.miclickderecho.com/webhook/ibon-web`
**Workflow ID:** `aw1Z0944wtQvvrm7`
**Estado:** Activo

---

## Arquitectura por Proyecto

### 1. ibon-landing (ibonpalacio.com)

| Componente | Tipo | Servicio | Webhook |
|------------|------|----------|---------|
| Chat Bubble | Flotante | `ChatService` | `https://flow.miclickderecho.com/webhook/ibon-web` |

**Archivos:**
- Servicio: `src/app/services/chat.service.ts`
- Componente: `src/app/components/common/chat-bubble/`

---

### 2. reto-gluteos (retogluteos.ibonpalacio.com)

| Componente | Tipo | Servicio | Webhook |
|------------|------|----------|---------|
| Chat Bubble | Flotante | `ChatService` | `https://flow.miclickderecho.com/webhook/ibon-web` |
| Home Chat | Integrado | `ChatService` | `https://flow.miclickderecho.com/webhook/ibon-web` |

**Archivos:**
- Servicio: `src/app/services/chat.service.ts`
- Burbuja: `src/app/components/common/chat-bubble/`
- Home Chat: `src/app/components/common/home-chat/`

**Caracteristicas del Home Chat:**
- Video avatar: `av-video.mp4`
- Icono play: Remixicon (ri-play-fill)
- Tema: Fucsia (#FF006E)

---

### 3. palacio-fitness (fitness.ibonpalacio.com)

| Componente | Tipo | Servicio | Webhook |
|------------|------|----------|---------|
| Chat Bubble | Flotante | `ChatService` | `https://flow.miclickderecho.com/webhook/ibon-web` |
| Home Chat | Integrado | `ChatService` | `https://flow.miclickderecho.com/webhook/ibon-web` |

**Archivos:**
- Servicio: `src/app/services/chat.service.ts`
- Burbuja: `src/app/components/layouts/chat-bubble/`
- Home Chat: `src/app/components/layouts/home-chat/`

**Caracteristicas del Home Chat:**
- Video avatar: `av-avatar.mp4`
- Icono play: CSS Triangle (FontAwesome no disponible en Angular 11)
- Tema: Naranja (#FF6B35)

**Nota:** Angular 11 - usa CSS en lugar de SCSS

---

### 4. by-ibon (by.ibonpalacio.com)

| Componente | Tipo | Servicio | Webhook |
|------------|------|----------|---------|
| Chat Bubble | Flotante | `IbonChatService` | `https://flow.miclickderecho.com/webhook/ibon-web` |
| Home Chat | Integrado | `IbonChatService` | `https://flow.miclickderecho.com/webhook/ibon-web` |

**Archivos:**
- Servicio Principal: `src/app/services/ibon-chat.service.ts`
- Servicio Legacy (no usado): `src/app/services/chat.service.ts`
- Burbuja: `src/app/components/common/chat-bubble/`
- Home Chat: `src/app/components/common/home-chat/`

**Caracteristicas del Home Chat:**
- Video avatar: `av-avatar.mp4`
- Icono play: Remixicon (ri-play-fill)
- Tema: Burgundy (#722F37)
- Funcionalidades avanzadas: Audio recording, Camera, File upload

**Nota:** El `chat.service.ts` tiene webhook vacio pero NO se usa. Todos los componentes usan `IbonChatService`.

---

## Workflow n8n: ibon-web

### Informacion General
- **ID:** `aw1Z0944wtQvvrm7`
- **Nombre:** ibon-web
- **Estado:** Activo
- **Creado:** 2026-01-10
- **Actualizado:** 2026-01-15
- **Total Nodos:** 51

### Arquitectura del Workflow

```
[Webhook] --> [Classify-Source] --> [Switch-Origin] --> [Set-Msg-*] --> [Merge-Origins]
                                          |
                    ┌─────────────────────┼─────────────────────┐
                    |           |         |         |           |
                    v           v         v         v           v
            [Set-Msg-     [Set-Msg-  [Set-Msg-  [Set-Msg-  [Set-Msg-
             IbonLanding]  RetoGluteos] PalacioFitness] ByIbon] WhatsApp]
                    |           |         |         |           |
                    └─────────────────────┼─────────────────────┘
                                          v
                                   [Merge-Origins]
                                          |
                                          v
                              [IF Clear Memory] --> [Text Classifier] --> [AI Agent]
                                                                              |
                                                                              v
                                                               [mail-agent, calendar-agent]
                                                                              |
                                                                              v
                                                               [ElevenLabs TTS]
                                                                              |
                                                                              v
                                                               [Respond to Webhook]
```

### Nodos de Routing por Origen (Switch-Origin)

El workflow detecta automaticamente desde que sitio proviene el mensaje y aplica instrucciones especificas.

| Nodo | Tipo | Funcion |
|------|------|---------|
| Classify-Source | n8n-nodes-base.set | Detecta origen: Web vs WhatsApp, extrae `sourceProject` |
| Switch-Origin | n8n-nodes-base.switch | Enruta segun `source.project` |
| Set-Msg-IbonLanding | n8n-nodes-base.set | Instrucciones para portal general |
| Set-Msg-RetoGluteos | n8n-nodes-base.set | Instrucciones para programa 28 dias |
| Set-Msg-PalacioFitness | n8n-nodes-base.set | Instrucciones para gimnasio |
| Set-Msg-ByIbon | n8n-nodes-base.set | Instrucciones para mentoria |
| Set-Msg-WhatsApp | n8n-nodes-base.set | Instrucciones para WhatsApp |
| Merge-Origins | n8n-nodes-base.merge | Combina todas las ramas |

### Instrucciones por Sitio

| Sitio | Enfoque | Tono |
|-------|---------|------|
| ibon-landing | Portal general, presenta 3 lineas de negocio | Acogedora, orientadora |
| reto-gluteos | Programa 28 dias para gluteos | Motivadora, enfocada en resultados |
| palacio-fitness | Gimnasio exclusivo para mujeres | Profesional, bienestar femenino |
| by-ibon | Mentoria y empoderamiento | Firme, inspiradora |
| whatsapp | Atencion directa | Conversacional, personal |

### Saludos y Despedidas Personalizados

Los nodos `Random Saludo` y `Random Despedida` seleccionan mensajes segun `sourceProject`:

```javascript
// Ejemplo de saludo para reto-gluteos
"¡Buenos días, María! Bienvenida al Reto Glúteos.
28 días para transformar tus glúteos con método y disciplina.
¿Lista para el reto?"
```

### Nodos Principales

| Nodo | Tipo | Funcion |
|------|------|---------|
| Webhook | n8n-nodes-base.webhook | Recibe POST en `/ibon-web` |
| AI Agent | @n8n/n8n-nodes-langchain.agent | GPT-4o con memoria |
| Text Classifier | @n8n/n8n-nodes-langchain.textClassifier | Clasifica: saludo/consulta/despedida |
| Simple Memory | @n8n/n8n-nodes-langchain.memoryBufferWindow | Memoria de conversacion |
| Convert text to speech | @elevenlabs/n8n-nodes-elevenlabs | TTS con ElevenLabs |

### Herramientas del Agente

| Tool | Tipo | Funcion |
|------|------|---------|
| check-availability | HTTP Request | Consulta horas disponibles para agendar cita |
| create-appointment | HTTP Request | Crea cita en BD + envia email confirmacion |
| mail-agent | Agent Tool | Envio de emails via Gmail |
| calendar-agent | Agent Tool | Gestion de Google Calendar |
| get-data-ibon | Google Sheets | Datos de Ibon Palacio |

### Agendamiento de Citas via Chat

La IA puede agendar citas directamente desde el chat usando los tools:

**Flujo:**
1. Usuario: "Quiero agendar una cita"
2. IA usa `check-availability` para consultar horas disponibles
3. IA solicita datos: nombre, correo, telefono, fecha, hora, motivo
4. IA usa `create-appointment` para crear la cita
5. Sistema envia email de confirmacion automatico con ICS

**APIs usadas:**
- `GET https://ibonpalacio.com/api/check-availability.php?fecha=YYYY-MM-DD`
- `POST https://ibonpalacio.com/api/send-email.php`

**Base de datos:** MySQL `jhtriqfo_ibondata` (tabla `citas`)

---

## Payload Estandar

Todos los chats envian el siguiente payload al webhook:

```json
{
  "event": "messages.upsert",
  "instance": "ByIbonWeb|RetoGluteosWeb|PalacioFitnessWeb|IbonLandingWeb",
  "origin": {
    "platform": "web",
    "project": "by-ibon|reto-gluteos|palacio-fitness|ibon-landing",
    "url": "https://by.ibonpalacio.com|https://retogluteos.ibonpalacio.com|https://fitness.ibonpalacio.com|https://ibonpalacio.com",
    "component": "home-chat|chat-bubble"
  },
  "data": {
    "key": {
      "remoteJid": "sessionId@web.ibon.net",
      "fromMe": false,
      "id": "msg_unique_id"
    },
    "pushName": "Nombre del usuario",
    "message": {
      "conversation": "texto del mensaje"
    },
    "messageType": "conversation|audio",
    "messageTimestamp": 1234567890
  },
  "date_time": "2026-01-13T12:00:00.000Z",
  "sender": "sessionId@web.ibon.net",
  "sessionId": "web_unique_session_id",
  "userInfo": {
    "nombre": "Nombre",
    "email": "email@example.com",
    "telefono": "3001234567",
    "audioEnabled": true
  }
}
```

### Campo `origin` - Identificacion del Origen

| Campo | Descripcion | Valores Posibles |
|-------|-------------|------------------|
| `platform` | Plataforma de origen | `web` |
| `project` | Nombre del proyecto | `ibon-landing`, `reto-gluteos`, `palacio-fitness`, `by-ibon` |
| `url` | URL de produccion | URLs completas de cada proyecto |
| `component` | Componente que envia | `chat-bubble`, `home-chat` |

### En n8n, acceder al origen:

```javascript
// En un nodo Code de n8n
const origin = $json.origin;
const project = origin.project;      // "by-ibon"
const component = origin.component;  // "home-chat"
const url = origin.url;              // "https://by.ibonpalacio.com"
```

---

## URLs de Produccion

| Proyecto | URL | Chat Bubble | Home Chat |
|----------|-----|-------------|-----------|
| ibon-landing | https://ibonpalacio.com | Si | No |
| reto-gluteos | https://retogluteos.ibonpalacio.com | Si | Si |
| palacio-fitness | https://fitness.ibonpalacio.com | Si | Si |
| by-ibon | https://by.ibonpalacio.com | Si | Si |

---

## Notas Tecnicas

### Diferencias entre Proyectos

| Aspecto | ibon-landing | reto-gluteos | palacio-fitness | by-ibon |
|---------|--------------|--------------|-----------------|---------|
| Angular | 17 | 17 | 11 | 17 |
| Iconos | Remixicon | Remixicon | FontAwesome | Remixicon |
| Estilos | SCSS+Tailwind | SCSS+Tailwind | CSS+Bootstrap | SCSS+Tailwind |
| Servicio Chat | ChatService | ChatService | ChatService | IbonChatService |

### Archivos de Configuracion

Cada proyecto tiene su servicio de chat en:
- Angular 17: `src/app/services/chat.service.ts` o `ibon-chat.service.ts`
- Angular 11: `src/app/services/chat.service.ts`

### Consideraciones de CORS

El webhook n8n tiene `allowedOrigins: "*"` configurado, permitiendo requests desde cualquier origen.

---

## Mantenimiento

### Para cambiar el webhook URL

1. Actualizar en cada proyecto el archivo de servicio correspondiente
2. La variable es `WEBHOOK_URL` o `webhookUrl`
3. Rebuild y deploy de cada proyecto

### Para agregar un nuevo proyecto

1. Copiar estructura de chat de un proyecto existente
2. Configurar el servicio con la URL del webhook
3. Asegurar que el payload siga el formato estandar

---

## Backups

Los backups del workflow se guardan en este repositorio:

| Archivo | Descripcion | Fecha |
|---------|-------------|-------|
| `ibon-web.json` | Version actual del workflow | Actualizado |
| `ibon-web-backup-2026-01-15-pre-scheduling.json` | Antes de agregar tools de agendamiento | 2026-01-15 |
| `ibon-web-backup-2026-01-15-switch-origin.json` | Version estable con Switch-Origin | 2026-01-15 |

### Restaurar un Backup

```bash
# 1. Copiar backup sobre el archivo principal
cp ibon-web-backup-FECHA.json ibon-web.json

# 2. Subir a n8n via API
curl -X PUT "https://flow.miclickderecho.com/api/v1/workflows/aw1Z0944wtQvvrm7" \
  -H "X-N8N-API-KEY: TU_API_KEY" \
  -H "Content-Type: application/json" \
  --data-binary @ibon-web.json
```

---

## Ultima Actualizacion

- **Fecha:** 2026-01-15
- **Autor:** Claude Code
- **Version:** 3.0
- **Cambios:**
  - v2.0: Implementacion de Switch-Origin con instrucciones y saludos por sitio
  - v3.0: Tools de agendamiento (check-availability, create-appointment) conectados a BD MySQL
