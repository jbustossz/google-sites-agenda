# Documentación Técnica de la API - Consulta de Citas Confirmadas

## 📋 Resumen Ejecutivo

Esta documentación proporciona toda la información necesaria para que desarrolladores y proveedores externos puedan consultar **citas confirmadas por rango de fechas** mediante la API OData de RedSalud.

### Objetivo del Requerimiento
Consultar todas las citas con estado `Confirmed` dentro de un rango de fechas específico, con soporte completo de **paginación** para manejar grandes volúmenes de datos.

### Estado Actual
- ✅ **Endpoint disponible:** `https://proxy.megasalud.cl/ThirdPartyService/Appointments`
- ✅ **Funcionalidad:** Consulta de citas confirmadas con paginación
- ⏳ **Mejoras planificadas:** Ruta segura con credenciales diferenciadas (1 sprint - Pedro Wittig)

---

## 📚 Estructura de la Documentación

### 1. [API_TECHNICAL_REFERENCE.md](./API_TECHNICAL_REFERENCE.md) ⭐ **INICIO AQUÍ**
**Documentación técnica completa y detallada**

Contiene:
- ✅ Contexto completo del requerimiento
- ✅ Información de autenticación y headers
- ✅ Especificación detallada del endpoint
- ✅ **Guía completa de paginación** con ejemplos
- ✅ Formato de respuestas y códigos de error
- ✅ Límites y restricciones
- ✅ Roadmap de mejoras futuras

**👤 Para:** Desarrolladores que necesitan entender completamente la API y su implementación.

---

### 2. [API_EXAMPLES.md](./API_EXAMPLES.md)
**Ejemplos prácticos listos para usar**

Contiene:
- ✅ Ejemplos de código en múltiples lenguajes (cURL, JavaScript, Python, C#)
- ✅ **Implementaciones completas de paginación**
- ✅ Ejemplos de diferentes operaciones
- ✅ Código listo para copiar y usar

**👤 Para:** Desarrolladores que necesitan código de ejemplo funcional inmediatamente.

---

### 3. [API_DOCUMENTATION_GUIDE.md](./API_DOCUMENTATION_GUIDE.md)
**Guía de qué documentar y mejores prácticas**

Contiene:
- ✅ Checklist de documentación recomendada
- ✅ Mejores prácticas de documentación
- ✅ Herramientas recomendadas
- ✅ Referencias a especificaciones OData

**👤 Para:** Equipos que quieren mantener o mejorar la documentación.

---

## 🚀 Inicio Rápido

### Paso 1: Obtener Credenciales
Contactar al equipo de integración para obtener un token Bearer.

### Paso 2: Realizar Primera Consulta
```bash
curl --location 'https://proxy.megasalud.cl/ThirdPartyService/Appointments?%24filter=DateTimeFrom%20ge%202025-12-15T21%3A48%3A36-03%3A00%20and%20DateTimeTo%20le%202026-02-13T21%3A48%3A36-03%3A00%20and%20Status%20eq%20WebApiModel.Enum.AppointmentStatus%27Confirmed%27&%24orderby=DateTimeFrom%20asc&%24top=500&%24skip=0' \
--header 'Authorization: Bearer {TU_TOKEN}' \
--header 'X-AppTimezone: -240' \
--header 'Accept: application/json'
```

### Paso 3: Implementar Paginación
Ver ejemplos completos en [API_EXAMPLES.md](./API_EXAMPLES.md) para JavaScript, Python y otros lenguajes.

---

## ⚠️ Consideraciones Técnicas Obligatorias

**ANTES DE IMPLEMENTAR EN PRODUCCIÓN:**

1. **Pruebas de Estrés en QA:** El proceso DEBE pasar pruebas de calidad de estrés en ambiente QA antes de producción
2. **Ventana de Ejecución:** Las consultas DEBEN ejecutarse después de las 03:00 AM (horario local)
3. **Paginación Obligatoria:** El parámetro `$top` SIEMPRE debe ser `500` (máximo permitido)
4. **Confirmar con Operaciones:** Verificar si hay definiciones adicionales a considerar

## 🔑 Puntos Clave

### ✅ Paginación Obligatoria
- **OBLIGATORIO:** Siempre incluir `$top=500` y `$skip={offset}` en las consultas
- **OBLIGATORIO:** El valor de `$top` SIEMPRE debe ser `500` (máximo permitido, no se permite otro)
- Detectar última página cuando `resultados.length < 500`

### ✅ Filtro de Estado
- Sintaxis requerida: `Status eq WebApiModel.Enum.AppointmentStatus'Confirmed'`
- No usar comillas simples alrededor del valor del enum directamente

### ✅ Headers Requeridos
- `Authorization: Bearer {token}` - **OBLIGATORIO**
- `X-AppTimezone: -240` - **OBLIGATORIO** (para UTC-4)

### ✅ Formato de Fechas
- Formato ISO 8601: `YYYY-MM-DDTHH:mm:ss-TZ:tz`
- Ejemplo: `2025-12-15T21:48:36-03:00`

---

## 📊 Endpoint Principal

```
GET /ThirdPartyService/Appointments
```

**Query Parameters:**
- `$filter`: Filtros de fecha y estado
- `$orderby`: Ordenamiento (recomendado: `DateTimeFrom asc`)
- `$top`: Límite de resultados (obligatorio: máximo 500)
- `$skip`: Offset para paginación

**Ejemplo de URL completa:**
```
https://proxy.megasalud.cl/ThirdPartyService/Appointments?
  $filter=DateTimeFrom ge 2025-12-15T21:48:36-03:00 
    and DateTimeTo le 2026-02-13T21:48:36-03:00 
    and Status eq WebApiModel.Enum.AppointmentStatus'Confirmed'
  &$orderby=DateTimeFrom asc
  &$top=500
  &$skip=0
```

---

## 🔄 Roadmap

### Estado Actual (v1.0)
- ✅ Endpoint funcional con credenciales actuales
- ✅ Documentación completa
- ✅ Ejemplos de código con paginación

### Próximas Mejoras (1 Sprint)
- 🔄 **Responsable:** Pedro Wittig
- 🔄 Ruta segura dedicada para proveedores
- 🔄 Credenciales diferenciadas
- 🔄 Rate limiting específico
- 🔄 Mejor auditoría y logging

**Nota:** Una vez implementadas las mejoras, esta documentación será actualizada.

---

## 📞 Soporte

### Para Obtener Credenciales
Contactar al equipo de integración de RedSalud.

### Para Reportar Problemas
- Revisar sección de "Manejo de Errores" en [API_TECHNICAL_REFERENCE.md](./API_TECHNICAL_REFERENCE.md)
- Contactar al equipo técnico

### Para Consultas Técnicas
- **Responsable de integración:** Pedro Wittig
- **Equipo técnico:** Contactar a través del equipo de integración de RedSalud

---

## 📝 Changelog

| Fecha | Versión | Cambios |
|-------|---------|---------|
| 2025-01-XX | 1.0 | Documentación inicial con paginación completa |
| TBD | 2.0 | Actualización con ruta segura (Pedro Wittig) |

---

## 🔗 Enlaces Rápidos

- [📖 Documentación Técnica Completa](./API_TECHNICAL_REFERENCE.md)
- [💻 Ejemplos de Código](./API_EXAMPLES.md)
- [📋 Guía de Documentación](./API_DOCUMENTATION_GUIDE.md)
- [🔍 Metadata OData](https://proxy.megasalud.cl/ThirdPartyService/$metadata)

---

**Última actualización:** 2025-01-XX  
**Versión:** 1.0  
**Mantenido por:** Equipo de Integración RedSalud

