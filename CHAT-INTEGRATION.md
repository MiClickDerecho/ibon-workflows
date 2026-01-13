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
- **Actualizado:** 2026-01-13
- **Total Nodos:** 46

### Arquitectura del Workflow

```
[Webhook] --> [Switch-Data-Msg] --> [Text Classifier] --> [AI Agent]
                    |                                          |
                    v                                          v
             [Transcribe-Audio]                    [mail-agent, calendar-agent]
                    |                                          |
                    v                                          v
             [Set-Transcription]                   [Google Sheets Tools]
                                                              |
                                                              v
                                                   [ElevenLabs TTS]
                                                              |
                                                              v
                                                   [Respond to Webhook]
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
| mail-agent | Agent Tool | Envio de emails via Gmail |
| calendar-agent | Agent Tool | Gestion de Google Calendar |
| Get_consultoria | Google Sheets | Datos de consultoria |
| Get_soft_medida | Google Sheets | Datos de software a medida |
| Get_academias | Google Sheets | Datos de academias |
| Get_pos | Google Sheets | Datos de POS |
| Get_fact_electronica | Google Sheets | Datos de facturacion electronica |
| Get_politicos | Google Sheets | Datos de politicos |

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

## Ultima Actualizacion

- **Fecha:** 2026-01-13
- **Autor:** Claude Code
- **Version:** 1.0
