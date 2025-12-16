# Guía de Documentación Técnica de la API

Esta guía describe qué documentación técnica es necesaria para que cualquier desarrollador pueda utilizar la API OData de manera efectiva.

## Ejemplo Completo de Uso

### Obtener Citas Confirmadas - cURL Completo

Este es un ejemplo real y funcional de cómo consultar citas confirmadas en un rango de fechas:

```bash
curl --location 'https://proxy.megasalud.cl/ThirdPartyService/Appointments?%24filter=DateTimeFrom%20ge%202025-12-15T21%3A48%3A36-03%3A00%20and%20DateTimeTo%20le%202026-02-13T21%3A48%3A36-03%3A00%20and%20Status%20eq%20WebApiModel.Enum.AppointmentStatus%27Confirmed%27&%24orderby=DateTimeFrom%20asc&%24top=500' \
--header 'sec-ch-ua-platform: "Windows"' \
--header 'Referer: https://agenda.redsalud.cl/' \
--header 'User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/143.0.0.0 Safari/537.36' \
--header 'Accept: application/json, text/plain, */*' \
--header 'sec-ch-ua: "Google Chrome";v="143", "Chromium";v="143", "Not A(Brand";v="24"' \
--header 'sec-ch-ua-mobile: ?0' \
--header 'X-AppTimezone: -240' \
--header 'Authorization: Bearer Tj17YiZYaDx5OFMzWl4iKw=='
```

**Desglose del ejemplo:**

1. **URL Base:** `https://proxy.megasalud.cl/ThirdPartyService/Appointments`
2. **Query Parameters (URL decoded):**
   - `$filter=DateTimeFrom ge 2025-12-15T21:48:36-03:00 and DateTimeTo le 2026-02-13T21:48:36-03:00 and Status eq WebApiModel.Enum.AppointmentStatus'Confirmed'`
   - `$orderby=DateTimeFrom asc`
   - `$top=1000` (OBLIGATORIO: valor fijo)
3. **Headers críticos:**
   - `Authorization: Bearer {token}` - Token de autenticación
   - `X-AppTimezone: -240` - Zona horaria (UTC-4)

**Nota:** Reemplaza el token en el header `Authorization` con tu token válido.

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

### 1.2. Endpoint para Consulta de Citas Confirmadas

#### Endpoint Base
- **ThirdPartyService/Appointments** - Endpoint para consultar citas médicas

#### Información del Endpoint
- **URL Base:** `https://proxy.megasalud.cl/ThirdPartyService/Appointments`
- **Método HTTP:** `GET` (solo lectura para este requerimiento)
- **Propósito:** Consultar citas confirmadas dentro de un rango de fechas con soporte de paginación

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
- Solo se pueden consultar citas confirmadas
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

**Valores de AppointmentStatus:**
- `Booked` (0) - Reservada
- `Confirmed` (1) - Confirmada
- `Cancelled` (2) - Cancelada
- `CheckedIn` (3) - Registrada/Check-in realizado
- `ServicePerformed` (4) - Servicio realizado
- `Blocked` (5) - Bloqueada
- `NotPerformed` (6) - No realizada

**Ejemplos de uso en filtros:**
```
# Solo citas confirmadas
Status eq WebApiModel.Enum.AppointmentStatus'Confirmed'

# Citas canceladas o no realizadas
Status eq WebApiModel.Enum.AppointmentStatus'Cancelled' or Status eq WebApiModel.Enum.AppointmentStatus'NotPerformed'

# Excluir citas canceladas
Status ne WebApiModel.Enum.AppointmentStatus'Cancelled'
```

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
- $top: 1000 (OBLIGATORIO: valor fijo para todas las consultas)

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
curl --location 'https://proxy.megasalud.cl/ThirdPartyService/Appointments?$filter=DateTimeFrom ge 2025-12-15T21:48:36-03:00 and DateTimeTo le 2026-02-13T21:48:36-03:00 and Status eq WebApiModel.Enum.AppointmentStatus'\''Confirmed'\''&$orderby=DateTimeFrom asc&$top=1000&$skip=0' \
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
- `Status eq WebApiModel.Enum.AppointmentStatus'Confirmed'` - Solo citas confirmadas
- `$orderby=DateTimeFrom asc` - Ordenadas por fecha de inicio ascendente
- `$top=1000` - OBLIGATORIO: valor fijo de 1000 resultados por página

**Headers importantes:**
- `Authorization: Bearer {token}` - **REQUERIDO** - Token de autenticación
- `X-AppTimezone: -240` - **REQUERIDO** - Offset de zona horaria en minutos (UTC-4 = -240 minutos)
- `Accept: application/json` - Formato de respuesta esperado

### 4.2. Obtener Slots Disponibles por Ubicación
```bash
curl 'https://proxy.megasalud.cl/AWAPatients/Slots/GetSlotsByLocationOptimized(includeSelfPayer=false,expertBookingMode=false,includeNotBookable=true,startingLocationId={locationId},maximumDistance=0,rescheduleAppointmentId=null)?$filter=StartTime ge 2025-12-15T21:48:36-03:00 and FinishTime le 2026-02-13T21:48:36-03:00&$orderby=StartTime asc' \
  -H 'Authorization: Bearer {token}' \
  -H 'X-AppTimezone: -240' \
  -H 'Accept: application/json'
```

### 4.3. Obtener un Paciente por ID
```bash
curl 'https://proxy.megasalud.cl/ThirdPartyService/Patients({patientId})?$expand=CoveragePlans' \
  -H 'Authorization: Bearer {token}' \
  -H 'Accept: application/json'
```

### 4.4. Crear una Cita
```bash
curl -X POST 'https://proxy.megasalud.cl/ThirdPartyService/Appointments' \
  -H 'Authorization: Bearer {token}' \
  -H 'Content-Type: application/json' \
  -H 'X-AppTimezone: -240' \
  -d '{
    "PatientId": "{patientId}",
    "ResourceId": "{resourceId}",
    "CenterId": "{centerId}",
    "ServiceId": "{serviceId}",
    "DateTimeFrom": "2025-12-20T10:00:00-03:00",
    "DateTimeTo": "2025-12-20T10:30:00-03:00",
    "AppointmentTypeId": "{appointmentTypeId}",
    "Status": "Booked"
  }'
```

## 5. Funciones y Acciones Personalizadas

### 5.1. Funciones de Appointments
- `GetByUserLocations(filterByUserLocations={bool})` - Obtener citas filtradas por ubicaciones del usuario
- `GetAppointmentsPdf(ResourceId, CenterId, DateFrom, DateTo, Status, Title, Language)` - Generar PDF de citas
- `GetByExternalKey(origin, key)` - Obtener cita por clave externa
- `GetByToken(Token)` - Obtener cita por token

### 5.2. Funciones y Acciones Disponibles

**Nota importante:** Para el requerimiento actual (consultar citas confirmadas), solo se requiere el método `GET`. Las siguientes funciones y acciones están disponibles en la API pero **no son necesarias** para este requerimiento específico:

**Funciones disponibles (no requeridas para este caso):**
- `GetByUserLocations(filterByUserLocations)` - Obtener citas filtradas por ubicaciones del usuario
- `GetAppointmentsPdf(...)` - Generar PDF de citas
- `GetByExternalKey(origin, key)` - Obtener cita por clave externa
- `GetByToken(Token)` - Obtener cita por token

**Acciones disponibles (no requeridas para este caso):**
- `UpdateStatus` - Actualizar estado de una cita (requiere permisos de escritura)
- `CancelAppointment` - Cancelar una cita (requiere permisos de escritura)
- `StartCall` - Iniciar llamada (requiere permisos de escritura)
- `EndCall` - Finalizar llamada (requiere permisos de escritura)
- `Import` - Importar citas (requiere permisos de escritura)
- `ModifyByToken` - Modificar cita por token (requiere permisos de escritura)

**Para este requerimiento:** Solo se necesita usar el método `GET` con filtros `$filter`, `$orderby`, `$top` y `$skip` para consultar citas confirmadas con paginación.

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
**Estado:** Endpoint temporal - Mejoras planificadas para 1 sprint (Pedro Wittig)
**Mantenido por:** Equipo de Integración RedSalud

