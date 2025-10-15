# Proceso de Borrado Lógico de Usuarios en Salesforce

## Descripción del Requerimiento

Este documento describe el proceso para implementar un borrado lógico de usuarios en Salesforce, donde:
1. Al eliminar un usuario, se realiza un borrado lógico (el registro permanece en la base de datos pero se marca como inactivo)
2. Después de 30 días, los datos del usuario se eliminan definitivamente de la base de datos

## Soluciones Propuestas

A continuación se presentan tres posibles soluciones para implementar este requerimiento en Salesforce.

### Solución 1: Campos Personalizados + Process Builder + Batch Apex

#### Componentes:
- **Campos personalizados en el objeto User**:
  - `Is_Logically_Deleted__c` (Checkbox)
  - `Logical_Deletion_Date__c` (Date/Time)
- **Process Builder** para capturar eventos de desactivación
- **Batch Apex** programado para eliminar definitivamente

#### Implementación:

1. **Proceso de Borrado Lógico**:
   ```mermaid
   graph TD
      A[Usuario solicita eliminación] --> B[Trigger Before Delete en User]
      B --> C{Es administrador?}
      C -->|No| D[Cancelar eliminación estándar]
      C -->|Sí| D
      D --> E[Marcar Is_Logically_Deleted__c = true]
      E --> F[Registrar Logical_Deletion_Date__c = HOY]
      F --> G[Desactivar usuario]
   ```

2. **Proceso de Eliminación Definitiva**:
   ```mermaid
   graph TD
      A[Batch Apex programado diariamente] --> B[Consultar usuarios con Is_Logically_Deleted__c = true]
      B --> C[Filtrar usuarios con Logical_Deletion_Date__c > 30 días]
      C --> D[Para cada usuario]
      D --> E[Eliminar registros relacionados]
      E --> F[Eliminar definitivamente el usuario]
   ```

#### Ventajas:
- Implementación relativamente sencilla
- No requiere componentes externos
- Utiliza herramientas declarativas (Process Builder) y programáticas (Apex)

#### Desventajas:
- Requiere mantenimiento del código Batch Apex
- Posibles problemas con límites del gobernador para grandes volúmenes de datos

### Solución 2: Platform Events + Flow + Scheduled Actions

#### Componentes:
- **Campo personalizado en User**: `Logical_Deletion_Date__c` (Date/Time)
- **Platform Event**: `User_Deletion_Event__c`
- **Flow** para procesar eventos de eliminación
- **Scheduled Actions** para la eliminación definitiva

#### Implementación:

1. **Proceso de Borrado Lógico**:
   ```mermaid
   graph TD
      A[Usuario solicita eliminación] --> B[UI personalizada o botón]
      B --> C[Publicar Platform Event User_Deletion_Event__c]
      C --> D[Flow suscrito al evento]
      D --> E[Marcar Logical_Deletion_Date__c = HOY]
      E --> F[Desactivar usuario]
   ```

2. **Proceso de Eliminación Definitiva**:
   ```mermaid
   graph TD
      A[Scheduled Action configurada para ejecutarse después de 30 días] --> B[Flow de eliminación]
      B --> C[Verificar que han pasado 30 días]
      C --> D[Eliminar registros relacionados]
      D --> E[Eliminar definitivamente el usuario]
   ```

#### Ventajas:
- Arquitectura orientada a eventos
- Mayor escalabilidad
- Menor código personalizado (más declarativo)
- Mejor manejo de errores y reintentos

#### Desventajas:
- Mayor complejidad en la configuración inicial
- Requiere monitoreo de los Platform Events

### Solución 3: Custom Metadata + Apex API + Einstein Automation

#### Componentes:
- **Custom Metadata Type**: `User_Deletion_Settings__mdt` para configuración
- **Apex API personalizada** para gestionar el proceso de eliminación
- **Einstein Automation** (anteriormente Einstein Next Best Action) para orquestar el proceso

#### Implementación:

1. **Proceso de Borrado Lógico**:
   ```mermaid
   graph TD
      A[Solicitud a través de API personalizada] --> B[Apex Controller valida permisos]
      B --> C[Registrar en objeto User_Deletion_Log__c]
      C --> D[Marcar usuario como inactivo]
      D --> E[Einstein Automation inicia seguimiento]
   ```

2. **Proceso de Eliminación Definitiva**:
   ```mermaid
   graph TD
      A[Einstein Automation monitorea tiempo transcurrido] --> B{Han pasado 30 días?}
      B -->|No| A
      B -->|Sí| C[Invocar Apex API de eliminación]
      C --> D[Ejecutar proceso de anonimización]
      D --> E[Eliminar definitivamente o archivar]
   ```

#### Ventajas:
- Altamente configurable mediante Custom Metadata
- API bien definida para integraciones
- Proceso robusto con capacidades avanzadas de orquestación
- Mejor auditoría y cumplimiento normativo

#### Desventajas:
- Mayor complejidad técnica
- Requiere licencias adicionales para Einstein Automation
- Mayor esfuerzo de desarrollo inicial

## Consideraciones Adicionales

### Aspectos de Seguridad y Cumplimiento
- Asegurar que el proceso cumple con regulaciones como GDPR, CCPA, etc.
- Implementar un registro de auditoría completo del proceso
- Considerar la encriptación de datos sensibles durante el período de retención

### Impacto en Integraciones
- Evaluar el impacto en sistemas integrados que dependen de datos de usuario
- Establecer mecanismos para notificar a sistemas externos sobre el borrado lógico

### Recomendación Final

Basado en el análisis de las tres soluciones, la **Solución 2: Platform Events + Flow + Scheduled Actions** ofrece el mejor equilibrio entre:
- Facilidad de implementación
- Mantenibilidad a largo plazo
- Escalabilidad
- Menor código personalizado

Esta solución aprovecha las capacidades nativas de Salesforce y sigue las mejores prácticas de arquitectura orientada a eventos, lo que la hace más robusta y adaptable a futuros cambios en los requisitos.