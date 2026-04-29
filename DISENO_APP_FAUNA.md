# Diseño de aplicación: Toma de datos de terreno (línea base y monitoreo de fauna)

## 1) Objetivo
Diseñar una aplicación que permita planificar campañas de terreno y registrar observaciones de fauna, manteniendo trazabilidad entre:

- **Proyecto**
- **Campaña** (línea base o monitoreo)
- **Estación de muestreo**
- **Metodología aplicada**
- **Registros de especies**
- **Usuarios participantes**

---

## 2) Roles y permisos

### Administrador
- Crear/editar/eliminar proyectos.
- Crear campañas dentro de cada proyecto.
- Definir tipo de campaña: **Línea base** o **Monitoreo**.
- Definir grupo objetivo: **Fauna vertebrada** o **invertebrada**.
- Crear y asignar estaciones de muestreo.
- Asignar usuarios a campañas.
- Habilitar metodologías por estación.
- Revisar/validar registros de especies.

### Usuario de terreno
- Ver campañas asignadas.
- Registrar datos en estaciones asignadas.
- Crear registros de especies en metodologías habilitadas.
- Editar solo sus registros (o según política definida por admin).

---

## 3) Modelo funcional (jerarquía)

```text
Proyecto
 └── Campañas
      ├── Tipo: Línea base | Monitoreo
      ├── Grupo fauna: Vertebrada | Invertebrada
      ├── Usuarios asignados
      └── Estaciones de muestreo
           └── Metodologías (Transecto, Punto de observación de aves, Búsqueda dirigida, etc.)
                └── Registros de especies
```

---

## 4) Modelo de datos propuesto (entidades)

## 4.1. users
- id (uuid)
- nombre
- email (único)
- rol (admin, terreno)
- activo
- created_at, updated_at

## 4.2. projects
- id (uuid)
- nombre
- descripcion
- cliente/mandante (opcional)
- fecha_inicio
- fecha_fin
- estado (planificado, activo, cerrado)
- created_by (FK users)
- created_at, updated_at

## 4.3. campaigns
- id (uuid)
- project_id (FK projects)
- nombre
- tipo_campana (linea_base, monitoreo)
- grupo_fauna (vertebrada, invertebrada)
- fecha_inicio
- fecha_fin
- estado (planificada, en_ejecucion, finalizada)
- created_at, updated_at

## 4.4. campaign_users (tabla puente N:N)
- id (uuid)
- campaign_id (FK campaigns)
- user_id (FK users)
- rol_en_campana (jefe_campana, observador, digitador, etc.)
- created_at

## 4.5. sampling_stations
- id (uuid)
- campaign_id (FK campaigns)
- codigo_estacion
- nombre
- latitud
- longitud
- altitud (opcional)
- habitat (opcional)
- observaciones
- created_at, updated_at

## 4.6. methodologies_catalog (catálogo)
- id (uuid)
- codigo (transecto, punto_aves, busqueda_dirigida, etc.)
- nombre
- descripcion
- activa

## 4.7. station_methodologies
- id (uuid)
- station_id (FK sampling_stations)
- methodology_id (FK methodologies_catalog)
- fecha_hora_inicio
- fecha_hora_fin
- esfuerzo_muestreo (min, horas, km, etc.)
- condiciones_climaticas (opcional)
- observaciones
- created_by (FK users)
- created_at, updated_at

## 4.8. species_catalog (catálogo taxonómico)
- id (uuid)
- reino
- filo
- clase
- orden
- familia
- genero
- especie
- nombre_comun
- codigo_taxonomico (opcional)
- estado_conservacion (opcional)

## 4.9. species_records
- id (uuid)
- station_methodology_id (FK station_methodologies)
- species_id (FK species_catalog)
- fecha_hora_registro
- cantidad
- tipo_registro (visual, auditivo, captura, huella, fecas, etc.)
- comportamiento (opcional)
- evidencia_url (foto/audio) (opcional)
- coordenada_lat (opcional, si difiere de estación)
- coordenada_lon (opcional)
- observaciones
- registrado_por (FK users)
- validado_por (FK users, opcional)
- estado_validacion (pendiente, validado, rechazado)
- created_at, updated_at

---

## 5) Relaciones clave

- **1 Proyecto** tiene **N Campañas**.
- **1 Campaña** tiene **N Estaciones**.
- **1 Campaña** tiene **N Usuarios** (N:N vía `campaign_users`).
- **1 Estación** tiene **N Metodologías**.
- **1 Metodología aplicada en estación** tiene **N Registros de especies**.
- `species_records` referencia a una especie en `species_catalog`.

---

## 6) Flujo principal de uso

1. Admin crea **Proyecto**.
2. Admin crea una o más **Campañas** en el proyecto.
3. Para cada campaña, define:
   - Tipo: línea base / monitoreo.
   - Grupo: vertebrada / invertebrada.
4. Admin asigna **Usuarios** a la campaña.
5. Admin crea **Estaciones de muestreo**.
6. En cada estación, habilita una o más **Metodologías**.
7. Usuario de terreno ingresa **Registros de especies** por metodología.
8. Admin o revisor valida registros.
9. Sistema consolida reportes por proyecto/campaña/metodología/especie.

---

## 7) Estructura sugerida de pantallas (UX)

## 7.1. Dashboard
- Proyectos activos.
- Campañas en ejecución.
- Registros pendientes de validación.
- Indicadores rápidos (n° estaciones, n° registros, n° especies).

## 7.2. Módulo Proyectos
- Lista de proyectos (filtros por estado, fecha, cliente).
- Crear/editar proyecto.
- Vista detalle del proyecto con pestañas:
  - Campañas
  - Equipo
  - Resumen de datos

## 7.3. Módulo Campañas
- Lista de campañas por proyecto.
- Formulario de campaña:
  - Nombre, tipo, grupo fauna, fechas.
- Pestañas internas:
  - Usuarios asignados
  - Estaciones
  - Reportes

## 7.4. Módulo Estaciones
- Tabla + mapa.
- Campos rápidos: código, nombre, coordenadas, hábitat.
- Botón “Agregar metodología”.

## 7.5. Módulo Metodologías por estación
- Lista de metodologías activas en la estación.
- Cada metodología abre una vista de esfuerzo + registros.

## 7.6. Módulo Registros de especies
- Formulario optimizado para terreno (móvil):
  - Especie (autocompletar).
  - Cantidad.
  - Tipo de registro.
  - Hora automática + editable.
  - Evidencia (foto/audio).
- Historial de registros recientes.

## 7.7. Validación y QA
- Bandeja de pendientes.
- Filtros por campaña/usuario/metodología.
- Aprobar/rechazar con comentario.

---

## 8) Recomendaciones técnicas

- Base de datos relacional (PostgreSQL).
- Soporte geográfico con PostGIS (estaciones y registros georreferenciados).
- API REST o GraphQL.
- Modo offline para terreno (sin señal) con sincronización posterior.
- Trazabilidad: auditoría de cambios (quién editó y cuándo).
- Exportación a CSV/Excel y reportes PDF.

---

## 9) MVP sugerido (fase 1)

Alcance mínimo para lanzar:
- Autenticación + roles admin/terreno.
- CRUD de proyectos.
- CRUD de campañas.
- Asignación de usuarios a campañas.
- CRUD de estaciones.
- Asignación de metodologías por estación.
- Ingreso de registros de especies.
- Listado/exportación básica por campaña.

Fase 2:
- Validación avanzada y control de calidad.
- Mapa interactivo completo.
- Modo offline móvil.
- Dashboards e indicadores automáticos.
