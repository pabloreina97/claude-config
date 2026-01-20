# Task Planning Workflow

Este documento describe el workflow coordinado de tres agentes para planificar, revisar y sincronizar tareas de desarrollo con Notion.

## Descripción general

El workflow involucra cuatro agentes especializados trabajando en secuencia:

```
Solicitud del usuario
    ↓
[1. Task Planner] → Lee lineamientos y genera tareas detalladas
    ↓
[2. Architecture Reviewer] → Verificación técnica final (ajustes menores)
    ↓
[3. Notion Task Creator] → Sincroniza tareas con base de datos de Notion
    ↓
✓ Tareas listas en Notion para asignar e implementar
```

## Los agentes

### 1. Task Planner
**Archivo**: `.claude/agents/task-planner.md`

**Propósito**: Descomponer proyectos/funcionalidades en tareas específicas siguiendo lineamientos

**Proceso**:
- **Lee CLAUDE.md**
- Realiza entrevista interactiva con el usuario
- Pregunta sobre funcionalidad, UI/UX, datos, prioridades
- Genera tareas siguiendo patrones definidos

**Output**: Archivos de tareas en `.claude/docs/tasks/XXX-nombre.md`

**Estructura de cada tarea**:
```markdown
# Título descriptivo

**ID:** 001 (El id será generado por Notion)
**Tipo:** feature
**Prioridad:** alta
**Estado:** pendiente
**Creado:** 2025-10-28

## Descripción
Descripción completa de la tarea...

## Detalles técnicos
### Contexto
Archivos y sistemas involucrados...

### Enfoque recomendado
Estrategia de implementación...

### Dependencias
Otras tareas relacionadas...

## Notas de implementación
### Archivos a modificar/crear
- `src/app/ruta/page.tsx` - Descripción

### Consideraciones clave
- Detalle importante 1

### Desafíos potenciales
- Caso edge anticipado

## Requisitos de testing
Tests necesarios...

## Consideraciones de UI/UX
Requisitos visuales...
```

**Ejemplo de uso**:
```
Usuario: "Quiero planificar la implementación del sistema de turnos"
↓
Task Planner pregunta:
- ¿Qué funcionalidades necesitas?
- ¿Quiénes crean/ven turnos?
- ¿Qué datos se manejan?
- ¿Prioridades?
↓
Genera:
- 001-crear-modelo-turno.md
- 002-implementar-form-turno.md
- 003-crear-vista-calendario.md
- 004-agregar-validaciones.md
- 005-crear-tests.md
```

### 2. Architecture Reviewer
**Archivo**: `.claude/agents/architecture-reviewer.md`

**Propósito**: Validar y mejorar tareas desde una perspectiva técnica de Next.js

**Proceso**:
- Lee todas las tareas generadas
- Verifica decisiones arquitectónicas:
  - ✅ Server vs Client Components
  - ✅ Data fetching patterns
  - ✅ Patrones de formularios
  - ✅ Estructura de archivos
  - ✅ Database operations
  - ✅ Performance considerations
- **Solo modifica lo que necesita mejoras**
- Genera reporte de cambios

**Output**:
- Tareas ajustadas (si es necesario)

**Ejemplo de ajuste**:

**Antes**:
```markdown
### Archivos a modificar/crear
- `src/app/turnos/page.tsx` - Crear página de turnos
```

**Después**:
```markdown
### Archivos a modificar/crear
- `src/app/turnos/page.tsx` - Crear página de turnos (Server Component)
  - Usar acceso directo a Prisma para obtener turnos
  - Importar y usar `getTurnos()` de `lib/queries/turnos.ts`
  - Pasar props a componentes interactivos como Client Components
  - No necesita `'use client'` en la página principal

### Enfoque recomendado
1. **Query function**: Crear `getTurnos()` en `lib/queries/turnos.ts`
   - Usar `cache()` de React para deduplicación
   - Incluir relaciones necesarias (empleado, ubicación)
2. **Server Component**: Implementar page.tsx
   - Fetch data con la query function
   - Renderizar lista de turnos
3. **Client Components**: Solo para:
   - TurnoFilters (necesita useState para filtros)
   - TurnoForm (necesita React Hook Form)
```

### 3. Notion Task Creator
**Archivo**: `.claude/agents/notion-task-creator.md`

**Propósito**: Sincronizar tareas locales con base de datos de Notion

**Proceso**:
- Identifica la base de datos de Notion (búsqueda o ID directo)
- Verifica estructura de propiedades
- Lee archivos de tareas en `.claude/docs/tasks/`
- Parsea metadata y contenido
- Crea páginas en Notion con el MCP
- Maneja duplicados inteligentemente

**Output**: Tareas creadas en Notion con reporte de éxito/errores

**Propiedades mapeadas**:
- **Title** → Título de la tarea
- **ID** (rich_text) → TAREA-XXX
- **Tipo** (select) → feature, bug, mejora, etc.
- **Prioridad** (select) → alta, media, baja
- **Estado** (select) → pendiente, en progreso, completada
- **Descripción** (rich_text) → Contenido completo o resumen

**Ejemplo de uso**:
```
Usuario: "Sube las tareas a Notion"
↓
Notion Task Creator:
- Busca bases de datos con "tareas" o "backlog"
- Muestra: "Backlog" (id: abc123)
- Pregunta: ¿Usar esta base de datos?
↓
Usuario: "Sí"
↓
Creator:
- Verifica estructura (Title, Estado, Prioridad, Tipo)
- Lee 5 archivos de .claude/docs/tasks/
- Crea 5 páginas en Notion
↓
Resultado:
✅ 001: Crear modelo turno
✅ 002: Implementar formulario
✅ 003: Crear vista calendario
✅ 004: Agregar validaciones
✅ 005: Crear tests

🎉 5 tareas sincronizadas en Notion
Ver: https://notion.so/database/abc123
```

## Pasos del workflow

### Paso 1: Solicitud del usuario

El usuario describe qué quiere planificar:
```
Usuario: "Quiero planificar la implementación del módulo de reportes"
```

### Paso 2: Planificación de tareas

El orquestador invoca el **Task Planner**:
```
@task-planner planifica la implementación del módulo de reportes.
```
Primero revisa CLAUDE.md.

Luego:
1. Hace preguntas clarificadoras:
   - ¿Qué tipos de reportes?
   - ¿Quién los genera?
   - ¿Qué datos incluyen?
   - ¿Exportación a PDF/Excel?
   - ¿Prioridades?

2. Genera tareas individuales:
   ```
   001-crear-modelo-reporte.md
   002-disenar-interfaz-reportes.md
   003-implementar-generador-pdf.md
   004-crear-api-datos-reporte.md
   005-agregar-filtros-fecha.md
   006-implementar-exportacion-excel.md
   007-crear-tests-reportes.md
   ```

3. Informa al usuario:
   ```
   ✅ 7 tareas generadas en .claude/docs/tasks/

   Por tipo:
   - feature: 5
   - testing: 1
   - documentación: 1

   Por prioridad:
   - alta: 3
   - media: 3
   - baja: 1
   ```

### Paso 3: Revisión arquitectónica

El orquestador invoca el **Architecture Reviewer**:
```
@architecture-reviewer revisa las tareas de .claude/docs/tasks/
```

El reviewer:
1. Lee todas las tareas generadas
2. Verifica cada una:
   - 001: ✅ Correcto (migración Prisma bien especificada)
   - 002: ⚠️ Ajuste necesario (agregar consideraciones de Server Components)
   - 003: ⚠️ Ajuste necesario (especificar librería de PDF recomendada)
   - 004: ⚠️ Ajuste necesario (agregar patrón de queries con cache)
   - 005: ✅ Correcto
   - 006: ⚠️ Ajuste necesario (agregar dependencias npm necesarias)
   - 007: ✅ Correcto

3. Hace ajustes en las tareas que lo necesitan
4. Devuelve la salida de las modificaciones realizadas.

### Paso 4: Sincronización con Notion

El orquestador invoca el **Notion Task Creator**:
```
@notion-task-creator sube las tareas a Notion
```

El creator:
1. Busca base de datos de Notion:
   ```
   Encontradas:
   1. "Backlog" (id: abc123)
   2. "Tareas Personales" (id: def456)

   ¿Cuál usar?
   ```

2. Usuario selecciona: `1`

3. Verifica estructura:
   ```
   ✅ Base de datos compatible:
   - Title (title)
   - Estado (select): pendiente, en progreso, completada
   - Prioridad (select): alta, media, baja
   - Tipo (select): feature, bug, mejora, testing
   - Descripción (rich_text)
   ```

4. Pregunta: `¿Crear 7 tareas en Notion? (y/n)`

5. Usuario confirma: `y`

6. Crea las tareas:
   ```
   Creando tareas en Notion...

   ✅ 001: Crear modelo reporte
   ✅ 002: Diseñar interfaz reportes
   ✅ 003: Implementar generador PDF
   ✅ 004: Crear API datos reporte
   ✅ 005: Agregar filtros fecha
   ✅ 006: Implementar exportación Excel
   ✅ 007: Crear tests reportes

   🎉 7 tareas creadas exitosamente en Notion!

   Ver en: https://notion.so/database/abc123
   ```

## Casos de uso

```
Usuario: "Planifica sistema completo de notificaciones"

Task Planner:
- Hace 15+ preguntas sobre tipos, canales, preferencias, etc.
- Genera 20 tareas:
  * 8 de database/backend
  * 6 de frontend/UI
  * 3 de integraciones (email, push)
  * 2 de testing
  * 1 de documentación

Architecture Reviewer:
- Revisa 20 tareas
- Ajusta 12 (agrega patrones, librerías, consideraciones)
- 8 quedan sin cambios

Notion Creator:
- Crea 20 tareas nuevas
- ✓ Sistema completo planificado
```

## Principios clave

Cada tarea debe ser:
- **Específica**: Objetivo claro y único
- **Accionable**: Se puede implementar de inmediato
- **Verificable**: Tiene criterios de aceptación claros
- **Estimable**: Tamaño razonable (no épicas gigantes)

## Estructura de archivos

```
.claude/
├── agents/
│   ├── task-planner.md              # Paso 1: Planificación
│   ├── architecture-reviewer.md      # Paso 2: Revisión técnica
│   └── notion-task-creator.md        # Paso 3: Sincronización
├── docs/
│   ├── task_template.md              # Plantilla para tareas
│   ├── tasks/                        # Tareas generadas
│   │   ├── 001-nombre.md
│   │   ├── 002-nombre.md
│   │   └── ...
│   └── architecture-review-*.md      # Reportes de revisión
└── workflows/
    └── task-planning.md              # Este archivo
```
