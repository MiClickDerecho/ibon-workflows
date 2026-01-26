# Migración de Instrucciones del Chat a Base de Datos

## Fecha: 2026-01-25
## Backup: `ibon-web-backup-2026-01-25-pre-db-instructions.json`

---

## Resumen

Migrar las instrucciones del chat (actualmente hardcoded en los nodos Set-Msg-*) a la base de datos MySQL, permitiendo editarlas desde el CRM.

## API Endpoint

**URL:** `https://oficina.ibonpalacio.com/api/chat-instructions/public-list.php`
**Método:** GET
**Autenticación:** No requerida (endpoint público)
**Parámetro:** `origen` (requerido)

**Valores de origen:**
- `ibon-landing`
- `reto-gluteos`
- `palacio-fitness`
- `by-ibon`
- `whatsapp`

**Ejemplo de respuesta:**
```json
{
  "success": true,
  "origen": "palacio-fitness",
  "instructions": {
    "instr_personalidad": "...",
    "instr_negocio": "...",
    "instr_situaciones": "...",
    "instr_protocolo": "...",
    "instr_herramientas": "..."
  }
}
```

---

## Opción A: Modificación Simple (Recomendada)

### Paso 1: Agregar nodos HTTP Request

Para cada origen, agregar un nodo HTTP Request **ANTES** del nodo Set-Msg correspondiente:

| Origen | Nuevo nodo | Conectar antes de |
|--------|------------|-------------------|
| ibon-landing | HTTP-Instr-IbonLanding | Set-Msg-IbonLanding |
| reto-gluteos | HTTP-Instr-RetoGluteos | Set-Msg-RetoGluteos |
| palacio-fitness | HTTP-Instr-PalacioFitness | Set-Msg-PalacioFitness |
| by-ibon | HTTP-Instr-ByIbon | Set-Msg-ByIbon |
| whatsapp | HTTP-Instr-WhatsApp | Set-Msg-WhatsApp |

### Paso 2: Configurar HTTP Request

Para cada nodo HTTP Request:

**Method:** GET
**URL:** `https://oficina.ibonpalacio.com/api/chat-instructions/public-list.php?origen=ORIGEN`

(Reemplazar ORIGEN con el valor correspondiente: `palacio-fitness`, `ibon-landing`, etc.)

**Authentication:** None
**Response Format:** JSON

### Paso 3: Modificar Set-Msg nodes

En cada Set-Msg, cambiar los valores de las instrucciones de texto hardcoded a expresiones que referencien el nodo HTTP:

| Campo | Valor Anterior | Nuevo Valor |
|-------|----------------|-------------|
| instr_personalidad | (texto largo) | `={{ $('HTTP-Instr-PalacioFitness').item.json.instructions.instr_personalidad \|\| '' }}` |
| instr_negocio | (texto largo) | `={{ $('HTTP-Instr-PalacioFitness').item.json.instructions.instr_negocio \|\| '' }}` |
| instr_situaciones | (texto largo) | `={{ $('HTTP-Instr-PalacioFitness').item.json.instructions.instr_situaciones \|\| '' }}` |
| instr_protocolo | (texto largo) | `={{ $('HTTP-Instr-PalacioFitness').item.json.instructions.instr_protocolo \|\| '' }}` |
| instr_herramientas | (texto largo) | `={{ $('HTTP-Instr-PalacioFitness').item.json.instructions.instr_herramientas \|\| '' }}` |

**Nota:** Cambiar `HTTP-Instr-PalacioFitness` por el nombre del nodo HTTP correspondiente a cada origen.

---

## Opción B: Modificación Centralizada

### Arquitectura Alternativa

1. Crear un nodo Code después de Classify-Source que determine el origen
2. Agregar un único nodo HTTP Request que use el origen dinámico
3. Modificar los Set-Msg para leer del HTTP Request

### Nodo HTTP Request Único

**Name:** HTTP-Get-Instructions
**Method:** GET
**URL:** `https://oficina.ibonpalacio.com/api/chat-instructions/public-list.php?origen={{ $json.source.project }}`

Luego en los Set-Msg:
```
={{ $('HTTP-Get-Instructions').item.json.instructions.instr_personalidad || '' }}
```

---

## Verificación

1. Después de hacer los cambios, probar el chat desde cada origen
2. Verificar que las instrucciones se cargan correctamente
3. Modificar una instrucción desde el CRM y verificar que el chat la usa

---

## Rollback

Si algo falla, restaurar el backup:

```bash
cp ibon-web-backup-2026-01-25-pre-db-instructions.json ibon-web.json
```

Y subir a n8n via la interfaz web (importar workflow).

---

## Próximos Pasos

1. Cargar las instrucciones actuales en la BD desde el CRM
2. Implementar los cambios en n8n
3. Probar en todos los orígenes
4. Documentar los cambios
