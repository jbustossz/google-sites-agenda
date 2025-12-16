# Ejemplos Prácticos de la API - Consulta de Citas

Este documento contiene ejemplos listos para usar para **consultar citas** mediante la API.

## Contexto del Requerimiento

Este conjunto de ejemplos está enfocado en **consultar todas las citas (excepto anuladas) por rango de fechas** con soporte completo de **paginación**.

**Estados incluidos:** Booked (Agendado), Confirmed (Confirmado), Checked-In (Presentado), Treated (Atendido), Blocked (Bloqueado), Not Treated (No Presentado)

**Estados excluidos:** Cancelled (Anulado) - NO se incluyen en los resultados

**⚠️ IMPORTANTE - Estado Actual:**
- **Endpoint actual:** `https://proxy.megasalud.cl/ThirdPartyService/Appointments`
- **Estado:** NO disponible para proveedores externos hasta completar requisitos críticos
- **Requisitos pendientes:**
  - Enmascaramiento mediante API Gateway (Pedro Wittig)
  - Certificaciones de calidad completas
  - Ruta segura con credenciales diferenciadas

**Nota:** Estos ejemplos muestran la estructura de las consultas. Las rutas finales estarán disponibles a través del API Gateway una vez completados los requisitos de seguridad y certificaciones.

## ⚠️ Consideraciones Técnicas Obligatorias

**ANTES DE IMPLEMENTAR EN PRODUCCIÓN:**

1. **Pruebas de Estrés en QA:** El proceso DEBE pasar pruebas de calidad de estrés en ambiente QA antes de producción
2. **Ventana de Ejecución:** Las consultas DEBEN ejecutarse después de las 03:00 AM
3. **Paginación Obligatoria:** El parámetro `$top` SIEMPRE debe ser `500` (máximo permitido)
4. **Confirmar con Operaciones:** Verificar si hay definiciones adicionales a considerar

## Configuración Base

**URL Base:** `https://proxy.megasalud.cl/ThirdPartyService`

**Headers Requeridos:**
```bash
--header 'Authorization: Bearer {TU_TOKEN_AQUI}'
--header 'X-AppTimezone: -240'
--header 'Accept: application/json'
```

## Ejemplo 1: Obtener Citas Confirmadas en Rango de Fechas (CON PAGINACIÓN)

### cURL - Primera Página
```bash
curl --location 'https://proxy.megasalud.cl/ThirdPartyService/Appointments?%24filter=DateTimeFrom%20ge%202025-12-15T21%3A48%3A36-03%3A00%20and%20DateTimeTo%20le%202026-02-13T21%3A48%3A36-03%3A00%20and%20Status%20eq%20WebApiModel.Enum.AppointmentStatus%27Confirmed%27&%24orderby=DateTimeFrom%20asc&%24top=500&%24skip=0' \
--header 'Authorization: Bearer {TU_TOKEN}' \
--header 'X-AppTimezone: -240' \
--header 'Accept: application/json, text/plain, */*'
```

**⚠️ IMPORTANTE:** 
- El valor `$top=500` es obligatorio y es el máximo permitido.
- **Sobre el filtro:** `Status eq Confirmed` significa "Status **igual a** Confirmed (Confirmado)"
  - `eq` = "equal" (igual a)
  - Esto **INCLUYE** solo las citas con estado Confirmado

### cURL - Segunda Página
```bash
curl --location 'https://proxy.megasalud.cl/ThirdPartyService/Appointments?%24filter=DateTimeFrom%20ge%202025-12-15T21%3A48%3A36-03%3A00%20and%20DateTimeTo%20le%202026-02-13T21%3A48%3A36-03%3A00%20and%20Status%20eq%20WebApiModel.Enum.AppointmentStatus%27Confirmed%27&%24orderby=DateTimeFrom%20asc&%24top=500&%24skip=500' \
--header 'Authorization: Bearer {TU_TOKEN}' \
--header 'X-AppTimezone: -240' \
--header 'Accept: application/json, text/plain, */*'
```

### cURL - Tercera Página
```bash
curl --location 'https://proxy.megasalud.cl/ThirdPartyService/Appointments?%24filter=DateTimeFrom%20ge%202025-12-15T21%3A48%3A36-03%3A00%20and%20DateTimeTo%20le%202026-02-13T21%3A48%3A36-03%3A00%20and%20Status%20eq%20WebApiModel.Enum.AppointmentStatus%27Confirmed%27&%24orderby=DateTimeFrom%20asc&%24top=500&%24skip=1000' \
--header 'Authorization: Bearer {TU_TOKEN}' \
--header 'X-AppTimezone: -240' \
--header 'Accept: application/json, text/plain, */*'
```

### URL Decodificada (para referencia)
```
https://proxy.megasalud.cl/ThirdPartyService/Appointments?
  $filter=DateTimeFrom ge 2025-12-15T21:48:36-03:00 
    and DateTimeTo le 2026-02-13T21:48:36-03:00 
    and Status eq WebApiModel.Enum.AppointmentStatus'Confirmed'
  &$orderby=DateTimeFrom asc
  &$top=500
```

**Explicación del filtro:**
- `DateTimeFrom ge {fecha_inicio}` - Citas desde esta fecha/hora (mayor o igual)
- `DateTimeTo le {fecha_fin}` - Citas hasta esta fecha/hora (menor o igual)
- `Status eq Confirmed` - Estado igual a Confirmed (Confirmado) - Citas que han sido confirmadas

### JavaScript (fetch) - CON PAGINACIÓN COMPLETA
```javascript
async function getAllAppointments(dateFrom, dateTo, token) {
  const baseUrl = 'https://proxy.megasalud.cl/ThirdPartyService/Appointments';
  const pageSize = 500; // OBLIGATORIO: Máximo permitido es 500
  let skip = 0;
  let allAppointments = [];
  let hasMore = true;

  const filter = `DateTimeFrom ge ${dateFrom} and DateTimeTo le ${dateTo} and Status eq WebApiModel.Enum.AppointmentStatus'Confirmed'`;

  while (hasMore) {
    const params = new URLSearchParams({
      '$filter': filter,
      '$orderby': 'DateTimeFrom asc',
      '$top': pageSize.toString(),
      '$skip': skip.toString()
    });

    const response = await fetch(`${baseUrl}?${params}`, {
      headers: {
        'Authorization': `Bearer ${token}`,
        'X-AppTimezone': '-240',
        'Accept': 'application/json'
      }
    });

    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }

    const data = await response.json();
    const appointments = data.value || [];

    allAppointments = allAppointments.concat(appointments);

    // Si hay menos resultados que el pageSize, es la última página
    hasMore = appointments.length === pageSize;
    skip += pageSize;

    console.log(`Página ${Math.floor(skip / pageSize)}: ${appointments.length} citas obtenidas`);
  }

  return allAppointments;
}

// Uso
const appointments = await getAllConfirmedAppointments(
  '2025-12-15T21:48:36-03:00',
  '2026-02-13T21:48:36-03:00',
  'TU_TOKEN_AQUI'
);

console.log(`Total de citas confirmadas: ${appointments.length}`);
```

### C# (.NET)
```csharp
using System;
using System.Net.Http;
using System.Threading.Tasks;

var client = new HttpClient();
client.DefaultRequestHeaders.Add("Authorization", "Bearer TU_TOKEN_AQUI");
client.DefaultRequestHeaders.Add("X-AppTimezone", "-240");
client.DefaultRequestHeaders.Add("Accept", "application/json");

var filter = "DateTimeFrom ge 2025-12-15T21:48:36-03:00 and DateTimeTo le 2026-02-13T21:48:36-03:00 and Status eq WebApiModel.Enum.AppointmentStatus'Confirmed'";
var url = $"https://proxy.megasalud.cl/ThirdPartyService/Appointments?$filter={Uri.EscapeDataString(filter)}&$orderby=DateTimeFrom asc&$top=500"; // OBLIGATORIO: Máximo permitido es 500

var response = await client.GetAsync(url);
var content = await response.Content.ReadAsStringAsync();
Console.WriteLine(content);
```

### Python (requests) - CON PAGINACIÓN COMPLETA
```python
import requests
from typing import List, Dict

def get_all_appointments(date_from: str, date_to: str, token: str) -> List[Dict]:
    """
    Obtiene todas las citas confirmadas en un rango de fechas con paginación automática.
    
    Args:
        date_from: Fecha de inicio en formato ISO 8601 (ej: '2025-12-15T21:48:36-03:00')
        date_to: Fecha de fin en formato ISO 8601 (ej: '2026-02-13T21:48:36-03:00')
        token: Token de autenticación Bearer
    
    Returns:
        Lista de todas las citas confirmadas
    """
    base_url = "https://proxy.megasalud.cl/ThirdPartyService/Appointments"
    page_size = 500  # OBLIGATORIO: Máximo permitido es 500
    skip = 0
    all_appointments = []
    has_more = True

    filter_query = f"DateTimeFrom ge {date_from} and DateTimeTo le {date_to} and Status eq WebApiModel.Enum.AppointmentStatus'Confirmed'"

    headers = {
        "Authorization": f"Bearer {token}",
        "X-AppTimezone": "-240",
        "Accept": "application/json"
    }

    page_number = 1
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
        
        print(f"Página {page_number}: {len(appointments)} citas obtenidas")
        page_number += 1

    return all_appointments

# Uso
appointments = get_all_appointments(
    "2025-12-15T21:48:36-03:00",
    "2026-02-13T21:48:36-03:00",
    "TU_TOKEN_AQUI"
)

print(f"Total de citas (excepto anuladas): {len(appointments)}")
```

## Ejemplo 2: Obtener una Cita por ID (Excepto Anuladas)

### cURL
```bash
curl --location 'https://proxy.megasalud.cl/ThirdPartyService/Appointments({CITA_ID})?$expand=Patient,ResourceAppointments' \
--header 'Authorization: Bearer {TU_TOKEN}' \
--header 'X-AppTimezone: -240' \
--header 'Accept: application/json'
```

### JavaScript
```javascript
const appointmentId = '7b21fa47-f65b-4385-b82d-a61e00f0ec87';
const url = `https://proxy.megasalud.cl/ThirdPartyService/Appointments(${appointmentId})?$expand=Patient,ResourceAppointments`;

fetch(url, {
  headers: {
    'Authorization': `Bearer ${token}`,
    'X-AppTimezone': '-240',
    'Accept': 'application/json'
  }
})
.then(response => response.json())
.then(data => console.log(data));
```

## Ejemplo 3: Modos de Consulta por Estado

**⚠️ IMPORTANTE:** El tope máximo de consulta es **500 resultados por página** (usar `$top=500` en todos los casos).

### 3.1. Consultar Solo Citas Confirmadas

**Filtro:** Estado igual a Confirmed (Confirmado)

```bash
curl --location 'https://proxy.megasalud.cl/ThirdPartyService/Appointments?%24filter=DateTimeFrom%20ge%202025-12-15T21%3A48%3A36-03%3A00%20and%20DateTimeTo%20le%202026-02-13T21%3A48%3A36-03%3A00%20and%20Status%20eq%20WebApiModel.Enum.AppointmentStatus%27Confirmed%27&%24orderby=DateTimeFrom%20asc&%24top=500&%24skip=0' \
--header 'Authorization: Bearer {TU_TOKEN}' \
--header 'X-AppTimezone: -240'
```

**Explicación:** `Status eq Confirmed` significa "Status igual a Confirmed (Confirmado)", por lo tanto INCLUYE solo las citas confirmadas.

### 3.2. Consultar Solo Citas Agendadas (Booked)

```bash
curl --location 'https://proxy.megasalud.cl/ThirdPartyService/Appointments?%24filter=DateTimeFrom%20ge%202025-12-15T21%3A48%3A36-03%3A00%20and%20DateTimeTo%20le%202026-02-13T21%3A48%3A36-03%3A00%20and%20Status%20eq%20WebApiModel.Enum.AppointmentStatus%27Booked%27&%24orderby=DateTimeFrom%20asc&%24top=500&%24skip=0' \
--header 'Authorization: Bearer {TU_TOKEN}' \
--header 'X-AppTimezone: -240'
```

**Estado:** `Booked` = **Agendado** (citas agendadas pero aún no confirmadas)

### 3.3. Consultar Solo Citas Confirmadas (Confirmed)

```bash
curl --location 'https://proxy.megasalud.cl/ThirdPartyService/Appointments?%24filter=DateTimeFrom%20ge%202025-12-15T21%3A48%3A36-03%3A00%20and%20DateTimeTo%20le%202026-02-13T21%3A48%3A36-03%3A00%20and%20Status%20eq%20WebApiModel.Enum.AppointmentStatus%27Confirmed%27&%24orderby=DateTimeFrom%20asc&%24top=500&%24skip=0' \
--header 'Authorization: Bearer {TU_TOKEN}' \
--header 'X-AppTimezone: -240'
```

**Estado:** `Confirmed` = **Confirmado** (citas confirmadas)

### 3.4. Consultar Solo Citas Presentadas (CheckedIn)

```bash
curl --location 'https://proxy.megasalud.cl/ThirdPartyService/Appointments?%24filter=DateTimeFrom%20ge%202025-12-15T21%3A48%3A36-03%3A00%20and%20DateTimeTo%20le%202026-02-13T21%3A48%3A36-03%3A00%20and%20Status%20eq%20WebApiModel.Enum.AppointmentStatus%27CheckedIn%27&%24orderby=DateTimeFrom%20asc&%24top=500&%24skip=0' \
--header 'Authorization: Bearer {TU_TOKEN}' \
--header 'X-AppTimezone: -240'
```

**Estado:** `CheckedIn` = **Presentado** (citas donde el paciente se ha presentado)

### 3.5. Consultar Solo Citas Atendidas (ServicePerformed)

```bash
curl --location 'https://proxy.megasalud.cl/ThirdPartyService/Appointments?%24filter=DateTimeFrom%20ge%202025-12-15T21%3A48%3A36-03%3A00%20and%20DateTimeTo%20le%202026-02-13T21%3A48%3A36-03%3A00%20and%20Status%20eq%20WebApiModel.Enum.AppointmentStatus%27ServicePerformed%27&%24orderby=DateTimeFrom%20asc&%24top=500&%24skip=0' \
--header 'Authorization: Bearer {TU_TOKEN}' \
--header 'X-AppTimezone: -240'
```

**Estado:** `ServicePerformed` = **Atendido** (citas donde el servicio ya fue realizado)

### 3.6. Consultar Solo Citas Bloqueadas (Blocked)

```bash
curl --location 'https://proxy.megasalud.cl/ThirdPartyService/Appointments?%24filter=DateTimeFrom%20ge%202025-12-15T21%3A48%3A36-03%3A00%20and%20DateTimeTo%20le%202026-02-13T21%3A48%3A36-03%3A00%20and%20Status%20eq%20WebApiModel.Enum.AppointmentStatus%27Blocked%27&%24orderby=DateTimeFrom%20asc&%24top=500&%24skip=0' \
--header 'Authorization: Bearer {TU_TOKEN}' \
--header 'X-AppTimezone: -240'
```

**Estado:** `Blocked` = **Bloqueado** (citas bloqueadas)

### 3.7. Consultar Solo Citas No Presentadas (NotPerformed)

```bash
curl --location 'https://proxy.megasalud.cl/ThirdPartyService/Appointments?%24filter=DateTimeFrom%20ge%202025-12-15T21%3A48%3A36-03%3A00%20and%20DateTimeTo%20le%202026-02-13T21%3A48%3A36-03%3A00%20and%20Status%20eq%20WebApiModel.Enum.AppointmentStatus%27NotPerformed%27&%24orderby=DateTimeFrom%20asc&%24top=500&%24skip=0' \
--header 'Authorization: Bearer {TU_TOKEN}' \
--header 'X-AppTimezone: -240'
```

**Estado:** `NotPerformed` = **No Presentado** (citas donde el paciente no se presentó)

### Resumen de Estados

| Valor en API | Descripción en Español | Valor Numérico |
|--------------|------------------------|----------------|
| `Booked` | **Agendado** | 0 |
| `Confirmed` | **Confirmado** | 1 |
| `CheckedIn` | **Presentado** | 3 |
| `ServicePerformed` | **Atendido** | 4 |
| `Blocked` | **Bloqueado** | 5 |
| `NotPerformed` | **No Presentado** | 6 |
| `Cancelled` | **Anulado** | 2 |

## Ejemplo 4: Obtener Citas por Paciente

### cURL
```bash
curl --location 'https://proxy.megasalud.cl/ThirdPartyService/Appointments?%24filter=PatientId%20eq%20{PATIENT_ID}&%24orderby=DateTimeFrom%20desc&%24top=50' \
--header 'Authorization: Bearer {TU_TOKEN}' \
--header 'X-AppTimezone: -240'
```

### URL Decodificada
```
https://proxy.megasalud.cl/ThirdPartyService/Appointments?
  $filter=PatientId eq {PATIENT_ID}
  &$orderby=DateTimeFrom desc
  &$top=50
```

## Ejemplo 5: Obtener Citas por Recurso (Médico)

### cURL
```bash
curl --location 'https://proxy.megasalud.cl/ThirdPartyService/Appointments?%24filter=ResourceAppointments/any(x:%20x/ResourceId%20eq%20{RESOURCE_ID})%20and%20DateTimeFrom%20ge%20{ fecha_inicio }&%24orderby=DateTimeFrom%20asc' \
--header 'Authorization: Bearer {TU_TOKEN}' \
--header 'X-AppTimezone: -240'
```

## Ejemplo 6: Paginación de Resultados - Citas Confirmadas

### Primera página (primeros 500)
```bash
curl --location 'https://proxy.megasalud.cl/ThirdPartyService/Appointments?%24filter=DateTimeFrom%20ge%202025-12-15T21%3A48%3A36-03%3A00%20and%20DateTimeTo%20le%202026-02-13T21%3A48%3A36-03%3A00%20and%20Status%20eq%20WebApiModel.Enum.AppointmentStatus%27Confirmed%27&%24top=500&%24skip=0&%24orderby=DateTimeFrom%20asc' \
--header 'Authorization: Bearer {TU_TOKEN}' \
--header 'X-AppTimezone: -240'
```

**⚠️ IMPORTANTE:** El valor `$top=500` es obligatorio y es el máximo permitido.

### Segunda página (siguientes 500)
```bash
curl --location 'https://proxy.megasalud.cl/ThirdPartyService/Appointments?%24filter=DateTimeFrom%20ge%202025-12-15T21%3A48%3A36-03%3A00%20and%20DateTimeTo%20le%202026-02-13T21%3A48%3A36-03%3A00%20and%20Status%20eq%20WebApiModel.Enum.AppointmentStatus%27Confirmed%27&%24top=500&%24skip=500&%24orderby=DateTimeFrom%20asc' \
--header 'Authorization: Bearer {TU_TOKEN}' \
--header 'X-AppTimezone: -240'
```

### Fórmula para páginas siguientes
- Página 1: `$skip=0`, `$top=500`
- Página 2: `$skip=500`, `$top=500`
- Página 3: `$skip=1000`, `$top=500`
- Página N: `$skip=(N-1)*500`, `$top=500`

**Detectar última página:** Cuando la respuesta contiene menos de 500 resultados.

## Ejemplo 7: Seleccionar Solo Campos Específicos

### cURL
```bash
curl --location 'https://proxy.megasalud.cl/ThirdPartyService/Appointments?%24select=Id,DateTimeFrom,DateTimeTo,Status,PatientId&%24top=100' \
--header 'Authorization: Bearer {TU_TOKEN}' \
--header 'X-AppTimezone: -240'
```

## Ejemplo 8: Expandir Relaciones

### Obtener citas con información del paciente
```bash
curl --location 'https://proxy.megasalud.cl/ThirdPartyService/Appointments?%24expand=Patient&%24top=100' \
--header 'Authorization: Bearer {TU_TOKEN}' \
--header 'X-AppTimezone: -240'
```

### Obtener citas con recursos y pacientes
```bash
curl --location 'https://proxy.megasalud.cl/ThirdPartyService/Appointments?%24expand=Patient,ResourceAppointments($expand=Resource)&%24top=100' \
--header 'Authorization: Bearer {TU_TOKEN}' \
--header 'X-AppTimezone: -240'
```

## Valores de Enums - Estados de Citas

### AppointmentStatus - Explicación Completa

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

### Sintaxis en Filtros

**Modo 1: Excluir citas anuladas (incluye todos los demás estados):**
```
Status eq WebApiModel.Enum.AppointmentStatus'Confirmed'
```

**Modo 2: Consultar por estado específico:**
```
Status eq WebApiModel.Enum.AppointmentStatus'Booked'        # Solo Agendado
Status eq WebApiModel.Enum.AppointmentStatus'Confirmed'     # Solo Confirmado
Status eq WebApiModel.Enum.AppointmentStatus'CheckedIn'      # Solo Presentado
Status eq WebApiModel.Enum.AppointmentStatus'ServicePerformed'  # Solo Atendido
Status eq WebApiModel.Enum.AppointmentStatus'Blocked'       # Solo Bloqueado
Status eq WebApiModel.Enum.AppointmentStatus'NotPerformed'  # Solo No Presentado
```

**Operadores:**
- `eq` = "equal" (igual a)
- `ne` = "not equal" (diferente de / no igual a)

## Zonas Horarias Comunes

| Zona | Offset (minutos) | Header Value |
|------|------------------|--------------|
| UTC-4 (Chile continental) | -240 | `-240` |
| UTC-3 (Chile insular) | -180 | `-180` |
| UTC | 0 | `0` |
| UTC+1 | 60 | `60` |

## Notas Importantes

1. **Autenticación:** Siempre incluir el header `Authorization: Bearer {token}`
2. **Zona Horaria:** El header `X-AppTimezone` es crítico para el manejo correcto de fechas
3. **Paginación:** 
   - **OBLIGATORIO:** Siempre incluir `$skip` y `$top` para paginación
   - **OBLIGATORIO:** El valor de `$top` SIEMPRE debe ser `500` (máximo permitido)
   - Detectar última página cuando `resultados.length < 500`
4. **Límites:** 
   - `$top`: 500 (obligatorio, máximo permitido)
   - MaxExpansionDepth: 2
5. **Ventana de Ejecución:**
   - **OBLIGATORIO:** Ejecutar consultas después de las 03:00 AM
6. **Pruebas:**
   - **OBLIGATORIO:** Realizar pruebas de estrés en QA antes de producción
5. **URL Encoding:** Los espacios y caracteres especiales deben estar codificados en la URL
6. **Enums:** Usar siempre la sintaxis completamente calificada: `WebApiModel.Enum.AppointmentStatus'Value'`
7. **Estado Actual:** 
   - ⚠️ Endpoint NO disponible para proveedores hasta completar:
   - Enmascaramiento mediante API Gateway (Pedro Wittig)
   - Certificaciones de calidad completas
   - Ruta segura con credenciales diferenciadas

## Troubleshooting

### Error: "A binary operator with incompatible types"
- **Causa:** Sintaxis incorrecta de enum
- **Solución:** Usar `Status eq WebApiModel.Enum.AppointmentStatus'Confirmed'`

### Error: "401 Unauthorized"
- **Causa:** Token inválido o expirado
- **Solución:** Verificar y renovar el token

### Error: "The query specified in the URI is not valid"
- **Causa:** Sintaxis OData incorrecta
- **Solución:** Verificar la sintaxis en la documentación OData v4

---

**Última actualización:** 2025-01-XX

