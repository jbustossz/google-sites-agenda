# Documentación Técnica de la API - Consulta de Citas

**Autor:** Juan Pablo Bustos Sáez

## 📋 Resumen Ejecutivo

Esta documentación proporciona toda la información necesaria para que desarrolladores y proveedores externos puedan consultar **todas las citas (excepto anuladas) por rango de fechas** mediante la API OData de RedSalud.

### Objetivo del Requerimiento
Consultar todas las citas (excepto las anuladas) dentro de un rango de fechas específico, con soporte completo de **paginación** para manejar grandes volúmenes de datos.

**Estados incluidos:** Booked (Agendado), Confirmed (Confirmado), CheckedIn (Presentado), ServicePerformed (Atendido), Blocked (Bloqueado), NotPerformed (No Presentado)

**Estados excluidos:** Cancelled (Anulado) - NO se incluyen en los resultados

---

## ⚠️ Estado Actual y Requisitos Críticos

### Estado Actual (NO DISPONIBLE PARA PROVEEDORES)

⚠️ **IMPORTANTE:** Estas rutas NO están disponibles para proveedores externos hasta completar los requisitos críticos.

- ⚠️ **Endpoint actual:** `https://proxy.megasalud.cl/ThirdPartyService/Appointments` (NO disponible para proveedores)
- ⚠️ **Estado:** Requiere enmascaramiento mediante Apigee y certificaciones de calidad
- ⚠️ **Autenticación:** Bearer Token (sistema actual - solo para uso interno)

### Requisitos Críticos para Disponibilización

**ANTES de disponibilizar estas rutas para consumo de integraciones o proveedores externos, se DEBEN cumplir:**

1. **Enmascaramiento mediante Apigee** ⚠️ **CRÍTICO**
   - Las rutas DEBEN ser enmascaradas por Apigee
   - El endpoint actual NO debe exponerse directamente a proveedores externos
   - El Apigee actúa como capa de seguridad y control
   - **Esfuerzo estimado:** 1 sprint (Pedro Wittig)

2. **Certificaciones de Calidad Completas** ⚠️ **CRÍTICO**
   - Pasar TODAS las certificaciones de calidad
   - Incluye: pruebas de estrés, seguridad, performance, validaciones funcionales
   - No se permite disponibilizar sin completar todas las certificaciones

3. **Ruta Segura con Credenciales Diferenciadas**
   - Sistema de autenticación separado para proveedores
   - Rate limiting específico
   - Mejor auditoría y logging

**Responsable:** Pedro Wittig  
**Esfuerzo estimado:** 1 sprint para enmascaramiento mediante Apigee

**Nota:** Una vez implementado el Apigee y completadas todas las certificaciones, esta documentación será actualizada con las nuevas rutas y credenciales.

---

## 🔑 Información de Autenticación

### Headers Requeridos

```bash
Authorization: Bearer {TU_TOKEN}
X-AppTimezone: -240
Accept: application/json
```

### Obtener Credenciales
Contactar al equipo de integración de RedSalud para obtener un token Bearer válido.

---

## 📊 Endpoint Principal

### Base URL
```
https://proxy.megasalud.cl/ThirdPartyService
```

### Endpoint de Citas
```
GET /ThirdPartyService/Appointments
```

### Endpoint de Metadata
```
GET /ThirdPartyService/$metadata
```
Proporciona la definición completa del modelo de datos OData.

---

## 🔍 Parámetros OData

### $filter (Filtros)

**Sintaxis para filtros de fecha:**
```
DateTimeFrom ge {fecha_inicio} and DateTimeTo le {fecha_fin}
```

**Sintaxis para filtros de estado:**
```
Status eq WebApiModel.Enum.AppointmentStatus'Confirmed'
```

**Operadores disponibles:**
- `eq` = igual a
- `ne` = diferente de
- `ge` = mayor o igual que
- `le` = menor o igual que
- `gt` = mayor que
- `lt` = menor que
- `and` = operador lógico AND
- `or` = operador lógico OR

### $orderby (Ordenamiento)

**⚠️ NOTA:** El parámetro `$orderby` no está disponible en este endpoint. Los resultados se devuelven en el orden predeterminado del sistema.

### $top (Límite de resultados) ⚠️ OBLIGATORIO
```
$top=500
```
**IMPORTANTE:** El valor máximo permitido es **500**. Siempre usar `$top=500` en todas las consultas.

### $skip (Offset para paginación) ⚠️ OBLIGATORIO
```
$skip=0    # Primera página
$skip=500  # Segunda página
$skip=1000 # Tercera página
```

**Fórmula:** `$skip = (número_página - 1) * 500`

### $select (Seleccionar campos específicos)
```
$select=Id,DateTimeFrom,DateTimeTo,Status,PatientId
```

### $expand (Expandir relaciones)
```
$expand=Patient
$expand=Patient,ResourceAppointments($expand=Resource)
```

---

## 📅 Estados de Citas

| Valor en API | Descripción en Español | Valor Numérico | Descripción |
|--------------|------------------------|---------------|-------------|
| `Booked` | **Agendado** | 0 | Citas que han sido agendadas pero aún no confirmadas |
| `Confirmed` | **Confirmado** | 1 | Citas que han sido confirmadas |
| `CheckedIn` | **Presentado** | 3 | Citas donde el paciente se ha presentado (check-in realizado) |
| `ServicePerformed` | **Atendido** | 4 | Citas donde el servicio ya fue realizado/atendido |
| `Blocked` | **Bloqueado** | 5 | Citas que están bloqueadas |
| `NotPerformed` | **No Presentado** | 6 | Citas donde el paciente no se presentó |
| `Cancelled` | **Anulado** | 2 | Citas que han sido anuladas (excluidas del requerimiento) |

### 📊 Estadísticas de Estados (Datos Reales)

**Estudio basado en los últimos 6 meses de datos del sistema (junio a diciembre).**

**Para ver el detalle completo de números de estado por mes, consultar:**
- [📊 Detalle de Estadísticas de Estados (Google Sheets)](https://docs.google.com/spreadsheets/d/1UgmU2IRTFhFHY03BcmEacrk6BFTw3vaegPxdQAK-o5E/edit?gid=0#gid=0)
- [📊 Estudio BBDD Estados de Citas (SQL)](https://drive.google.com/file/d/1uSQo4ZSk7OTSlgKNv6qeuUHZv7SZkspM/view?usp=sharing)

Estadísticas basadas en datos reales del sistema:

| Mes | Estado | Cantidad (Hits) | Porcentaje |
|-----|--------|-----------------|------------|
| **Junio** | Agendado | 23,572 | 3.19% |
| | Anulado | 245,795 | 33.27% |
| | Atendido | 366,317 | 49.58% |
| | Bloqueado | 749 | 0.10% |
| | Confirmado | 29,961 | 4.05% |
| | No Presentado | 25,337 | 3.43% |
| | Presentado | 47,153 | 6.38% |
| **Julio** | Agendado | 54,994 | 3.23% |
| | Anulado | 560,683 | 32.94% |
| | Atendido | 845,677 | 49.68% |
| | Bloqueado | 2,184 | 0.13% |
| | Confirmado | 67,227 | 3.95% |
| | No Presentado | 57,429 | 3.37% |
| | Presentado | 114,070 | 6.70% |
| **Agosto** | Agendado | 61,885 | 3.82% |
| | Anulado | 528,550 | 32.59% |
| | Atendido | 800,369 | 49.35% |
| | Bloqueado | 2,420 | 0.15% |
| | Confirmado | 58,715 | 3.62% |
| | No Presentado | 60,751 | 3.75% |
| | Presentado | 109,002 | 6.72% |
| **Septiembre** | Agendado | 52,237 | 3.46% |
| | Anulado | 498,134 | 32.96% |
| | Atendido | 757,012 | 50.08% |
| | Bloqueado | 2,625 | 0.17% |
| | Confirmado | 51,828 | 3.43% |
| | No Presentado | 54,868 | 3.63% |
| | Presentado | 94,804 | 6.27% |
| **Octubre** | Agendado | 61,280 | 3.51% |
| | Anulado | 549,898 | 31.51% |
| | Atendido | 904,362 | 51.81% |
| | Bloqueado | 2,038 | 0.12% |
| | Confirmado | 62,051 | 3.56% |
| | No Presentado | 63,396 | 3.63% |
| | Presentado | 102,384 | 5.87% |
| **Noviembre** | Agendado | 54,768 | 3.47% |
| | Anulado | 504,172 | 31.97% |
| | Atendido | 822,348 | 52.14% |
| | Bloqueado | 2,121 | 0.13% |
| | Confirmado | 55,273 | 3.50% |
| | No Presentado | 56,461 | 3.58% |
| | Presentado | 82,109 | 5.21% |
| **Diciembre** | Agendado | 29,179 | 3.68% |
| | Anulado | 252,350 | 31.81% |
| | Atendido | 411,506 | 51.87% |
| | Bloqueado | 1,114 | 0.14% |
| | Confirmado | 28,411 | 3.58% |
| | No Presentado | 27,568 | 3.47% |
| | Presentado | 43,272 | 5.45% |

### 📈 Análisis de Distribución de Estados

**Promedio de distribución (últimos 6 meses - junio a diciembre):**

- **Atendido (ServicePerformed):** ~50% - Estado más común
- **Anulado (Cancelled):** ~32% - Segundo estado más común (excluido del requerimiento)
- **Presentado (CheckedIn):** ~6% 
- **Agendado (Booked):** ~3.5%
- **Confirmado (Confirmed):** ~3.6%
- **No Presentado (NotPerformed):** ~3.5%
- **Bloqueado (Blocked):** ~0.13% - Estado menos común

**Nota:** Los estados **Anulado (Cancelled)** representan aproximadamente el 32% de las citas pero están **excluidos del requerimiento**, por lo que las consultas deben filtrar estos registros.

### Sintaxis para Filtros de Estado

**Consultar por estado específico:**
```
Status eq WebApiModel.Enum.AppointmentStatus'Booked'        # Solo Agendado
Status eq WebApiModel.Enum.AppointmentStatus'Confirmed'     # Solo Confirmado
Status eq WebApiModel.Enum.AppointmentStatus'CheckedIn'     # Solo Presentado
Status eq WebApiModel.Enum.AppointmentStatus'ServicePerformed'  # Solo Atendido
Status eq WebApiModel.Enum.AppointmentStatus'Blocked'       # Solo Bloqueado
Status eq WebApiModel.Enum.AppointmentStatus'NotPerformed'  # Solo No Presentado
```

**Operadores:**
- `eq` = "equal" (igual a) - Para filtrar por un estado específico
- `ne` = "not equal" (diferente de) - Para excluir un estado

---

## 💻 Ejemplos de Uso

**⚠️ IMPORTANTE - Formato de URL:**
Todos los ejemplos usan el formato verificado que funciona correctamente:
- Usar `$` directamente (no `%24`) para parámetros OData: `$filter`, `$top`, `$skip`
- Usar URL encoding para espacios: `%20`
- Usar URL encoding para dos puntos: `%3A` 
- Usar URL encoding para comillas simples: `%27`
- **Nota:** El parámetro `$orderby` no está disponible en este endpoint

### Ejemplo 1: Obtener Todas las Citas (Sin Filtro)

```bash
curl --location 'https://proxy.megasalud.cl/ThirdPartyService/Appointments?$top=500&$skip=0' \
--header 'Authorization: Bearer {TU_TOKEN}' \
--header 'X-AppTimezone: -240' \
--header 'Accept: application/json'
```

**Resultado:** Obtiene todas las citas de todos los estados.

### Ejemplo 2: Obtener Solo Citas Confirmadas

```bash
curl --location 'https://proxy.megasalud.cl/ThirdPartyService/Appointments?$filter=Status%20eq%20WebApiModel.Enum.AppointmentStatus%27Confirmed%27&$top=500&$skip=0' \
--header 'Authorization: Bearer {TU_TOKEN}' \
--header 'X-AppTimezone: -240' \
--header 'Accept: application/json'
```

### Ejemplo 3: Obtener Citas por Rango de Fechas y Estado (FORMATO VERIFICADO)

**⚠️ Este es el formato que funciona correctamente:**

```bash
curl --location 'https://proxy.megasalud.cl/ThirdPartyService/Appointments?$filter=DateTimeFrom%20ge%202025-12-15T21%3A48%3A36-03%3A00%20and%20DateTimeTo%20le%202026-02-13T21%3A48%3A36-03%3A00%20and%20Status%20eq%20WebApiModel.Enum.AppointmentStatus%27Confirmed%27&$top=500&$skip=0' \
--header 'Authorization: Bearer {TU_TOKEN}' \
--header 'X-AppTimezone: -240' \
--header 'Accept: application/json'
```

**URL decodificada (para referencia):**
```
https://proxy.megasalud.cl/ThirdPartyService/Appointments?
  $filter=DateTimeFrom ge 2025-12-15T21:48:36-03:00 
    and DateTimeTo le 2026-02-13T21:48:36-03:00 
    and Status eq WebApiModel.Enum.AppointmentStatus'Confirmed'
  &$top=500
  &$skip=0
```

**⚠️ IMPORTANTE - Formato de URL:**
- Usar `$` directamente (no `%24`) para parámetros OData: `$filter`, `$top`, `$skip`
- Usar URL encoding para espacios: `%20`
- Usar URL encoding para dos puntos: `%3A`
- Usar URL encoding para comillas simples: `%27`

### Ejemplo 4: Paginación - Primera Página

```bash
curl --location 'https://proxy.megasalud.cl/ThirdPartyService/Appointments?$filter=DateTimeFrom%20ge%202025-12-15T21%3A48%3A36-03%3A00%20and%20DateTimeTo%20le%202026-02-13T21%3A48%3A36-03%3A00%20and%20Status%20eq%20WebApiModel.Enum.AppointmentStatus%27Confirmed%27&$top=500&$skip=0' \
--header 'Authorization: Bearer {TU_TOKEN}' \
--header 'X-AppTimezone: -240' \
--header 'Accept: application/json'
```

### Ejemplo 5: Paginación - Segunda Página

```bash
curl --location 'https://proxy.megasalud.cl/ThirdPartyService/Appointments?$filter=DateTimeFrom%20ge%202025-12-15T21%3A48%3A36-03%3A00%20and%20DateTimeTo%20le%202026-02-13T21%3A48%3A36-03%3A00%20and%20Status%20eq%20WebApiModel.Enum.AppointmentStatus%27Confirmed%27&$top=500&$skip=500' \
--header 'Authorization: Bearer {TU_TOKEN}' \
--header 'X-AppTimezone: -240' \
--header 'Accept: application/json'
```

### Ejemplo 6: Seleccionar Solo Campos Específicos

```bash
curl --location 'https://proxy.megasalud.cl/ThirdPartyService/Appointments?$select=Id,DateTimeFrom,DateTimeTo,Status,PatientId&$top=500&$skip=0' \
--header 'Authorization: Bearer {TU_TOKEN}' \
--header 'X-AppTimezone: -240' \
--header 'Accept: application/json'
```

---

## 📄 Paginación Obligatoria

### ⚠️ Requisitos de Paginación

1. **$top SIEMPRE debe ser 500** (máximo permitido)
2. **$skip es obligatorio** para navegar entre páginas
3. **No se permite realizar consultas sin paginación**

### Fórmula de Paginación

- **Página 1:** `$skip=0`, `$top=500`
- **Página 2:** `$skip=500`, `$top=500`
- **Página 3:** `$skip=1000`, `$top=500`
- **Página N:** `$skip=(N-1)*500`, `$top=500`

### Detectar Última Página

Cuando la respuesta contiene menos de 500 resultados, es la última página.

---

## 📅 Formato de Fechas

### Formato Requerido
- **Formato ISO 8601 con timezone:** `YYYY-MM-DDTHH:mm:ss-TZ:tz`
- **Ejemplo:** `2025-12-15T21:48:36-03:00`

### Zona Horaria
- **Header requerido:** `X-AppTimezone: -240` (UTC-4 = -240 minutos)
- **Otras zonas comunes:**
  - UTC-3: `-180`
  - UTC-5: `-300`

---

## ⚠️ Consideraciones Técnicas Obligatorias

**ANTES DE IMPLEMENTAR EN PRODUCCIÓN:**

1. **Enmascaramiento mediante Apigee** ⚠️ **CRÍTICO**
   - Las rutas DEBEN ser enmascaradas por Apigee antes de disponibilizar
   - **Esfuerzo estimado:** 1 sprint (Pedro Wittig)
   - Contactar a Pedro Wittig para coordinación

2. **Certificaciones de Calidad Completas** ⚠️ **CRÍTICO**
   - Pasar TODAS las certificaciones de calidad
   - Incluye: pruebas de estrés, seguridad, performance
   - Contactar al equipo de operaciones

3. **Pruebas de Estrés en QA**
   - El proceso DEBE pasar pruebas de calidad de estrés en ambiente QA antes de producción
   - Contactar al equipo de operaciones para coordinar

4. **Ventana de Ejecución: Después de las 03:00 AM**
   - **OBLIGATORIO:** Las consultas DEBEN ejecutarse después de las 03:00 AM (horario local)
   - Las consultas ejecutadas fuera de esta ventana pueden ser rechazadas o limitadas

5. **Paginación Obligatoria con $top=500**
   - **OBLIGATORIO:** Todas las consultas DEBEN ser paginadas
   - El parámetro `$top` **SIEMPRE** debe ser `500` (máximo permitido)

6. **Confirmación con Operaciones**
   - Verificar si hay definiciones adicionales a considerar
   - Contactar al equipo de operaciones antes de iniciar la integración en producción

---

## 🔧 Manejo de Errores

### Códigos HTTP Comunes

- `200 OK` - Solicitud exitosa
- `400 Bad Request` - Solicitud inválida (sintaxis OData incorrecta, validación fallida)
- `401 Unauthorized` - Token de autenticación inválido o faltante
- `403 Forbidden` - No tiene permisos para acceder al recurso
- `404 Not Found` - Recurso no encontrado
- `500 Internal Server Error` - Error interno del servidor
- `503 Service Unavailable` - Servicio temporalmente no disponible

### Formato de Errores OData

```json
{
  "error": {
    "code": "ErrorCode",
    "message": "Mensaje de error descriptivo",
    "target": "Propiedad o recurso que causó el error"
  }
}
```

### Errores Comunes y Soluciones

**Error:** `A binary operator with incompatible types was detected`
- **Causa:** Comparación de tipos incompatibles (ej: enum con string)
- **Solución:** Usar sintaxis correcta para enums: `Status eq WebApiModel.Enum.AppointmentStatus'Confirmed'`

**Error:** `Could not find a property named 'X'`
- **Causa:** Propiedad no existe o nombre incorrecto
- **Solución:** Verificar en `$metadata` el nombre correcto de la propiedad

**Error:** `The query specified in the URI is not valid`
- **Causa:** Sintaxis OData incorrecta
- **Solución:** Validar sintaxis según especificación OData v4

---

## 📋 Propiedades Completas de AppointmentDTO

### Propiedades Principales

| Propiedad | Tipo | Descripción | Ejemplo |
|-----------|------|-------------|---------|
| `Id` | Guid | Identificador único de la cita | `"7b21fa47-f65b-4385-b82d-a61e00f0ec87"` |
| `DateTimeFrom` | DateTimeOffset | Fecha y hora de inicio de la cita (zona horaria local) | `"2016-06-07T17:00:00-03:00"` |
| `DateTimeTo` | DateTimeOffset | Fecha y hora de fin de la cita (zona horaria local) | `"2016-06-07T17:12:00-03:00"` |
| `DateTimeFromUTC` | DateTimeOffset | Fecha y hora de inicio de la cita en UTC | `"2016-06-07T20:00:00Z"` |
| `DateTimeToUTC` | DateTimeOffset | Fecha y hora de fin de la cita en UTC | `"2016-06-07T20:12:00Z"` |
| `Status` | AppointmentStatus (enum) | Estado de la cita (Booked, Confirmed, CheckedIn, ServicePerformed, Blocked, NotPerformed, Cancelled) | `"Booked"` |
| `Duration` | int | Duración de la cita en minutos | `12` |
| `PatientId` | Guid | Identificador único del paciente | `"d714a336-e7ad-4c55-92bf-a61e00e25ae1"` |
| `ResourceId` | Guid | Identificador único del recurso (médico/profesional) | `"d19eb039-180b-4f16-a02a-a5fa002a53ac"` |
| `ServiceId` | Guid | Identificador único del servicio | `"b37110ae-09cc-4432-94e5-a5f8004ed15c"` |
| `CenterId` | Guid | Identificador único del centro médico | `"4d4c071b-05b5-4773-8a55-a5f8002127fe"` |
| `CoveragePlanId` | Guid | Identificador del plan de cobertura | `"f2d3cd94-b91c-420e-9ec0-a5f800ca8dbc"` |
| `AppointmentTypeId` | Guid | Identificador del tipo de cita | `"4139651e-c3c2-4839-97ef-cdd5428b9975"` |

### Propiedades de Control y Estado

| Propiedad | Tipo | Descripción | Ejemplo |
|-----------|------|-------------|---------|
| `Cancelled` | bool | Indica si la cita está cancelada/anulada | `false` |
| `IsOvercapacity` | bool | Indica si la cita está en sobrecapacidad | `false` |
| `Archived` | bool | Indica si la cita está archivada | `true` |
| `Index` | int | Índice de la cita (usado para ordenamiento) | `1` |
| `Number` | string (nullable) | Número de la cita (puede ser null) | `null` |
| `NumberOfSlots` | int | Número de slots que ocupa la cita | `1` |

### Propiedades de Relaciones y Referencias

| Propiedad | Tipo | Descripción | Ejemplo |
|-----------|------|-------------|---------|
| `RescheduleAppointmentId` | Guid (nullable) | ID de la cita original si esta es una reagendación | `null` |
| `OldPatientId` | Guid (nullable) | ID del paciente anterior (si hubo cambio) | `null` |
| `TreatmentPlanId` | Guid (nullable) | ID del plan de tratamiento asociado | `null` |
| `BlockedByAvailabilityId` | Guid (nullable) | ID de la disponibilidad que bloqueó la cita | `null` |
| `BlockedByOriginalAvailabilityId` | Guid (nullable) | ID de la disponibilidad original que bloqueó | `null` |
| `CreatedByMainAvailabilityId` | Guid (nullable) | ID de la disponibilidad principal que creó la cita | `null` |
| `CreatedByOriginalAvailabilityId` | Guid (nullable) | ID de la disponibilidad original que creó la cita | `null` |
| `OversellingDefinitionId` | Guid (nullable) | ID de la definición de sobreventa aplicada | `null` |
| `SortQueryId` | Guid (nullable) | ID de la consulta de ordenamiento utilizada | `null` |
| `CreatedByMarketingCampaign` | string (nullable) | Campaña de marketing que originó la cita | `null` |

### Propiedades de Comentarios y Datos Adicionales

| Propiedad | Tipo | Descripción | Ejemplo |
|-----------|------|-------------|---------|
| `Comments` | Collection | Colección de comentarios asociados a la cita | `[]` |
| `CommentsCount` | int | Número total de comentarios | `0` |
| `DynamicData` | Collection | Datos dinámicos adicionales asociados a la cita | `[]` |

### Propiedades de Auditoría y Control

| Propiedad | Tipo | Descripción | Ejemplo |
|-----------|------|-------------|---------|
| `CreatedOn` | DateTimeOffset | Fecha y hora de creación de la cita (UTC) | `"2016-06-07T14:37:10.5696009Z"` |
| `ModifiedOn` | DateTimeOffset | Fecha y hora de última modificación (UTC) | `"2016-06-07T14:37:10.6335273Z"` |
| `CreatedBy` | Guid | ID del usuario que creó la cita | `"ae2a5eda-87b6-467c-85d1-a6040066d4e1"` |
| `ModifiedBy` | Guid | ID del usuario que modificó la cita por última vez | `"ae2a5eda-87b6-467c-85d1-a6040066d4e1"` |
| `CreatedByUser` | string | Nombre del usuario que creó la cita | `"CallcenterMega"` |
| `ModifiedByUser` | string | Nombre del usuario que modificó la cita | `"CallcenterMega"` |
| `CreatedByClient` | string | Cliente/aplicación que creó la cita | `"CallCenter"` |
| `ModifiedByClient` | string | Cliente/aplicación que modificó la cita | `"CallCenter"` |
| `RowVersion` | string | Versión de la fila para control de concurrencia optimista | `"AAAAABjLZc4="` |

### Propiedades de Llamadas

| Propiedad | Tipo | Descripción | Ejemplo |
|-----------|------|-------------|---------|
| `InCallBy` | Guid (nullable) | ID del usuario que está en llamada con esta cita | `null` |

### Propiedades OData

| Propiedad | Tipo | Descripción | Ejemplo |
|-----------|------|-------------|---------|
| `@odata.etag` | string | ETag para control de concurrencia (formato OData) | `"W/\"J0FBQUFBQmpMWmM0PSc=\""` |

### Notas Importantes

- **Campos Nullable:** Muchos campos pueden ser `null`, lo que indica que no tienen valor asignado
- **Fechas:** `DateTimeFrom`/`DateTimeTo` están en zona horaria local, mientras que `DateTimeFromUTC`/`DateTimeToUTC` están en UTC
- **Status:** El campo `Status` es un enum que puede tener los valores: Booked, Confirmed, CheckedIn, ServicePerformed, Blocked, NotPerformed, Cancelled
- **RowVersion y ETag:** Ambos se usan para control de concurrencia optimista
- **Relaciones Expandibles:** Algunas relaciones pueden expandirse usando `$expand`:
  - `Patient` - Información del paciente
  - `ResourceAppointments` - Relación con recursos (médicos)
  - `ResourceAppointments/Resource` - Recurso expandido

Para ver todas las propiedades disponibles y sus tipos exactos, consultar el endpoint `$metadata`:
```
GET https://proxy.megasalud.cl/ThirdPartyService/$metadata
```

---

## 🚀 Inicio Rápido

### Paso 1: Obtener Credenciales
Contactar al equipo de integración para obtener un token Bearer.

### Paso 2: Realizar Primera Consulta

**⚠️ IMPORTANTE:** Usar este formato exacto que está verificado y funciona:

```bash
curl --location 'https://proxy.megasalud.cl/ThirdPartyService/Appointments?$filter=DateTimeFrom%20ge%202025-12-15T21%3A48%3A36-03%3A00%20and%20DateTimeTo%20le%202026-02-13T21%3A48%3A36-03%3A00%20and%20Status%20eq%20WebApiModel.Enum.AppointmentStatus%27Confirmed%27&$top=500&$skip=0' \
--header 'Authorization: Bearer {TU_TOKEN}' \
--header 'X-AppTimezone: -240' \
--header 'Accept: application/json'
```

**Nota:** En la URL, usar `$` directamente (no `%24`) para `$filter`, `$top`, `$skip`. El resto de caracteres especiales deben estar URL-encoded (`%20` para espacios, `%3A` para `:`, `%27` para comillas simples). **El parámetro `$orderby` no está disponible en este endpoint.**

### Paso 3: Implementar Paginación
Usar los ejemplos proporcionados en la sección "Paginación Obligatoria" para implementar la paginación completa.

---

## 📞 Soporte

### Para Obtener Credenciales
Contactar al equipo de integración de RedSalud.

### Para Reportar Problemas
- Revisar sección de "Manejo de Errores"
- Contactar al equipo técnico

### Para Consultas Técnicas
- **Responsable de integración:** Pedro Wittig
- **Equipo técnico:** Contactar a través del equipo de integración de RedSalud

---

## 📝 Changelog

| Fecha | Versión | Cambios |
|-------|---------|---------|
| 2025-01-XX | 1.0 | Documentación inicial consolidada |
| TBD | 2.0 | Actualización con ruta segura del Apigee (Pedro Wittig) |

---

## 🔗 Enlaces Útiles

### Documentación y Referencias
- **Metadata OData:** https://proxy.megasalud.cl/ThirdPartyService/$metadata
- **Especificación OData v4:** https://www.odata.org/documentation/

### Recursos Adicionales
- **📊 Estudio BBDD Estados de Citas (SQL):** [Archivo SQL - Estudio BBDD Estados de Citas](https://drive.google.com/file/d/1uSQo4ZSk7OTSlgKNv6qeuUHZv7SZkspM/view?usp=sharing)
- **📮 Colección Postman:** [Colección Postman - Consulta Principal](https://drive.google.com/file/d/1zVc_p5LkOZSmSZFHVW6BaewkYct7A3Qh/view?usp=sharing)

---

**Última actualización:** 2025-01-XX  
**Versión:** 1.0  
**Autor:** Juan Pablo Bustos Sáez  
**Mantenido por:** Equipo de Integración RedSalud
