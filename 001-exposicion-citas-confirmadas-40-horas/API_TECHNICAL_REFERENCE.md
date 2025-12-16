# Documentación Técnica de la API - Consulta de Citas

## 1. Contexto del Requerimiento

### 1.1. Objetivo
Proporcionar a proveedores externos la capacidad de consultar todas las citas (excepto las anuladas) dentro de un rango de fechas específico mediante una API REST basada en OData v4.

### 1.2. Alcance
- **Entidad:** Appointments (Citas)
- **Filtro principal:** Todas las citas EXCEPTO estado `Cancelled` (Anulado)
- **Estados incluidos:** Booked (Agendado), Confirmed (Confirmado), Checked-In (Presentado), Treated (Atendido), Blocked (Bloqueado), Not Treated (No Presentado)
- **Filtro secundario:** Rango de fechas (`DateTimeFrom` y `DateTimeTo`)
- **Funcionalidad:** Consulta de solo lectura (GET) - Solo consulta, no se permite crear, actualizar o modificar
- **Paginación:** Requerida para manejar grandes volúmenes de datos

### 1.3. Estado de Implementación y Requisitos de Seguridad

#### ⚠️ REQUISITO CRÍTICO: API Gateway y Certificaciones

**ANTES de disponibilizar estas rutas para consumo de integraciones o proveedores externos, se DEBEN cumplir los siguientes requisitos obligatorios:**

1. **Enmascaramiento mediante API Gateway**
   - **OBLIGATORIO:** Las rutas DEBEN ser enmascaradas por API Gateway
   - El endpoint actual (`https://proxy.megasalud.cl/ThirdPartyService/Appointments`) NO debe exponerse directamente a proveedores externos
   - Se requiere implementar un API Gateway que actúe como intermediario y capa de seguridad

2. **Certificaciones de Calidad**
   - **OBLIGATORIO:** Pasar TODAS las certificaciones de calidad antes de disponibilizar el consumo
   - Incluye pruebas de estrés, seguridad, performance, y validaciones funcionales
   - No se permite disponibilizar el endpoint sin completar todas las certificaciones

3. **Responsable de Implementación**
   - **Responsable:** Pedro Wittig
   - **Tarea:** 
     - Implementar enmascaramiento mediante API Gateway
     - Disponibilizar ruta segura con credenciales diferenciadas para proveedores externos
     - Asegurar que todas las certificaciones de calidad sean aprobadas

#### Estado Actual (NO DISPONIBLE PARA PROVEEDORES)
- **Endpoint actual:** `https://proxy.megasalud.cl/ThirdPartyService/Appointments`
- **Estado:** Endpoint temporal, NO debe usarse directamente por proveedores externos
- **Autenticación:** Bearer Token (sistema actual)
- **Acceso:** Mediante credenciales existentes del sistema
- **⚠️ ADVERTENCIA:** Este endpoint NO está listo para consumo de proveedores externos hasta que se complete el enmascaramiento por API Gateway y las certificaciones de calidad

#### Estado Futuro (Requisitos para Disponibilización)
- **Requisito 1:** Enmascaramiento mediante API Gateway ✅ (Pendiente)
- **Requisito 2:** Certificaciones de calidad completadas ✅ (Pendiente)
- **Requisito 3:** Ruta segura con credenciales diferenciadas ✅ (Pendiente)
- **Beneficios una vez implementado:**
  - Seguridad mejorada con API Gateway como intermediario
  - Credenciales específicas para proveedores
  - Aislamiento de acceso
  - Mejor control y auditoría
  - Rate limiting específico
  - Protección de la base de datos productiva

**Nota:** Esta documentación describe el endpoint actual y los requisitos para su disponibilización. Una vez implementado el API Gateway y completadas las certificaciones, se actualizará esta documentación con las nuevas rutas y credenciales.

### 1.4. Consideraciones Técnicas Importantes

⚠️ **REQUISITOS OBLIGATORIOS ANTES DE DISPONIBILIZAR PARA PROVEEDORES:**

1. **Enmascaramiento mediante API Gateway** ⚠️ **CRÍTICO**
   - **OBLIGATORIO:** Las rutas DEBEN ser enmascaradas por API Gateway antes de disponibilizar su consumo
   - El endpoint actual NO debe exponerse directamente a integraciones o proveedores externos
   - El API Gateway actúa como capa de seguridad, control de acceso y protección

2. **Certificaciones de Calidad Completas** ⚠️ **CRÍTICO**
   - **OBLIGATORIO:** Pasar TODAS las certificaciones de calidad antes de disponibilizar
   - Incluye: pruebas de estrés, seguridad, performance, validaciones funcionales
   - No se permite disponibilizar sin completar todas las certificaciones
   - Contactar al equipo de operaciones para coordinar las certificaciones

3. **Pruebas de Calidad de Estrés en Ambiente QA**
   - El proceso de integración **DEBE** pasar pruebas de calidad de estrés en ambiente QA antes de ser desplegado a producción
   - Estas pruebas son obligatorias para no afectar la base de datos productiva
   - Contactar al equipo de operaciones para coordinar las pruebas de estrés

4. **Ventana de Ejecución: Después de las 03:00 AM**
   - **OBLIGATORIO:** El proveedor, integración o proceso que consuma la API **DEBE** ejecutar las comunicaciones después de las 03:00 AM (horario local)
   - Esta restricción aplica para evitar impacto en la carga de la base de datos durante horarios de alta demanda
   - Las consultas ejecutadas fuera de esta ventana pueden ser rechazadas o limitadas

5. **Paginación Obligatoria con $top=500**
   - **OBLIGATORIO:** Todas las consultas **DEBEN** ser paginadas
   - El parámetro `$top` **SIEMPRE** debe ser `500` (máximo permitido, no se permite otro valor)
   - El parámetro `$skip` es obligatorio para navegar entre páginas
   - No se permite realizar consultas sin paginación

6. **Confirmación con Operaciones**
   - Se debe confirmar con el equipo de operaciones si hace falta alguna definición adicional a considerar
   - Contactar al equipo de operaciones antes de iniciar la integración en producción
   - Verificar que el API Gateway esté configurado y las certificaciones de calidad estén completadas

---

## 2. Información General de la API

### 2.1. Base URL
```
https://proxy.megasalud.cl/ThirdPartyService
```

### 2.2. Protocolo
- **Protocolo:** HTTPS
- **Versión OData:** OData v4
- **Formato de respuesta:** JSON

### 2.3. Endpoint de Metadata
Para obtener la definición completa del modelo de datos:
```
GET https://proxy.megasalud.cl/ThirdPartyService/$metadata
```

---

## 3. Autenticación

### 3.1. Método Actual
**Tipo:** OAuth 2.0 Bearer Token

**Header requerido:**
```
Authorization: Bearer {token}
```

**Ejemplo:**
```
Authorization: Bearer Tj17YiZYaDx5OFMzWl4iKw==
```

### 3.2. Obtención de Token
Contactar al equipo de integración para obtener credenciales de acceso.

**Nota:** En la próxima versión (1 sprint), se proporcionarán credenciales específicas para proveedores externos.

### 3.3. Headers Requeridos

| Header | Valor | Requerido | Descripción |
|--------|-------|-----------|-------------|
| `Authorization` | `Bearer {token}` | Sí | Token de autenticación |
| `X-AppTimezone` | `-240` | Sí | Offset de zona horaria en minutos (UTC-4 = -240) |
| `Accept` | `application/json` | Recomendado | Formato de respuesta esperado |

**Ejemplo completo de headers:**
```http
Authorization: Bearer Tj17YiZYaDx5OFMzWl4iKw==
X-AppTimezone: -240
Accept: application/json, text/plain, */*
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36
```

---

## 4. Endpoint: Consultar Citas (Excluyendo Anuladas)

### 4.1. Endpoint Base
```
GET /ThirdPartyService/Appointments
```

### 4.2. Descripción
Retorna una colección de citas dentro de un rango de fechas especificado, **excluyendo las citas con estado Cancelled (Anulado)**.

**Estados incluidos en la consulta:**
- `Booked` (Agendado)
- `Confirmed` (Confirmado)
- `Checked-In` (Presentado)
- `Treated` (Atendido)
- `Blocked` (Bloqueado)
- `Not Treated` (No Presentado)

**Estados excluidos:**
- `Cancelled` (Anulado) - **NO se incluyen en los resultados**

**⚠️ PAGINACIÓN OBLIGATORIA:**
- Este endpoint **REQUIERE** paginación en todas las consultas
- Los parámetros `$top=500` y `$skip` son **OBLIGATORIOS**
- No se permite realizar consultas sin paginación
- El valor de `$top` **SIEMPRE** debe ser `500` (máximo permitido)

**⚠️ SOLO CONSULTA:**
- Este endpoint es de **solo lectura (GET)**
- No se permite crear, actualizar, modificar o eliminar citas
- Solo se puede consultar información

### 4.3. Parámetros de Query (OData)

**Parámetros Obligatorios:**
- `$filter` - Filtros de fecha y estado (OBLIGATORIO)
- `$top=500` - Tamaño de página (OBLIGATORIO, máximo permitido: 500)
- `$skip` - Offset para paginación (OBLIGATORIO)

**Parámetros Opcionales:**
- `$orderby` - Ordenamiento de resultados
- `$select` - Selección de campos específicos
- `$expand` - Expansión de relaciones

#### 4.3.1. $filter (OBLIGATORIO)
Filtros aplicados a la consulta.

**⚠️ IMPORTANTE:** El tope máximo de consulta es **500 resultados por página** (usar `$top=500`).

**Formato de fecha:**
- Formato ISO 8601 con timezone: `YYYY-MM-DDTHH:mm:ss-TZ:tz`
- Ejemplo: `2025-12-15T21:48:36-03:00`

##### Modos de Consulta Disponibles

###### Modo 1: Consultar Todos los Estados (Excepto Anuladas)

**Sintaxis:**
```
$filter=DateTimeFrom ge {fecha_inicio} and DateTimeTo le {fecha_fin} and Status ne WebApiModel.Enum.AppointmentStatus'Cancelled'
```

**Explicación:**
- `Status ne` = "Status **diferente de**" (ne = not equal)
- `Status ne Cancelled` = EXCLUYE las citas anuladas
- **Incluye automáticamente:** Booked, Confirmed, CheckedIn, ServicePerformed, Blocked, NotPerformed

**Ejemplo completo:**
```
$filter=DateTimeFrom ge 2025-12-15T21:48:36-03:00 and DateTimeTo le 2026-02-13T21:48:36-03:00 and Status ne WebApiModel.Enum.AppointmentStatus'Cancelled'&$top=500&$skip=0
```

###### Modo 2: Consultar por Estado Individual

Puedes filtrar por cada estado específico usando `Status eq` (equal = igual a):

**2.1. Solo Citas Agendadas (Booked)**
```
$filter=DateTimeFrom ge {fecha_inicio} and DateTimeTo le {fecha_fin} and Status eq WebApiModel.Enum.AppointmentStatus'Booked'&$top=500&$skip=0
```

**2.2. Solo Citas Confirmadas (Confirmed)**
```
$filter=DateTimeFrom ge {fecha_inicio} and DateTimeTo le {fecha_fin} and Status eq WebApiModel.Enum.AppointmentStatus'Confirmed'&$top=500&$skip=0
```

**2.3. Solo Citas Presentadas (CheckedIn)**
```
$filter=DateTimeFrom ge {fecha_inicio} and DateTimeTo le {fecha_fin} and Status eq WebApiModel.Enum.AppointmentStatus'CheckedIn'&$top=500&$skip=0
```

**2.4. Solo Citas Atendidas (ServicePerformed)**
```
$filter=DateTimeFrom ge {fecha_inicio} and DateTimeTo le {fecha_fin} and Status eq WebApiModel.Enum.AppointmentStatus'ServicePerformed'&$top=500&$skip=0
```

**2.5. Solo Citas Bloqueadas (Blocked)**
```
$filter=DateTimeFrom ge {fecha_inicio} and DateTimeTo le {fecha_fin} and Status eq WebApiModel.Enum.AppointmentStatus'Blocked'&$top=500&$skip=0
```

**2.6. Solo Citas No Presentadas (NotPerformed)**
```
$filter=DateTimeFrom ge {fecha_inicio} and DateTimeTo le {fecha_fin} and Status eq WebApiModel.Enum.AppointmentStatus'NotPerformed'&$top=500&$skip=0
```

**⚠️ NOTA:** No se recomienda consultar citas anuladas (Cancelled) ya que están excluidas del requerimiento, pero si fuera necesario:
```
$filter=DateTimeFrom ge {fecha_inicio} and DateTimeTo le {fecha_fin} and Status eq WebApiModel.Enum.AppointmentStatus'Cancelled'&$top=500&$skip=0
```

##### Explicación de Estados

| Valor en API | Descripción en Español | Valor Numérico | Uso Recomendado |
|--------------|------------------------|----------------|------------------|
| `Booked` | **Agendado** | 0 | Citas que han sido agendadas pero aún no confirmadas |
| `Confirmed` | **Confirmado** | 1 | Citas que han sido confirmadas |
| `CheckedIn` | **Presentado** | 3 | Citas donde el paciente se ha presentado (check-in realizado) |
| `ServicePerformed` | **Atendido** | 4 | Citas donde el servicio ya fue realizado/atendido |
| `Blocked` | **Bloqueado** | 5 | Citas que están bloqueadas |
| `NotPerformed` | **No Presentado** | 6 | Citas donde el paciente no se presentó |
| `Cancelled` | **Anulado** | 2 | Citas que han sido anuladas (excluidas del requerimiento principal) |

**Componentes comunes del filtro:**
- `DateTimeFrom ge {fecha_inicio}` - Citas desde esta fecha/hora (mayor o igual)
- `DateTimeTo le {fecha_fin}` - Citas hasta esta fecha/hora (menor o igual)
- `Status eq/ne {estado}` - Filtro por estado (eq = igual a, ne = diferente de)

#### 4.3.2. $top (OBLIGATORIO - Máximo: 500) - PAGINACIÓN
Límite de resultados por página:
```
$top=500
```

**⚠️ IMPORTANTE:**
- **OBLIGATORIO:** El valor de `$top` **SIEMPRE** debe ser `500` (máximo permitido)
- **No se permite** usar otro valor diferente a 500
- Todas las consultas **DEBEN** incluir este parámetro con el valor 500
- Este es un requisito técnico para proteger la base de datos productiva

#### 4.3.3. $skip (OBLIGATORIO) - PAGINACIÓN
Número de resultados a omitir (para paginación):
```
$skip=0
```

**⚠️ IMPORTANTE:**
- **OBLIGATORIO:** Todas las consultas **DEBEN** incluir `$skip` para paginación
- No se permite realizar consultas sin paginación
- Este parámetro es esencial para navegar entre páginas

**Uso para paginación:**
- Primera página: `$skip=0`, `$top=500`
- Segunda página: `$skip=500`, `$top=500`
- Tercera página: `$skip=1000`, `$top=500`
- Cuarta página: `$skip=1500`, `$top=500`
- Fórmula: `$skip = (número_página - 1) * 500`

**Ejemplo de paginación completa:**
```
# Página 1
GET /ThirdPartyService/Appointments?$filter=...&$top=500&$skip=0

# Página 2
GET /ThirdPartyService/Appointments?$filter=...&$top=500&$skip=500

# Página 3
GET /ThirdPartyService/Appointments?$filter=...&$top=500&$skip=500
```

#### 4.3.4. $orderby (Opcional pero Recomendado)
Ordenamiento de resultados:
```
$orderby=DateTimeFrom asc
```

**Opciones:**
- `asc` - Ascendente (recomendado para fechas)
- `desc` - Descendente

**Recomendación:** Usar `$orderby=DateTimeFrom asc` para garantizar consistencia en la paginación.

#### 4.3.5. $select (Opcional)
Seleccionar campos específicos:
```
$select=Id,DateTimeFrom,DateTimeTo,Status,PatientId,ResourceId,CenterId
```

#### 4.3.6. $expand (Opcional)
Expandir relaciones:
```
$expand=Patient,ResourceAppointments
```

**Límite:** MaxExpansionDepth = 2

### 4.4. URL Completa de Ejemplo con Paginación

#### 4.4.1. Primera Página (con paginación)

**URL codificada:**
```
https://proxy.megasalud.cl/ThirdPartyService/Appointments?%24filter=DateTimeFrom%20ge%202025-12-15T21%3A48%3A36-03%3A00%20and%20DateTimeTo%20le%202026-02-13T21%3A48%3A36-03%3A00%20and%20Status%20ne%20WebApiModel.Enum.AppointmentStatus%27Cancelled%27&%24orderby=DateTimeFrom%20asc&%24top=500&%24skip=0
```

**URL decodificada (para referencia):**
```
https://proxy.megasalud.cl/ThirdPartyService/Appointments?
  $filter=DateTimeFrom ge 2025-12-15T21:48:36-03:00 
    and DateTimeTo le 2026-02-13T21:48:36-03:00 
    and Status ne WebApiModel.Enum.AppointmentStatus'Cancelled'
  &$orderby=DateTimeFrom asc
  &$top=500
  &$skip=0
```

#### 4.4.2. Segunda Página (con paginación)

**URL decodificada:**
```
https://proxy.megasalud.cl/ThirdPartyService/Appointments?
  $filter=DateTimeFrom ge 2025-12-15T21:48:36-03:00 
    and DateTimeTo le 2026-02-13T21:48:36-03:00 
    and Status ne WebApiModel.Enum.AppointmentStatus'Cancelled'
  &$orderby=DateTimeFrom asc
  &$top=500
  &$skip=500
```

#### 4.4.3. Tercera Página (con paginación)

**URL decodificada:**
```
https://proxy.megasalud.cl/ThirdPartyService/Appointments?
  $filter=DateTimeFrom ge 2025-12-15T21:48:36-03:00 
    and DateTimeTo le 2026-02-13T21:48:36-03:00 
    and Status ne WebApiModel.Enum.AppointmentStatus'Cancelled'
  &$orderby=DateTimeFrom asc
  &$top=500
  &$skip=1000
```

**⚠️ NOTAS IMPORTANTES:**
- El valor de `$top=500` es **obligatorio y máximo permitido**. No se permite usar otro valor.
- El parámetro `$skip` es **obligatorio** y debe incrementarse en múltiplos de 500 para navegar entre páginas.
- **Todas las consultas DEBEN incluir paginación** (`$top` y `$skip`).

---

## 5. Paginación Obligatoria

### 5.1. Estrategia de Paginación
**⚠️ IMPORTANTE:** Este endpoint **REQUIERE** paginación en todas las consultas. No se permite realizar consultas sin paginación.

La API utiliza paginación basada en `$top` y `$skip` (offset-based pagination):
- `$top=500` - Tamaño máximo de página (obligatorio, máximo permitido: 500)
- `$skip` - Offset para navegar entre páginas (obligatorio)

### 5.2. Implementación Recomendada

#### Paso 1: Primera Página
```bash
GET /ThirdPartyService/Appointments?
  $filter=DateTimeFrom ge {fecha_inicio} and DateTimeTo le {fecha_fin} and Status ne WebApiModel.Enum.AppointmentStatus'Cancelled'
  &$orderby=DateTimeFrom asc
  &$top=500
  &$skip=0
```

**⚠️ IMPORTANTE:** El valor `$top=500` es obligatorio y es el máximo permitido. No se permite usar otro valor.

#### Paso 2: Páginas Siguientes
Incrementar `$skip` en múltiplos de 500:
- Página 1: `$skip=0`, `$top=500`
- Página 2: `$skip=500`, `$top=500`
- Página 3: `$skip=1000`, `$top=500`
- Página N: `$skip=(N-1)*500`, `$top=500`

#### Paso 3: Detectar Fin de Datos
Cuando la respuesta contiene menos de 500 resultados, se ha alcanzado la última página.

### 5.3. Ejemplo de Implementación

#### JavaScript
```javascript
async function getAllConfirmedAppointments(dateFrom, dateTo) {
  const baseUrl = 'https://proxy.megasalud.cl/ThirdPartyService/Appointments';
  const token = 'TU_TOKEN_AQUI';
  const pageSize = 500; // OBLIGATORIO: Máximo permitido es 500
  let skip = 0;
  let allAppointments = [];
  let hasMore = true;

  const filter = `DateTimeFrom ge ${dateFrom} and DateTimeTo le ${dateTo} and Status ne WebApiModel.Enum.AppointmentStatus'Cancelled'`;

  while (hasMore) {
    const params = new URLSearchParams({
      '$filter': filter,
      '$orderby': 'DateTimeFrom asc',
      '$top': '500', // OBLIGATORIO: Máximo permitido es 500
      '$skip': skip.toString()
    });

    const response = await fetch(`${baseUrl}?${params}`, {
      headers: {
        'Authorization': `Bearer ${token}`,
        'X-AppTimezone': '-240',
        'Accept': 'application/json'
      }
    });

    const data = await response.json();
    const appointments = data.value || [];

    allAppointments = allAppointments.concat(appointments);

    // Si hay menos resultados que el pageSize, es la última página
    hasMore = appointments.length === pageSize;
    skip += pageSize;
  }

  return allAppointments;
}

// Uso
const appointments = await getAllConfirmedAppointments(
  '2025-12-15T21:48:36-03:00',
  '2026-02-13T21:48:36-03:00'
);
```

#### Python
```python
import requests
from typing import List, Dict

def get_all_confirmed_appointments(date_from: str, date_to: str, token: str) -> List[Dict]:
    base_url = "https://proxy.megasalud.cl/ThirdPartyService/Appointments"
    page_size = 500  # OBLIGATORIO: Máximo permitido es 500
    skip = 0
    all_appointments = []
    has_more = True

    filter_query = f"DateTimeFrom ge {date_from} and DateTimeTo le {date_to} and Status ne WebApiModel.Enum.AppointmentStatus'Cancelled'"

    headers = {
        "Authorization": f"Bearer {token}",
        "X-AppTimezone": "-240",
        "Accept": "application/json"
    }

    while has_more:
        params = {
            "$filter": filter_query,
            "$orderby": "DateTimeFrom asc",
            "$top": 500,  # OBLIGATORIO: Máximo permitido es 500
            "$skip": skip
        }

        response = requests.get(base_url, headers=headers, params=params)
        response.raise_for_status()
        
        data = response.json()
        appointments = data.get("value", [])

        all_appointments.extend(appointments)

        # Si hay menos resultados que page_size, es la última página
        has_more = len(appointments) == page_size
        skip += page_size

    return all_appointments

# Uso
appointments = get_all_confirmed_appointments(
    "2025-12-15T21:48:36-03:00",
    "2026-02-13T21:48:36-03:00",
    "TU_TOKEN_AQUI"
)
```

#### cURL (Primera Página)
```bash
curl --location 'https://proxy.megasalud.cl/ThirdPartyService/Appointments?%24filter=DateTimeFrom%20ge%202025-12-15T21%3A48%3A36-03%3A00%20and%20DateTimeTo%20le%202026-02-13T21%3A48%3A36-03%3A00%20and%20Status%20ne%20WebApiModel.Enum.AppointmentStatus%27Cancelled%27&%24orderby=DateTimeFrom%20asc&%24top=500&%24skip=0' \
--header 'Authorization: Bearer {TU_TOKEN}' \
--header 'X-AppTimezone: -240' \
--header 'Accept: application/json'
```

#### cURL (Segunda Página)
```bash
curl --location 'https://proxy.megasalud.cl/ThirdPartyService/Appointments?%24filter=DateTimeFrom%20ge%202025-12-15T21%3A48%3A36-03%3A00%20and%20DateTimeTo%20le%202026-02-13T21%3A48%3A36-03%3A00%20and%20Status%20ne%20WebApiModel.Enum.AppointmentStatus%27Cancelled%27&%24orderby=DateTimeFrom%20asc&%24top=500&%24skip=500' \
--header 'Authorization: Bearer {TU_TOKEN}' \
--header 'X-AppTimezone: -240' \
--header 'Accept: application/json'
```

### 5.4. Mejores Prácticas de Paginación

1. **Tamaño de página:** ⚠️ **OBLIGATORIO:** Usar `$top=500` (máximo permitido, no se permite otro valor)
2. **Ordenamiento:** Siempre usar `$orderby=DateTimeFrom asc` para garantizar consistencia
3. **Manejo de errores:** Implementar retry logic para manejar errores temporales
4. **Ventana de ejecución:** ⚠️ **OBLIGATORIO:** Ejecutar consultas después de las 03:00 AM
5. **Validación:** Verificar que `$skip` no exceda el total de resultados esperados
6. **Pruebas:** Realizar pruebas de estrés en ambiente QA antes de producción

---

## 6. Respuesta de la API

### 6.1. Formato de Respuesta
```json
{
  "@odata.context": "https://proxy.megasalud.cl/ThirdPartyService/$metadata#Appointments",
  "value": [
    {
      "@odata.etag": "W/\"J0FBQUFBQmpMWmM0PSc=\"",
      "Id": "7b21fa47-f65b-4385-b82d-a61e00f0ec87",
      "DateTimeFrom": "2016-06-07T17:00:00-03:00",
      "DateTimeTo": "2016-06-07T17:12:00-03:00",
      "DateTimeFromUTC": "2016-06-07T20:00:00Z",
      "DateTimeToUTC": "2016-06-07T20:12:00Z",
      "Status": "Confirmed",
      "Duration": 12,
      "ResourceId": "d19eb039-180b-4f16-a02a-a5fa002a53ac",
      "ServiceId": "b37110ae-09cc-4432-94e5-a5f8004ed15c",
      "CoveragePlanId": "f2d3cd94-b91c-420e-9ec0-a5f800ca8dbc",
      "PatientId": "d714a336-e7ad-4c55-92bf-a61e00e25ae1",
      "CenterId": "4d4c071b-05b5-4773-8a55-a5f8002127fe",
      "AppointmentTypeId": "4139651e-c3c2-4839-97ef-cdd5428b9975",
      "IsOvercapacity": false,
      "Index": 1,
      "NumberOfSlots": 1,
      "CommentsCount": 0,
      "Archived": false,
      "Cancelled": false,
      "ModifiedOn": "2016-06-07T14:37:10.6335273Z",
      "CreatedOn": "2016-06-07T14:37:10.5696009Z",
      "RowVersion": "AAAAABjLZc4="
    }
  ]
}
```

### 6.2. Campos Principales

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `Id` | Guid | Identificador único de la cita |
| `DateTimeFrom` | DateTimeOffset | Fecha y hora de inicio de la cita (timezone local) |
| `DateTimeTo` | DateTimeOffset | Fecha y hora de fin de la cita (timezone local) |
| `DateTimeFromUTC` | DateTimeOffset | Fecha y hora de inicio en UTC (solo lectura) |
| `DateTimeToUTC` | DateTimeOffset | Fecha y hora de fin en UTC (solo lectura) |
| `Status` | String | Estado de la cita (siempre "Confirmed" en este caso) |
| `Duration` | Int | Duración de la cita en minutos |
| `PatientId` | Guid | Identificador del paciente |
| `ResourceId` | Guid | Identificador del recurso (médico/equipo) |
| `CenterId` | Guid | Identificador del centro médico |
| `ServiceId` | Guid | Identificador del servicio |
| `AppointmentTypeId` | Guid | Identificador del tipo de cita |
| `CoveragePlanId` | Guid? | Identificador del plan de cobertura (opcional) |
| `RowVersion` | String | Versión para control de concurrencia optimista |

### 6.3. Detección de Última Página

**Criterio:** Si `value.length < $top`, entonces es la última página.

**Ejemplo:**
```javascript
const isLastPage = data.value.length < pageSize;
```

---

## 7. Códigos de Estado HTTP

| Código | Descripción | Acción |
|--------|-------------|--------|
| `200 OK` | Solicitud exitosa | Procesar resultados |
| `400 Bad Request` | Solicitud inválida (sintaxis OData incorrecta) | Revisar parámetros de query |
| `401 Unauthorized` | Token inválido o expirado | Renovar token |
| `403 Forbidden` | Sin permisos para el recurso | Contactar administrador |
| `500 Internal Server Error` | Error del servidor | Reintentar después de un delay |

---

## 8. Manejo de Errores

### 8.1. Formato de Error OData
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

### 8.2. Errores Comunes

#### Error: "A binary operator with incompatible types was detected"
**Causa:** Sintaxis incorrecta de enum en el filtro  
**Solución:** Usar `Status ne WebApiModel.Enum.AppointmentStatus'Cancelled'` para excluir citas anuladas

#### Error: "The query specified in the URI is not valid"
**Causa:** Sintaxis OData incorrecta  
**Solución:** Verificar formato de fecha y sintaxis del filtro

#### Error: "401 Unauthorized"
**Causa:** Token inválido o expirado  
**Solución:** Obtener nuevo token

---

## 9. Ejemplo Completo: cURL

### 9.1. Primera Página
```bash
curl --location 'https://proxy.megasalud.cl/ThirdPartyService/Appointments?%24filter=DateTimeFrom%20ge%202025-12-15T21%3A48%3A36-03%3A00%20and%20DateTimeTo%20le%202026-02-13T21%3A48%3A36-03%3A00%20and%20Status%20ne%20WebApiModel.Enum.AppointmentStatus%27Cancelled%27&%24orderby=DateTimeFrom%20asc&%24top=500&%24skip=0' \
--header 'Authorization: Bearer Tj17YiZYaDx5OFMzWl4iKw==' \
--header 'X-AppTimezone: -240' \
--header 'Accept: application/json, text/plain, */*' \
--header 'User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36'
```

**⚠️ IMPORTANTE:** El valor `$top=500` es obligatorio y es el máximo permitido. No se permite usar otro valor.

### 9.2. Segunda Página
```bash
curl --location 'https://proxy.megasalud.cl/ThirdPartyService/Appointments?%24filter=DateTimeFrom%20ge%202025-12-15T21%3A48%3A36-03%3A00%20and%20DateTimeTo%20le%202026-02-13T21%3A48%3A36-03%3A00%20and%20Status%20ne%20WebApiModel.Enum.AppointmentStatus%27Cancelled%27&%24orderby=DateTimeFrom%20asc&%24top=500&%24skip=500' \
--header 'Authorization: Bearer Tj17YiZYaDx5OFMzWl4iKw==' \
--header 'X-AppTimezone: -240' \
--header 'Accept: application/json, text/plain, */*'
```

---

## 10. Valores de Enum

### 10.1. AppointmentStatus - Explicación Completa

**⚠️ IMPORTANTE:** El tope máximo de consulta es **500 resultados por página** (`$top=500`).

| Valor en API | Descripción en Español | Valor Numérico | Descripción |
|--------------|------------------------|----------------|-------------|
| `Booked` | **Agendado** | 0 | Citas que han sido agendadas pero aún no confirmadas |
| `Confirmed` | **Confirmado** | 1 | Citas que han sido confirmadas |
| `CheckedIn` | **Presentado** | 3 | Citas donde el paciente se ha presentado (check-in realizado) |
| `ServicePerformed` | **Atendido** | 4 | Citas donde el servicio ya fue realizado/atendido |
| `Blocked` | **Bloqueado** | 5 | Citas que están bloqueadas |
| `NotPerformed` | **No Presentado** | 6 | Citas donde el paciente no se presentó |
| `Cancelled` | **Anulado** | 2 | Citas que han sido anuladas (excluidas del requerimiento principal) |

### 10.2. Modos de Consulta y Sintaxis en Filtros

#### Modo 1: Consultar Todos los Estados (Excepto Anuladas)

**Sintaxis:**
```
Status ne WebApiModel.Enum.AppointmentStatus'Cancelled'
```

**Explicación:**
- `ne` = "not equal" (diferente de / no igual a)
- Esto EXCLUYE las citas anuladas
- Incluye automáticamente: Booked, Confirmed, CheckedIn, ServicePerformed, Blocked, NotPerformed

**Ejemplo completo con paginación:**
```
$filter=DateTimeFrom ge 2025-12-15T21:48:36-03:00 and DateTimeTo le 2026-02-13T21:48:36-03:00 and Status ne WebApiModel.Enum.AppointmentStatus'Cancelled'&$top=500&$skip=0
```

#### Modo 2: Consultar por Estado Individual

**Sintaxis para cada estado (siempre con `$top=500` y `$skip`):**

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

**Ejemplo completo para un estado específico:**
```
$filter=DateTimeFrom ge 2025-12-15T21:48:36-03:00 and DateTimeTo le 2026-02-13T21:48:36-03:00 and Status eq WebApiModel.Enum.AppointmentStatus'Confirmed'&$top=500&$skip=0
```

**Operadores:**
- `eq` = "equal" (igual a) - Para filtrar por un estado específico
- `ne` = "not equal" (diferente de) - Para excluir un estado

**Importante:** 
- La sintaxis completamente calificada es requerida para filtros de enum
- Siempre incluir `$top=500` (máximo permitido) y `$skip` para paginación

---

## 11. Zonas Horarias

### 11.1. Header X-AppTimezone
El header `X-AppTimezone` especifica el offset de zona horaria en minutos desde UTC.

| Zona | Offset (minutos) | Header Value |
|------|------------------|--------------|
| UTC-4 (Chile continental) | -240 | `-240` |
| UTC-3 (Chile insular) | -180 | `-180` |
| UTC | 0 | `0` |

### 11.2. Formato de Fechas
- **En queries:** Usar timezone local (ej: `-03:00`)
- **En respuestas:** Se incluyen tanto `DateTimeFrom`/`DateTimeTo` (local) como `DateTimeFromUTC`/`DateTimeToUTC` (UTC)

---

## 12. Límites y Restricciones

| Límite | Valor | Descripción |
|--------|-------|-------------|
| **$top (OBLIGATORIO)** | 500 | Máximo permitido para todas las consultas |
| **$skip (OBLIGATORIO)** | Variable | Obligatorio para paginación (0, 500, 1000, 1500, ...) |
| **MaxExpansionDepth** | 2 | Máxima profundidad de expansión de relaciones |
| **Ventana de Ejecución** | Después de 03:00 AM | Horario obligatorio para ejecutar consultas |
| **Timeout** | Variable | Timeout de conexión (consultar con el equipo) |

**⚠️ IMPORTANTE:** 
- El valor de `$top` **SIEMPRE** debe ser 500 (máximo permitido). No se permite otro valor.
- Todas las consultas **DEBEN** ejecutarse después de las 03:00 AM.
- Se **DEBEN** realizar pruebas de estrés en QA antes de producción.

---

## 13. Consideraciones Técnicas Adicionales

**⚠️ IMPORTANTE:** Antes de implementar en producción, revisar la sección 15 "Consideraciones Técnicas Obligatorias" que contiene el checklist completo de requisitos.

## 14. Roadmap y Requisitos para Disponibilización

### 14.1. Requisitos Críticos Antes de Disponibilizar

**⚠️ ESTAS RUTAS NO ESTÁN DISPONIBLES PARA PROVEEDORES HASTA COMPLETAR:**

**Responsable:** Pedro Wittig

**Requisitos obligatorios:**
1. **Enmascaramiento mediante API Gateway** ⚠️ **CRÍTICO**
   - Las rutas DEBEN ser enmascaradas por API Gateway
   - El endpoint actual NO debe exponerse directamente a proveedores externos
   - El API Gateway actúa como capa de seguridad y control

2. **Certificaciones de Calidad Completas** ⚠️ **CRÍTICO**
   - Pasar TODAS las certificaciones de calidad
   - Incluye: pruebas de estrés, seguridad, performance, validaciones funcionales
   - No se permite disponibilizar sin completar todas las certificaciones

3. **Ruta Segura Dedicada:** Nuevo endpoint específico para proveedores externos (a través del API Gateway)
4. **Credenciales Diferenciadas:** Sistema de autenticación separado para proveedores
5. **Rate Limiting:** Límites de requests por minuto/hora configurados en el API Gateway
6. **Mejor Auditoría:** Logging específico para acceso de proveedores
7. **Documentación Actualizada:** Esta documentación será actualizada con las nuevas rutas del API Gateway y credenciales

**Nota:** Una vez implementado el API Gateway y completadas todas las certificaciones de calidad, se notificará a los proveedores y se actualizará esta documentación con las nuevas rutas y credenciales.

---

## 15. Consideraciones Técnicas Obligatorias

### 15.1. Checklist Antes de Disponibilizar para Proveedores

⚠️ **REQUISITOS OBLIGATORIOS QUE DEBEN CUMPLIRSE:**

**Requisitos Críticos del Sistema:**
- [ ] **API Gateway Implementado:** Las rutas están enmascaradas mediante API Gateway
- [ ] **Certificaciones de Calidad:** TODAS las certificaciones de calidad han sido completadas y aprobadas
- [ ] **Ruta Segura Disponible:** Ruta segura con credenciales diferenciadas implementada

**Requisitos de Integración:**
- [ ] **Pruebas de Estrés en QA:** El proceso ha pasado pruebas de calidad de estrés en ambiente QA
- [ ] **Ventana de Ejecución:** El proceso está configurado para ejecutarse después de las 03:00 AM
- [ ] **Paginación Implementada:** Todas las consultas usan `$top=500` (máximo permitido)
- [ ] **Confirmación con Operaciones:** Se ha confirmado con el equipo de operaciones si hay definiciones adicionales

### 15.2. Detalles de los Requisitos

#### 15.2.1. Enmascaramiento mediante API Gateway
- **Obligatorio:** Las rutas DEBEN ser enmascaradas por API Gateway antes de disponibilizar
- **Objetivo:** Proteger la infraestructura, controlar el acceso y proporcionar capa de seguridad
- **Estado:** Pendiente de implementación
- **Responsable:** Pedro Wittig
- **Contacto:** Coordinar con el equipo de operaciones y Pedro Wittig

#### 15.2.2. Certificaciones de Calidad
- **Obligatorio:** Pasar TODAS las certificaciones de calidad antes de disponibilizar
- **Incluye:** Pruebas de estrés, seguridad, performance, validaciones funcionales
- **Objetivo:** Asegurar calidad, seguridad y estabilidad del servicio
- **Estado:** Pendiente de completar
- **Contacto:** Coordinar con el equipo de operaciones para las certificaciones

#### 15.2.3. Pruebas de Estrés en QA
- **Obligatorio:** Realizar pruebas de carga y estrés en ambiente QA antes de desplegar a producción
- **Objetivo:** Asegurar que el proceso no afecte la base de datos productiva
- **Contacto:** Coordinar con el equipo de operaciones para las pruebas

#### 15.2.4. Ventana de Ejecución (Después de 03:00 AM)
- **Obligatorio:** Todas las consultas DEBEN ejecutarse después de las 03:00 AM (horario local)
- **Razón:** Evitar impacto en la carga de la base de datos durante horarios de alta demanda
- **Implementación:** Configurar el proceso/programador de tareas para ejecutar después de las 03:00 AM

#### 15.2.5. Paginación Obligatoria ($top=500)
- **Obligatorio:** El parámetro `$top` SIEMPRE debe ser `500` (máximo permitido)
- **No se permite:** Usar otro valor diferente a 500
- **Razón:** Proteger la base de datos productiva limitando el tamaño de las respuestas

#### 15.2.6. Confirmación con Operaciones
- **Recomendado:** Contactar al equipo de operaciones para verificar si hay definiciones adicionales a considerar
- **Contacto:** A través del equipo de integración de RedSalud

## 16. Contacto y Soporte

### 15.1. Para Obtener Credenciales
Contactar al equipo de integración de RedSalud.

### 15.2. Para Reportar Problemas
- Contactar al equipo de integración de RedSalud
- Revisar la sección "Manejo de Errores" (sección 8) para errores comunes y soluciones

### 15.3. Para Consultas Técnicas
- **Responsable de integración:** Pedro Wittig
- **Equipo técnico:** Contactar a través del equipo de integración de RedSalud

### 15.4. Para Coordinar Pruebas de Estrés
- Contactar al equipo de operaciones a través del equipo de integración de RedSalud

---

## 17. Changelog

| Fecha | Versión | Cambios |
|-------|---------|---------|
| 2025-01-XX | 1.0 | Documentación inicial - Endpoint temporal con credenciales actuales |
| TBD | 2.0 | Implementación de ruta segura con credenciales diferenciadas (Pedro Wittig) |

---

**Última actualización:** 2025-01-XX  
**Versión de la documentación:** 1.0  
**Estado:** Endpoint temporal - Mejoras planificadas para 1 sprint

