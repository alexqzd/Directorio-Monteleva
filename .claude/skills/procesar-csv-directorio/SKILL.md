---
name: procesar-csv-directorio
description: Use when Karen sends a new CSV export of the Directorio Monteleva form responses. Guides archiving the old CSV, diffing against the new one, and deciding what to update in directory.json.
---

# Procesar CSV del Directorio Monteleva

## Contexto

Karen Niño administra el formulario de registro del Directorio Monteleva y manda exportaciones CSV periódicamente. Cada CSV es una exportación **completa** de todas las respuestas (no solo las nuevas). El objetivo es compararlo contra el anterior para encontrar qué cambió.

Los archivos viven en:
- `data/current.csv` — el CSV completo más reciente procesado
- `data/archive/YYYY-MM-DD.csv` — histórico de CSVs anteriores
- `data/overrides.json` — cambios manuales que NO deben revertirse (bajas, fusiones)
- `directory.json` — el directorio como fuente de verdad

## Paso 1: Archivar y reemplazar

```bash
# Archivar el current con la fecha de hoy
cp data/current.csv data/archive/YYYY-MM-DD.csv

# Reemplazar con el nuevo CSV recibido de Karen
cp /ruta/al/nuevo.csv data/current.csv
```

## Paso 2: Ver el diff

```bash
git diff --no-index data/archive/YYYY-MM-DD.csv data/current.csv
```

Líneas con `+` = agregadas en el nuevo CSV. Líneas con `-` = desaparecieron.

## Paso 3: Analizar el diff

### Ruido — ignorar

- Sociales vacíos o negativos: `NA`, `no tengo`, `nop`, `No`, `-`, `—`, `sin redes`, `solo whatsapp`, saltos de línea vacíos
- Cambios de formato de teléfono sin cambio de número (ej. espacios, puntos extra)
- Reordenamiento de filas sin cambio de contenido
- Timestamps idénticos con mínimas diferencias de espaciado

### Cambios reales — actuar

| Tipo | Señal en el diff | Acción |
|------|-----------------|--------|
| Persona nueva | Fila `+` cuyo nombre no existe en `directory.json` | Agregar entrada |
| Servicio actualizado | Fila `+` con nombre conocido pero descripción diferente | Actualizar `service` |
| Teléfono nuevo | Número distinto al registrado | Actualizar `phone` |
| Social nuevo | Handle o URL donde antes había nada | Agregar `socialMode/Display/Url` |
| Descuento cambiado | `Sí` → `Por ahora, no.` o viceversa | Actualizar `discount` |
| Baja / persona eliminada | Fila `-` sin correspondiente `+` | Verificar con `overrides.json` antes de actuar |

### Duplicados — siempre verificar

Si el mismo nombre aparece más de una vez en el diff:
- Mismo teléfono → probablemente reenvío del formulario, tomar la más reciente
- Distinto teléfono → misma persona con dos números, fusionar (ver overrides.json)
- Mismo handle de redes → misma persona aunque el nombre cambie ligeramente

## Paso 4: Revisar overrides.json

Antes de hacer cualquier cambio, leer `data/overrides.json`. Si una persona aparece con `"action": "remove"`, NO reagregarla aunque aparezca en el nuevo CSV. Si hay un `"action": "merge"`, mantener la entrada fusionada.

## Paso 5: Actualizar directory.json

Para cada cambio real identificado:
1. Localizar la entrada en `directory.json` por nombre (búsqueda flexible)
2. Actualizar los campos que cambiaron
3. Recalcular el `id` si cambió `service` o `phone`
4. Confirmar con el usuario antes de hacer commit si hay dudas

## Paso 6: Commit y mensaje para Karen

Formato del commit:
```
Update directory: add N new entries, update M existing
```

Mensaje para Karen incluye:
- Nuevos agregados (nombre y servicio)
- Actualizados (qué campo cambió)
- Duplicados detectados y cómo se resolvieron
- Total de registros
