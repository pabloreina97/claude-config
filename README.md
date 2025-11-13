# Claude Code Workflow - Metodología de trabajo con agentes especializados

Esta es mi forma de trabajar con Claude Code, optimizada para ahorro de tokens y máxima eficiencia mediante agentes especializados.

## Prerequisitos

### Primero: instalar el MCP de Serena

Serena es un MCP (Model Context Protocol) que proporciona acceso mejorado a herramientas y documentación. **Debe instalarse antes de comenzar a trabajar.**

```bash
# Instalar Serena según tu sistema operativo
# [Agregar instrucciones específicas de instalación]
```

### Configurar herramientas adicionales

Según `~/.claude/CLAUDE.md`, utilizamos:
- **eza** en lugar de `tree -L` para visualización de directorios
- **Title casing en español**: Minúsculas salvo la primera letra ("Este es un título", no "Este Es Un Título")

## Filosofía: agentes especializados por herramienta

### ESTO NO ❌

Agentes basados en **roles genéricos**:
- Architect
- Developer
- QA
- etc.

**Problema**: Consumen mucho y el resultado no mejora significativamente.

### Esto sí ✅

Agentes basados en **herramientas específicas**:

- **shadcn-ui-architect**: Experto en shadcn/ui, con acceso a la documentación reciente y al MCP de componentes
- **supabase-expert**: Experto en Supabase, con acceso a la documentación reciente y al MCP
- **next-js-expert**: Experto en Next.js, con acceso a la documentación reciente y al MCP
- **vercel-ai-sdk-v5-expert**: Experto en Vercel AI SDK v5, con acceso a documentación actualizada

**Ventaja**: Cada agente tiene conocimiento profundo y actualizado de su herramienta específica.

## Flujo de trabajo completo

### 1. Usuario solicita funcionalidad al agente general

```
Usuario → Agente General (Claude Code principal)
```

### 2. Agente general crea contexto de sesión

El agente general crea/actualiza un archivo de contexto:

```
.claude/context/context_session_x.md
```

**Contenido del contexto:**
- Objetivo de la sesión
- Plan general de trabajo
- Estado actual
- Decisiones tomadas
- Notas importantes

**Ejemplo:**
```markdown
# Sesión 3: implementación de sistema de autenticación

## Objetivo
Implementar login y registro con Supabase Auth

## Plan general
1. Investigar implementación de Supabase Auth (supabase-expert)
2. Implementar UI de login con shadcn (shadcn-ui-architect)
3. Integrar con Next.js App Router (next-js-expert)

## Estado actual
- Fase: Investigación inicial
- Agente activo: supabase-expert
```

### 3. Agente general identifica y delega al agente especializado

#### Delegación secuencial (tareas dependientes)

```
Agente General → Agente Especializado A → Agente General → Agente Especializado B
                  (con el archivo de contexto)
```

Cuando una tarea depende de otra:
```
"Necesito implementar autenticación con Supabase.
Contexto: .claude/context/context_session_3.md"
```

#### Delegación paralela (tareas independientes) ⚡

```
                  ┌→ Agente Especializado A
Agente General ──┤
                  └→ Agente Especializado B

(ambos con el archivo de contexto)
```

**Cuando usar paralelización:**
- Las tareas son completamente independientes
- No hay dependencias entre los resultados
- Ambos pueden trabajar con el mismo contexto

**Ejemplo:**
```
"Necesito investigar dos aspectos independientes:
1. UI del dashboard con shadcn (shadcn-ui-architect)
2. API de datos con Supabase (supabase-expert)

Contexto: .claude/context/context_session_7.md

Ejecutar ambos agentes EN PARALELO"
```

**Ventaja**: Ahorro de tiempo de ejecución, ambos agentes investigan simultáneamente.

### 4. Agente especializado investiga (SIN implementar)

**IMPORTANTE**: El agente especializado **NUNCA implementa**, solo investiga y planifica.

**Proceso del agente especializado:**

#### Fase 1: Investigación
1. Lee `.claude/context/context_session_x.md` para obtener contexto completo
2. Analiza el codebase actual
3. Revisa documentación actualizada (vía MCP)
4. Identifica patrones existentes
5. Consulta APIs y mejores prácticas recientes

#### Fase 2: Planificación
1. Diseña la arquitectura de la solución
2. Identifica **archivos específicos** a crear/modificar
3. Detalla **cambios exactos** por archivo
4. Documenta decisiones arquitectónicas
5. Incluye **notas sobre conocimiento desactualizado**

#### Fase 3: Documentación
1. Crea plan detallado en `.claude/docs/plans/[nombre-descriptivo].md`
2. Estructura el plan de forma accionable
3. Incluye ejemplos de código
4. Documenta dependencias necesarias

### 5. Agente especializado retorna plan

**Formato de salida obligatorio:**

```
I've created a plan at `.claude/docs/plans/[nombre].md`,
please read that first before you proceed.

Important notes:
- [Nota crítica sobre cambios recientes en la herramienta]
- [Advertencia sobre conocimiento desactualizado]
- [Consideración arquitectónica importante]
```

**El agente NO repite el contenido completo del plan en el mensaje.**

### 6. Agente general lee el plan e implementa

```
Agente General:
  1. Lee .claude/docs/plans/[nombre].md completo
  2. Ejecuta implementación paso a paso
  3. Actualiza context_session_x.md con resultados
```

**Ventaja clave**: El agente general se ahorra todos los tokens de la investigación, pudiendo tener **mucho más ahorro de contexto** y mantener más historial de la implementación.

## Reglas estrictas para agentes especializados

Los agentes especializados **DEBEN** cumplir estas reglas:

### 1. NUNCA implementar
- ❌ No ejecutar código
- ❌ No hacer cambios en archivos
- ❌ No correr `build`, `dev`, `test`
- ✅ Solo investigar y planificar

### 2. SIEMPRE leer contexto antes
```
Antes de cualquier trabajo:
  → Leer .claude/context/context_session_x.md
```

### 3. SIEMPRE crear plan después
```
Después de investigación:
  → Crear .claude/docs/plans/[nombre-descriptivo].md
```

### 4. SIEMPRE actualizar contexto
```
Al finalizar:
  → Actualizar .claude/context/context_session_x.md
  → Agregar resumen de la investigación
  → Documentar decisiones tomadas
```

### 5. NO delegar a otros agentes
- El agente especializado hace **toda** la investigación
- No debe llamar a otros sub-agentes

### 6. Asumir conocimiento desactualizado
- Siempre incluir notas sobre cambios recientes
- Documentar diferencias con versiones anteriores
- Advertir sobre deprecaciones y nuevas APIs

## Estructura de archivos

```
.claude/
├── tasks/                          # Contextos de sesiones
│   ├── context_session_1.md       # Sesión 1: Setup inicial
│   ├── context_session_2.md       # Sesión 2: Auth
│   └── context_session_3.md       # Sesión 3: Dashboard
│
├── doc/                            # Planes de implementación
│   ├── supabase-auth-plan.md      # Plan de autenticación
│   ├── dashboard-ui-plan.md       # Plan de UI dashboard
│   └── ai-chat-integration.md     # Plan de integración AI
│
├── agents/                         # Definiciones de agentes
│   ├── shadcn-ui-architect.md     # Agente UI
│   ├── supabase-expert.md         # Agente Supabase
│   ├── next-js-expert.md          # Agente Next.js
│   └── vercel-ai-sdk-expert.md    # Agente AI SDK
│
└── CLAUDE.md                       # Instrucciones globales
```

## Anatomía de un agente especializado

Ver ejemplo completo en: `.claude/agents/shadcn-ui-architect.md`

### Estructura básica

```markdown
---
name: nombre-del-agente
description: Cuándo usar este agente [con ejemplos]
tools: [Lista de herramientas MCP y estándar]
model: sonnet
color: cyan
---

[Prompt del sistema del agente]

## Goal

Tu objetivo es proponer un plan detallado de implementación que incluya:
- Qué archivos crear/cambiar específicamente
- Qué cambios/contenido exacto
- Notas importantes (asumiendo conocimiento desactualizado)

NUNCA hagas la implementación real, solo propón el plan.

Guarda el plan en `.claude/docs/plans/xxxxx.md`

## Workflow

### Fase 1: Investigación
[Descripción detallada de qué investigar]

### Fase 2: Planificación
[Descripción detallada de qué planificar]

### Fase 3: Documentación
[Descripción detallada del formato del plan]

## Output format

Tu mensaje final DEBE incluir la ruta del archivo creado:

"I've created a plan at `.claude/docs/plans/xxxxx.md`,
please read that first before you proceed.

Important notes:
- [Nota importante 1]
- [Nota importante 2]"

NO repitas el contenido completo del plan en el mensaje.

## Rules

- NUNCA hagas la implementación actual
- NUNCA ejecutes build, dev, o comandos similares
- ANTES de trabajar: DEBE leer `.claude/context/context_session_x.md`
- DESPUÉS de terminar: DEBE crear `.claude/docs/plans/xxxxx.md`
- DESPUÉS de terminar: DEBE actualizar context_session_x.md
- NO delegues a otros sub-agentes
- Tú eres el experto, tú haces toda la investigación
```

### Secciones clave

#### 1. Goal (Objetivo)
Define claramente:
- Qué debe producir el agente (un plan, no código)
- Nivel de detalle requerido
- Dónde guardar el resultado

#### 2. Output format (Formato de salida)
Especifica exactamente:
- Formato del mensaje final
- Qué incluir (ruta del plan + notas importantes)
- Qué NO incluir (contenido completo del plan)

#### 3. Rules (Reglas)
Lista explícita de:
- ❌ Qué NUNCA hacer
- ✅ Qué SIEMPRE hacer
- 📋 Orden de operaciones

## Ejecución paralela de agentes ⚡

### Cuándo paralelizar

✅ **Paralelizar cuando:**
- Los agentes investigan **herramientas diferentes** (UI + Backend)
- No hay dependencias entre los resultados
- Las tareas son completamente independientes
- Quieres optimizar tiempo de ejecución

❌ **NO paralelizar cuando:**
- Un agente necesita el resultado del otro
- Las tareas están relacionadas secuencialmente
- Hay decisiones arquitectónicas compartidas

### Cómo solicitar ejecución paralela

**Sintaxis explícita:**
```
"Necesito investigar [tarea A] y [tarea B] EN PARALELO.

Agentes:
- [agente-a]: [descripción tarea A]
- [agente-b]: [descripción tarea B]

Contexto: .claude/context/context_session_x.md"
```

**Ejemplo real:**
```
"Necesito investigar la UI y el backend EN PARALELO.

Agentes:
- shadcn-ui-architect: Diseñar componentes del dashboard
- supabase-expert: Diseñar esquema de base de datos y queries

Contexto: .claude/context/context_session_5.md"
```

### Ventajas de la paralelización

1. **Ahorro de tiempo**: 2 agentes trabajando = ~50% del tiempo
2. **Independencia**: Cada agente se enfoca en su especialidad
3. **Planes complementarios**: Resultados que se integran después
4. **Eficiencia de tokens**: El agente principal consume menos tokens totales

### Flujo de trabajo paralelo

```
1. Usuario solicita funcionalidad compleja
   ↓
2. Agente general identifica aspectos independientes
   ↓
3. Agente general crea context_session_x.md
   ↓
4. Agente general lanza múltiples agentes EN PARALELO
   ├─→ Agente A investiga (aspecto 1)
   └─→ Agente B investiga (aspecto 2)
   ↓
5. Ambos agentes retornan sus planes
   ├─→ .claude/docs/plans/plan-a.md
   └─→ .claude/docs/plans/plan-b.md
   ↓
6. Agente general lee AMBOS planes
   ↓
7. Agente general implementa de forma coordinada
```

### Ejemplo completo paralelo

#### Solicitud
```
"Necesito un sistema de chat con IA usando Vercel AI SDK
y una UI moderna con shadcn"
```

#### Paso 1: Contexto
`.claude/context/context_session_8.md`:
```markdown
# Sesión 8: sistema de chat con IA

## Objetivo
Implementar chat con streaming de IA y UI moderna

## Plan general
1. 🔀 PARALELO: Investigar UI (shadcn-ui-architect) + AI SDK (vercel-ai-sdk-expert)
2. Implementar UI de chat
3. Implementar integración con AI
4. Conectar ambos sistemas

## Estado actual
- Fase: Investigación paralela
- Agentes activos: shadcn-ui-architect + vercel-ai-sdk-expert
```

#### Paso 2: Delegación paralela
```
Agente general lanza EN PARALELO:

1. shadcn-ui-architect:
   "Diseña UI de chat con mensajes, input, y streaming.
   Contexto: .claude/context/context_session_8.md"

2. vercel-ai-sdk-expert:
   "Diseña integración con Vercel AI SDK para streaming.
   Contexto: .claude/context/context_session_8.md"
```

#### Paso 3: Ambos agentes trabajan simultáneamente
```
shadcn-ui-architect:              vercel-ai-sdk-expert:
- Lee contexto                    - Lee contexto
- Analiza componentes             - Analiza AI SDK v5
- Diseña UI de chat               - Diseña streaming
- Crea plan-ui.md                 - Crea plan-ai.md
```

#### Paso 4: Resultados
```
Plan A: .claude/docs/plans/chat-ui-plan.md
- Componentes: Card, ScrollArea, Input, Button
- Layout responsivo
- Estados de loading

Plan B: .claude/docs/plans/chat-ai-integration-plan.md
- useChat() hook de Vercel AI SDK
- Streaming con Server Actions
- Manejo de errores
```

#### Paso 5: Implementación coordinada
```
Agente general:
1. Lee ambos planes
2. Identifica puntos de integración
3. Implementa UI
4. Implementa AI
5. Conecta ambos
```

**Resultado**: Tiempo reducido ~50%, planes complementarios, implementación eficiente.

## Ventajas de este workflow

### 1. Ahorro masivo de tokens en el contexto principal
```
Sin agentes especializados:
  Investigación (50k tokens) + Implementación (30k tokens) = 80k tokens

Con agentes especializados:
  Agente: Investigación (50k tokens) → Plan (5k tokens)
  Principal: Lee plan (5k tokens) + Implementación (30k tokens) = 35k tokens
```

### 2. Conocimiento especializado y actualizado
- Cada agente tiene acceso a MCPs específicos
- Documentación siempre actualizada
- Mejores prácticas recientes

### 3. Separación de responsabilidades
- Investigación vs Ejecución claramente separadas
- Los agentes pueden profundizar sin presión
- Planes revisables antes de implementar

### 4. Trazabilidad completa
- Todo queda documentado
- Fácil revisar decisiones pasadas
- Historial de contexto preservado

### 5. Continuidad entre sesiones
- Múltiples sesiones sobre el mismo proyecto
- Contexto preservado entre invocaciones
- Trabajo incremental posible

### 6. Ejecución paralela ⚡
- Múltiples agentes trabajando simultáneamente
- Ahorro de ~50% de tiempo en tareas independientes
- Planes complementarios que se integran después
- Máxima eficiencia en proyectos complejos

## Ejemplo completo de flujo

### Solicitud inicial
```
Usuario: "Necesito implementar un dashboard con gráficos
usando shadcn/ui y datos de Supabase"
```

### Paso 1: Crear contexto
Agente general crea `.claude/context/context_session_7.md`:

```markdown
# Sesión 7: Dashboard con datos de Supabase

## Objetivo
Implementar dashboard con visualización de datos en tiempo real

## Plan general
1. 🔀 PARALELO: Investigar UI (shadcn-ui-architect) + Supabase (supabase-expert)
2. Implementar UI base
3. Implementar integración de datos
4. Añadir subscripciones real-time

## Estado actual
- Fase: Investigación paralela
- Agentes activos: shadcn-ui-architect + supabase-expert
```

> **💡 Nota sobre paralelización**: En este ejemplo, ambos agentes pueden trabajar EN PARALELO porque las tareas son independientes (UI no depende de backend, backend no depende de UI). Esto reduce el tiempo de investigación a la mitad.

### Paso 2: Consultar ambos agentes EN PARALELO

Agente general lanza simultáneamente:

**shadcn-ui-architect:**
```
"Necesito un plan para implementar un dashboard con gráficos.
Los datos vendrán de Supabase.
Contexto: .claude/context/context_session_7.md"
```

**supabase-expert:**
```
"Necesito un plan para integrar datos real-time de Supabase.
El UI mostrará tablas y gráficos.
Contexto: .claude/context/context_session_7.md"
```

### Paso 3: Ambos agentes trabajan simultáneamente ⚡

**shadcn-ui-architect** (en paralelo):
1. ✅ Lee context_session_7.md
2. 🔍 Analiza estructura del proyecto
3. 📚 Consulta documentación shadcn/ui reciente (MCP)
4. 🎨 Identifica componentes: Card, Table, Chart
5. 📝 Crea `.claude/docs/plans/dashboard-ui-plan.md`

**supabase-expert** (en paralelo):
1. ✅ Lee context_session_7.md
2. 🔍 Analiza schema de Supabase actual
3. 📚 Consulta documentación Supabase reciente (MCP)
4. 🔄 Investiga real-time subscriptions
5. 📝 Crea `.claude/docs/plans/supabase-integration-plan.md`

### Paso 4: Ambos planes retornados

**Plan UI:**
```
I've created a plan at `.claude/docs/plans/dashboard-ui-plan.md`,
please read that first before you proceed.

Important notes:
- shadcn/ui charts now use recharts 2.x with new API
- Use the new Data Table pattern with TanStack Table v8
- Chart components require separate installation: npx shadcn@latest add chart
```

**Plan Supabase:**
```
I've created a plan at `.claude/docs/plans/supabase-integration-plan.md`,
please read that first before you proceed.

Important notes:
- Supabase JS client v2.x has breaking changes in real-time API
- Use the new channel-based subscriptions instead of old .on('*')
- Row Level Security must be configured before real-time works
```

### Paso 5: Implementación coordinada

Agente general:
1. 📖 Lee **ambos planes** (dashboard-ui-plan.md + supabase-integration-plan.md)
2. 🔗 Identifica puntos de integración entre UI y datos
3. 💻 Implementa UI según el plan
4. 💻 Implementa integración de datos según el plan
5. 🔄 Conecta ambos sistemas
6. ✅ Actualiza context_session_7.md con resultados

**Resultado**:
- ⚡ Tiempo de investigación reducido ~50% gracias a paralelización
- 💾 Ahorro masivo de tokens en el agente principal
- 📋 Dos planes complementarios que se integran perfectamente
- ✨ Implementación completa y eficiente

## Best practices

### Para agentes especializados

✅ **Sé exhaustivo en la investigación**
- Lee TODO el código relevante
- Consulta documentación oficial
- Revisa issues y changelogs recientes

✅ **Sé específico en el plan**
- Nombres exactos de archivos
- Rutas completas
- Código de ejemplo cuando sea necesario

✅ **Documenta decisiones**
- Explica el "por qué" de cada elección
- Compara alternativas consideradas
- Justifica la opción seleccionada

✅ **Asume desactualización**
- Incluye notas sobre cambios recientes
- Advierte sobre breaking changes
- Documenta migraciones necesarias

✅ **Actualiza el contexto**
- Resume tu investigación
- Documenta decisiones tomadas
- Deja notas para el agente principal

### Para el agente principal

✅ **Siempre mantén contexto**
- Crea context_session_x.md al inicio
- Actualiza después de cada tarea
- Preserva historial de decisiones

✅ **Lee los planes completos**
- No asumas, lee todo
- Entiende las notas importantes
- Respeta las decisiones arquitectónicas

✅ **Comunica problemas**
- Si hay inconsistencias, consulta de nuevo
- No improvises soluciones arquitectónicas
- Vuelve al agente especializado si necesario

✅ **Implementa fielmente**
- Sigue el plan al pie de la letra
- No cambies decisiones arquitectónicas
- Respeta las advertencias del plan

### Para usuarios

✅ **Sé específico en las solicitudes**
- Más contexto = mejor plan
- Menciona restricciones conocidas
- Comparte objetivos de negocio

✅ **Revisa los planes**
- Lee los archivos en `.claude/docs/plans/`
- Verifica que el plan cumpla tus necesidades
- Pide cambios ANTES de la implementación

✅ **Mantén sesiones lógicas**
- Agrupa trabajo relacionado
- Usa el mismo session_id
- Preserva contexto entre solicitudes

✅ **Consulta el historial**
- Los archivos context_session_x.md son tu historial
- Revisa decisiones pasadas
- Entiende el estado actual

## Troubleshooting

### ❌ "El agente está implementando en lugar de planificar"

**Causa**: Falta claridad en las Rules del agente

**Solución**: Verifica que el agente tenga:
```markdown
## Rules

- NUNCA hagas la implementación actual
- NUNCA ejecutes build, dev, o comandos similares
```

### ❌ "No encuentro el plan que creó el agente"

**Causa**: El agente no retornó la ruta correctamente

**Solución**:
1. Revisa los archivos recientes en `.claude/docs/plans/`
2. Verifica que el agente tenga Output format definido
3. Busca archivos modificados recientemente: `ls -lt .claude/docs/plans/`

### ❌ "El contexto se perdió entre sesiones"

**Causa**: No se está usando el mismo session_id

**Solución**:
1. Usa el mismo session_id para trabajo relacionado
2. Actualiza context_session_x.md después de cada tarea
3. Lee el contexto al inicio de cada sesión
4. Verifica que los agentes lean el contexto antes de trabajar

### ❌ "El plan no incluye suficiente detalle"

**Causa**: El Goal del agente no es lo suficientemente específico

**Solución**: Actualiza el Goal del agente para requerir:
- Nombres exactos de archivos
- Contenido específico de cambios
- Ejemplos de código
- Notas sobre conocimiento desactualizado
