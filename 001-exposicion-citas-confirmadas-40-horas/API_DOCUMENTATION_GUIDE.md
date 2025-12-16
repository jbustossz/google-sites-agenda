# Guía de Documentación Técnica de la API

**Autor:** Juan Pablo Bustos Sáez

Esta guía describe qué documentación técnica es necesaria para que cualquier desarrollador pueda utilizar la API OData de manera efectiva.

## Ejemplo Completo de Uso

### Ejemplo 1: Obtener Todas las Citas

Este ejemplo muestra cómo obtener todas las citas sin filtro de estado:

```bash
curl --location 'https://proxy.megasalud.cl/ThirdPartyService/Appointments?%24orderby=DateTimeFrom%20asc&%24top=500&%24skip=0' \
--header 'Authorization: Bearer {TU_TOKEN}' \
--header 'X-AppTimezone: -240' \
--header 'Accept: application/json, text/plain, */*'
```

**URL decodificada:**
```
https://proxy.megasalud.cl/ThirdPartyService/Appointments?$orderby=DateTimeFrom asc&$top=500&$skip=0
```

**Parámetros:**
- `$orderby=DateTimeFrom asc` - Ordenar por fecha de inicio ascendente
- `$top=500` - Máximo 500 resultados por página (OBLIGATORIO)
- `$skip=0` - Offset para paginación (OBLIGATORIO)

**Resultado:** Obtiene todas las citas de todos los estados.

**Nota:** Según el requerimiento, se deben excluir las citas anuladas. Ver Ejemplo 2 para filtrar por estado específico.

---

### Ejemplo 2: Obtener Solo Citas Confirmadas

Este ejemplo muestra cómo obtener solo las citas con estado Confirmado:

```bash
curl --location 'https://proxy.megasalud.cl/ThirdPartyService/Appointments?%24filter=Status%20eq%20WebApiModel.Enum.AppointmentStatus%27Confirmed%27&%24orderby=DateTimeFrom%20asc&%24top=500&%24skip=0' \
--header 'Authorization: Bearer {TU_TOKEN}' \
--header 'X-AppTimezone: -240' \
--header 'Accept: application/json, text/plain, */*'
```

**URL decodificada:**
```
https://proxy.megasalud.cl/ThirdPartyService/Appointments?$filter=Status eq WebApiModel.Enum.AppointmentStatus'Confirmed'&$orderby=DateTimeFrom asc&$top=500&$skip=0
```

**Parámetros:**
- `$filter=Status eq WebApiModel.Enum.AppointmentStatus'Confirmed'` - Filtro por estado Confirmado
- `$orderby=DateTimeFrom asc` - Ordenar por fecha de inicio ascendente
- `$top=500` - Máximo 500 resultados por página (OBLIGATORIO)
- `$skip=0` - Offset para paginación (OBLIGATORIO)

**Resultado:** Obtiene solo las citas con estado Confirmado.

**Explicación del filtro:**
- `eq` = "equal" (igual a)
- `Status eq Confirmed` = "Status igual a Confirmed (Confirmado)"
- **Esto INCLUYE solo las citas confirmadas**

---

### Ejemplo 3: Obtener Citas por Rango de Fechas y Estado Presentado

Este ejemplo combina filtro de rango de fechas y estado específico:

```bash
curl --location 'https://proxy.megasalud.cl/ThirdPartyService/Appointments?%24filter=DateTimeFrom%20ge%202025-12-15T21%3A48%3A36-03%3A00%20and%20DateTimeTo%20le%202026-02-13T21%3A48%3A36-03%3A00%20and%20Status%20eq%20WebApiModel.Enum.AppointmentStatus%27CheckedIn%27&%24orderby=DateTimeFrom%20asc&%24top=500&%24skip=0' \
--header 'Authorization: Bearer {TU_TOKEN}' \
--header 'X-AppTimezone: -240' \
--header 'Accept: application/json, text/plain, */*'
```

**URL decodificada:**
```
https://proxy.megasalud.cl/ThirdPartyService/Appointments?$filter=DateTimeFrom ge 2025-12-15T21:48:36-03:00 and DateTimeTo le 2026-02-13T21:48:36-03:00 and Status eq WebApiModel.Enum.AppointmentStatus'CheckedIn'&$orderby=DateTimeFrom asc&$top=500&$skip=0
```

**Parámetros:**
- `$filter=DateTimeFrom ge {fecha_inicio} and DateTimeTo le {fecha_fin} and Status eq CheckedIn` - Rango de fechas y estado Presentado
- `$orderby=DateTimeFrom asc` - Ordenar por fecha de inicio ascendente
- `$top=500` - Máximo 500 resultados por página (OBLIGATORIO)
- `$skip=0` - Offset para paginación (OBLIGATORIO)

**Resultado:** Obtiene todas las citas en el rango de fechas especificado con estado Presentado (CheckedIn).

**Explicación del filtro:**
- `DateTimeFrom ge {fecha_inicio}` = Citas desde esta fecha/hora (mayor o igual)
- `DateTimeTo le {fecha_fin}` = Citas hasta esta fecha/hora (menor o igual)
- `Status eq CheckedIn` = Estado igual a CheckedIn (Presentado)

## 1. Documentación de Referencia de la API

### 1.1. Endpoint de Metadata OData
La API OData proporciona automáticamente documentación mediante el endpoint `$metadata`:

```
GET https://proxy.megasalud.cl/ThirdPartyService/$metadata
```

**Contenido que proporciona:**
- Definición completa del modelo de datos (EDM)
- Todas las entidades disponibles
- Propiedades de cada entidad y sus tipos
- Relaciones entre entidades
- Funciones y acciones disponibles
- Enums y sus valores
- Namespaces exactos para usar en queries

**Recomendación:** Incluir en la documentación cómo acceder e interpretar este endpoint.

### 1.2. Endpoint para Consulta de Citas (Excepto Anuladas)

#### Endpoint Base
- **ThirdPartyService/Appointments** - Endpoint para consultar citas médicas

#### Información del Endpoint
- **URL Base:** `https://proxy.megasalud.cl/ThirdPartyService/Appointments`
- **Método HTTP:** `GET` (solo lectura para este requerimiento)
- **Propósito:** Consultar todas las citas (excepto anuladas) dentro de un rango de fechas con soporte de paginación

#### Propiedades Principales de AppointmentDTO
Las siguientes propiedades están disponibles en las respuestas:

**Identificadores:**
- `Id` (Guid) - Identificador único de la cita
- `PatientId` (Guid) - Identificador del paciente
- `ResourceId` (Guid) - Identificador del recurso (médico/equipo)
- `CenterId` (Guid) - Identificador del centro médico
- `ServiceId` (Guid) - Identificador del servicio
- `AppointmentTypeId` (Guid) - Identificador del tipo de cita
- `CoveragePlanId` (Guid?) - Identificador del plan de cobertura (opcional)

**Fechas y Tiempo:**
- `DateTimeFrom` (DateTimeOffset) - Fecha y hora de inicio (timezone local)
- `DateTimeTo` (DateTimeOffset) - Fecha y hora de fin (timezone local)
- `DateTimeFromUTC` (DateTimeOffset) - Fecha y hora de inicio en UTC (solo lectura)
- `DateTimeToUTC` (DateTimeOffset) - Fecha y hora de fin en UTC (solo lectura)
- `Duration` (int) - Duración de la cita en minutos

**Estado y Control:**
- `Status` (AppointmentStatus) - Estado de la cita (enum)
- `RowVersion` (string) - Versión para control de concurrencia optimista
- `Archived` (bool) - Indica si la cita está archivada
- `Cancelled` (bool) - Indica si la cita está cancelada

**Otros:**
- `IsOvercapacity` (bool) - Indica si está sobre capacidad
- `NumberOfSlots` (int) - Número de slots de la cita
- `Index` (int) - Índice de la cita
- `Number` (int?) - Número de la cita (opcional)
- `CommentsCount` (int) - Cantidad de comentarios

**Relaciones disponibles (mediante $expand):**
- `Patient` - Información del paciente
- `Resource` - Información del recurso
- `Center` - Información del centro
- `Service` - Información del servicio
- `ResourceAppointments` - Relación con recursos adicionales
- `AppointmentType` - Tipo de cita

**Nota:** Para obtener información detallada de todas las propiedades y relaciones, consultar el endpoint `$metadata`.

## 2. Guía de Autenticación y Autorización

### 2.1. Autenticación

**Tipo de autenticación:** OAuth 2.0 Bearer Token

**Cómo obtener el token:**
- Contactar al equipo de integración de RedSalud para obtener credenciales de acceso
- El token se proporciona como string base64

**Formato del header:**
```
Authorization: Bearer {token}
```

**Ejemplo:**
```
Authorization: Bearer Tj17YiZYaDx5OFMzWl4iKw==
```

**Validez y expiración:**
- Los tokens tienen una validez limitada
- Si se recibe un error `401 Unauthorized`, el token puede haber expirado
- Contactar al equipo de integración para renovar el token

**Nota:** En la próxima versión (1 sprint), se implementarán credenciales específicas para proveedores externos con mejor gestión de tokens.

### 2.2. Headers Requeridos

**Headers obligatorios:**
- `Authorization: Bearer {token}` - Token de autenticación OAuth 2.0
  - Formato: `Bearer {token_base64}`
  - Ejemplo: `Bearer Tj17YiZYaDx5OFMzWl4iKw==`

- `X-AppTimezone: {offset}` - Zona horaria de la aplicación
  - Offset en minutos desde UTC
  - Ejemplos:
    - `-240` para UTC-4 (Chile continental)
    - `-180` para UTC-3
    - `0` para UTC

**Headers opcionales pero recomendados:**
- `Accept: application/json, text/plain, */*` - Formato de respuesta esperado
- `User-Agent: {user_agent}` - Identificador de la aplicación cliente
- `Referer: {url}` - URL de origen de la solicitud

### 2.3. Autorización por Cliente

**Endpoints disponibles:**
- **ThirdPartyService/** - Para integraciones de terceros (usado en este requerimiento)
- **AWAPatients/** - Para acceso de pacientes (no aplica para este requerimiento)

**Permisos para este requerimiento:**
- Acceso de solo lectura (GET) al endpoint `/ThirdPartyService/Appointments`
- Filtrado por rango de fechas y estado `Confirmed`
- Paginación de resultados

**Restricciones:**
- Solo se pueden consultar citas (excepto anuladas) - Solo lectura (GET)
- No se permite modificar, crear o eliminar citas mediante este endpoint
- El acceso está limitado a los datos permitidos por el token proporcionado

## 3. Guía de Consultas OData

### 3.1. Operadores de Filtrado ($filter)

**Operadores soportados en OData v4:**

**Operadores de comparación:**
- `eq` - Igual a
- `ne` - No igual a
- `gt` - Mayor que
- `ge` - Mayor o igual que
- `lt` - Menor que
- `le` - Menor o igual que

**Operadores lógicos:**
- `and` - Y lógico
- `or` - O lógico
- `not` - Negación

**Funciones de cadena (para este requerimiento no son necesarias, pero están disponibles):**
- `contains` - Contiene subcadena
- `startswith` - Comienza con
- `endswith` - Termina con
- `length` - Longitud
- `indexof` - Índice de subcadena

**Operadores de colecciones:**
- `any` - Cualquier elemento cumple condición
- `all` - Todos los elementos cumplen condición

**Ejemplos:**
```
# Filtrar por fecha
DateTimeFrom ge 2025-12-15T21:48:36-03:00

# Filtrar por múltiples condiciones
DateTimeFrom ge 2025-12-15T21:48:36-03:00 and DateTimeTo le 2026-02-13T21:48:36-03:00

# Filtrar en colecciones
ResourceAppointments/any(x: x/ResourceId eq {guid})
```

### 3.2. Filtrado por Enums

**Problema común:** Los enums requieren sintaxis especial.

**Sintaxis correcta (VERIFICADA):**
```
Status eq WebApiModel.Enum.AppointmentStatus'Confirmed'
```

**En URL encoding:**
```
Status%20eq%20WebApiModel.Enum.AppointmentStatus%27Confirmed%27
```
Donde `%27` es la comilla simple (`'`) y `%20` es el espacio.

**Valores de AppointmentStatus - Explicación Completa:**

**⚠️ IMPORTANTE:** El tope máximo de consulta es **500 resultados por página** (`$top=500`).

| Valor en API | Descripción en Español | Valor Numérico | Descripción |
|--------------|------------------------|---------------|-------------|
| `Booked` | **Agendado** | 0 | Citas que han sido agendadas pero aún no confirmadas |
| `Confirmed` | **Confirmado** | 1 | Citas que han sido confirmadas |
| `CheckedIn` | **Presentado** | 3 | Citas donde el paciente se ha presentado (check-in realizado) |
| `ServicePerformed` | **Atendido** | 4 | Citas donde el servicio ya fue realizado/atendido |
| `Blocked` | **Bloqueado** | 5 | Citas que están bloqueadas |
| `NotPerformed` | **No Presentado** | 6 | Citas donde el paciente no se presentó |
| `Cancelled` | **Anulado** | 2 | Citas que han sido anuladas (excluidas del requerimiento principal) |

**Modos de Consulta Disponibles:**

**Modo 1: Consultar por Estado Individual**
```
# Solo Agendado
Status eq WebApiModel.Enum.AppointmentStatus'Booked'

# Solo Confirmado
Status eq WebApiModel.Enum.AppointmentStatus'Confirmed'

# Solo Presentado
Status eq WebApiModel.Enum.AppointmentStatus'CheckedIn'

# Solo Atendido
Status eq WebApiModel.Enum.AppointmentStatus'ServicePerformed'

# Solo Bloqueado
Status eq WebApiModel.Enum.AppointmentStatus'Blocked'

# Solo No Presentado
Status eq WebApiModel.Enum.AppointmentStatus'NotPerformed'
```
- `eq` = "equal" (igual a)
- Filtra por un estado específico
- **Primera descripción:** Corresponde al filtro (ej: `Booked` => Agendado)
- **Segunda descripción:** Corresponde a la explicación del filtro (ej: Citas que han sido agendadas pero aún no confirmadas)

**Nota importante:** Aunque `EnableEnumPrefixFree(true)` está configurado, la sintaxis completamente calificada es la que funciona de manera confiable en esta implementación.

### 3.3. Ordenamiento ($orderby)
```
$orderby=DateTimeFrom asc
$orderby=DateTimeFrom desc, Status asc
```

### 3.4. Paginación ($top, $skip)
```
$top=100        # Limitar resultados
$skip=50        # Saltar resultados (para paginación)
$top=100&$skip=0
```

**Límites:**
- DefaultTop: 100
- $top: 500 (OBLIGATORIO: máximo permitido para todas las consultas)

### 3.5. Selección de Propiedades ($select)
```
$select=Id,DateTimeFrom,DateTimeTo,Status,PatientId
```

### 3.6. Expansión de Relaciones ($expand)
```
$expand=Patient,ResourceAppointments
$expand=ResourceAppointments($expand=Resource)
```

**Límites:**
- MaxExpansionDepth: 2 (para Appointments)

## 4. Ejemplos de Uso Comunes

### 4.1. Obtener Citas Confirmadas en un Rango de Fechas

**Ejemplo completo y funcional:**

```bash
curl --location 'https://proxy.megasalud.cl/ThirdPartyService/Appointments?$filter=DateTimeFrom ge 2025-12-15T21:48:36-03:00 and DateTimeTo le 2026-02-13T21:48:36-03:00 and Status eq WebApiModel.Enum.AppointmentStatus'\''Confirmed'\''&$orderby=DateTimeFrom asc&$top=500&$skip=0' \
  --header 'Authorization: Bearer {tu_token_aqui}' \
  --header 'X-AppTimezone: -240' \
  --header 'Accept: application/json, text/plain, */*' \
  --header 'User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36'
```

**URL decodificada para referencia:**
```
https://proxy.megasalud.cl/ThirdPartyService/Appointments?
  $filter=DateTimeFrom ge 2025-12-15T21:48:36-03:00 
    and DateTimeTo le 2026-02-13T21:48:36-03:00 
    and Status eq WebApiModel.Enum.AppointmentStatus'Confirmed'
  &$orderby=DateTimeFrom asc
  &$top=500
```

**Explicación del filtro:**
- `DateTimeFrom ge 2025-12-15T21:48:36-03:00` - Citas desde esta fecha/hora
- `DateTimeTo le 2026-02-13T21:48:36-03:00` - Citas hasta esta fecha/hora
- `Status eq WebApiModel.Enum.AppointmentStatus'Confirmed'` 
  - **`eq` = "equal" (igual a)**
  - Esto significa: "Status **igual a** Confirmed (Confirmado)"
  - **Resultado:** INCLUYE solo las citas con estado Confirmado
- `$orderby=DateTimeFrom asc` - Ordenadas por fecha de inicio ascendente
- `$top=500` - OBLIGATORIO: máximo permitido de 500 resultados por página
- `$skip=0` - OBLIGATORIO: offset para paginación

**Headers importantes:**
- `Authorization: Bearer {token}` - **REQUERIDO** - Token de autenticación
- `X-AppTimezone: -240` - **REQUERIDO** - Offset de zona horaria en minutos (UTC-4 = -240 minutos)
- `Accept: application/json` - Formato de respuesta esperado

## 6. Manejo de Errores

### 6.1. Códigos HTTP Comunes
- `200 OK` - Solicitud exitosa
- `201 Created` - Recurso creado exitosamente
- `400 Bad Request` - Solicitud inválida (sintaxis OData incorrecta, validación fallida)
- `401 Unauthorized` - Token inválido o faltante
- `403 Forbidden` - Sin permisos para el recurso
- `404 Not Found` - Recurso no encontrado
- `409 Conflict` - Conflicto de concurrencia (RowVersion)
- `500 Internal Server Error` - Error del servidor

### 6.2. Formato de Errores OData
```json
{
  "error": {
    "code": "ErrorCode",
    "message": "Mensaje de error descriptivo",
    "innererror": {
      "message": "Detalles técnicos del error",
      "type": "TipoDeExcepción",
      "stacktrace": "..."
    }
  }
}
```

### 6.3. Errores Comunes y Soluciones

**Error:** `A binary operator with incompatible types was detected`
- **Causa:** Comparación de tipos incompatibles (ej: enum con string)
- **Solución:** Usar sintaxis correcta para enums: `Status eq WebApiModel.Enum.AppointmentStatus'Confirmed'`

**Error:** `Could not find a property named 'X'`
- **Causa:** Propiedad no existe o nombre incorrecto
- **Solución:** Verificar en `$metadata` el nombre correcto de la propiedad

**Error:** `The query specified in the URI is not valid`
- **Causa:** Sintaxis OData incorrecta
- **Solución:** Validar sintaxis según especificación OData v4

## 7. Mejores Prácticas

### 7.1. Performance
- Usar `$select` para limitar campos retornados
- Usar `$top` para limitar cantidad de resultados
- Evitar `$expand` innecesario (tiene costo de performance)
- Usar filtros específicos en lugar de traer todos los datos

### 7.2. Zona Horaria
- Siempre incluir header `X-AppTimezone` con el offset correcto
- Las fechas en queries deben estar en el timezone de la aplicación
- Las fechas retornadas incluyen `DateTimeFromUTC` y `DateTimeToUTC`

### 7.3. Concurrencia
- Usar `RowVersion` para control de concurrencia optimista
- Incluir `@odata.etag` en headers para actualizaciones

### 7.4. Paginación
- Implementar paginación para grandes volúmenes de datos
- Usar `$skip` y `$top` en conjunto
- Considerar límites de `MaxTop`

## 8. Herramientas Recomendadas

### 8.1. Para Probar la API
- **Postman** - Colección de requests
- **Insomnia** - Cliente REST alternativo
- **OData Query Builder** - Herramientas visuales para construir queries
- **curl** - Línea de comandos

### 8.2. Para Documentación
- **Swagger/OpenAPI** - Generar documentación interactiva (recomendado implementar)
- **Postman Collections** - Compartir colecciones con ejemplos
- **Markdown** - Documentación estática (como este documento)

## 9. Recursos Adicionales

### 9.1. Especificación OData
- [OData v4 Specification](http://docs.oasis-open.org/odata/odata/v4.01/odata-v4.01-part1-protocol.html)
- [OData Query Options](http://docs.oasis-open.org/odata/odata/v4.01/odata-v4.01-part2-url-conventions.html)

### 9.2. Librerías Cliente OData
- **.NET:** Microsoft.OData.Client
- **JavaScript:** @odata/client o datajs
- **Python:** pyodata
- **Java:** Olingo OData Client

## 10. Checklist de Documentación Recomendada

- [ ] **README.md** actualizado con información básica del proyecto
- [ ] **Guía de Inicio Rápido** - Cómo hacer la primera llamada a la API
- [ ] **Documentación de Autenticación** - Cómo obtener y usar tokens
- [ ] **Referencia de Endpoints** - Lista completa de endpoints disponibles
- [ ] **Guía de Queries OData** - Ejemplos de filtros, ordenamiento, paginación
- [ ] **Documentación de Enums** - Valores y sintaxis correcta
- [ ] **Ejemplos de Código** - En múltiples lenguajes (cURL, C#, JavaScript, Python)
- [ ] **Colección Postman** - Requests pre-configurados
- [ ] **Swagger/OpenAPI** - Documentación interactiva (ideal)
- [ ] **Guía de Manejo de Errores** - Códigos y soluciones comunes
- [ ] **Changelog/Versionado** - Historial de cambios en la API
- [ ] **Límites y Cuotas** - Rate limiting, tamaño máximo de requests, etc.

## 11. Implementación de Swagger/OpenAPI (Recomendado)

Para mejorar significativamente la experiencia del desarrollador, se recomienda implementar:

1. **Swashbuckle** para ASP.NET Web API
2. **Generar documentación automática** desde los controladores
3. **Incluir ejemplos** en la documentación
4. **Probar endpoints** directamente desde la UI de Swagger

Esto proporcionaría:
- Documentación interactiva
- Pruebas en tiempo real
- Esquemas de datos actualizados automáticamente
- Ejemplos de requests/responses

---

**Última actualización:** 2025-01-XX
**Versión de la documentación:** 1.0
**Estado:** 
- ⚠️ Endpoint NO disponible para proveedores hasta completar:
- Enmascaramiento mediante API Gateway (Pedro Wittig)
- Certificaciones de calidad completas
- Ruta segura con credenciales diferenciadas
**Mantenido por:** Equipo de Integración RedSalud

