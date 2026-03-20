# Prompt para el Orchestrator: Habit Tracker (Angular + Lit)

> **Propósito:** Este prompt está diseñado para probar el framework de desarrollo en `.github/`. Antes de empezar, lee el archivo `.github/agents/orchestrator.agent.md` y sigue su Execution Flow completo. Usa todos los templates de `.github/templates/`, carga el domain profile de `.github/domains/` si existe uno que coincida con el stack, y genera todos los artefactos que el framework requiere (PRD, Tech Spec, Implementation Plan, VERIFICATION_LOG.md, PROJECT_STATUS.md, Review).

---

## El proyecto

Quiero construir **Habit Tracker** — una aplicación web SPA para hacer seguimiento de hábitos diarios con rachas (streaks) y progreso visual.

### Stack tecnológico

- **Angular 19+** (standalone components, sin NgModules) para la estructura de la app, routing, formularios y estado
- **Lit 3+** (Web Components) para los componentes visuales reutilizables (tarjetas, indicadores de progreso, badges)
- **Vite** como bundler (con el plugin de Angular correspondiente)
- **TypeScript** en modo strict
- **Vitest** para tests

Intenté usar NgModules pero añaden demasiado boilerplate para una app de este tamaño — quiero solo standalone components.

### Funcionalidades

#### 1. Lista de hábitos
- Mostrar todos los hábitos del usuario en una cuadrícula responsive
- Cada hábito se muestra en una **tarjeta visual** (componente Lit `<habit-card>`) que incluye:
  - Nombre del hábito
  - Icono (emoji seleccionable)
  - Anillo de progreso circular mostrando el porcentaje de completitud semanal
  - Badge de racha (streak) si lleva 3+ días consecutivos
  - Botón de check para marcar el hábito como completado hoy
- Cuando un hábito está marcado como completado hoy, la tarjeta debe **deshabilitar sus controles** y mostrar un estado visual de "done"

#### 2. Formulario de creación/edición
- Modal Angular para crear o editar un hábito
- Campos: nombre (requerido, max 30 chars), icono (emoji picker simplificado con 10 opciones predefinidas), frecuencia objetivo (diario, 5/semana, 3/semana)
- Validación reactiva con Angular Reactive Forms

#### 3. Vista de progreso
- Componente Lit `<progress-ring>` que muestra un anillo SVG con porcentaje animado
- Componente Lit `<streak-badge>` que muestra la racha actual con efectos visuales (fuego 🔥 para 7+, estrella ⭐ para 30+)
- Componente Angular para la página de estadísticas: mejor racha, tasa de completitud del mes, hábito más consistente

#### 4. Persistencia
- LocalStorage para almacenar hábitos y registros de completitud
- Servicio Angular con signals para gestionar el estado reactivo
- Al cargar la app, el servicio lee de localStorage y popula los signals
- Cada cambio de estado se persiste automáticamente

#### 5. Temas
- Soporte para tema claro y oscuro
- Toggle en el header (componente Lit `<theme-toggle>`)
- El tema se persiste en localStorage
- CSS custom properties para los colores del tema

### Requisitos no funcionales

- La app debe funcionar offline (no hay backend, todo es local)
- El anillo de progreso debe animar suavemente la transición de porcentaje (CSS transitions)
- La cuadrícula de hábitos debe ser responsive: 3 columnas en desktop, 2 en tablet, 1 en mobile
- Accesible: los botones deben tener aria-labels, los colores deben tener contraste suficiente

### Restricciones

- **No usar NgModules** — solo standalone components con `imports` directos
- **No usar librerías de componentes externas** (Material, PrimeNG, etc.) — los componentes Lit son el sistema de diseño
- **No usar backend ni Firebase** — solo localStorage
- El proyecto debe poder hacerse `npm run build` y `npm run dev` sin errores desde cero
- **No usar Tailwind CSS**

### Tests esperados

- Test unitario del servicio de hábitos (crear, completar, calcular racha)
- Test unitario de al menos un componente Lit (habit-card: renderizado, eventos)
- Test unitario de al menos un componente Angular (formulario: validación)
- Mínimo 70% de cobertura en el servicio de hábitos

### Estructura sugerida (no obligatoria)

```
habit-tracker/
├── src/
│   ├── app/                     # Angular
│   │   ├── core/services/       # HabitService, StorageService
│   │   ├── features/
│   │   │   ├── habits/          # Lista, formulario
│   │   │   └── stats/           # Página de estadísticas
│   │   └── shared/              # Tipos, utilidades
│   ├── components/              # Lit Web Components
│   │   ├── habit-card.ts
│   │   ├── progress-ring.ts
│   │   ├── streak-badge.ts
│   │   └── theme-toggle.ts
│   ├── styles/                  # CSS global, theme variables
│   ├── main.ts                  # Entry point
│   └── index.html
├── package.json
├── tsconfig.json
├── vite.config.ts
└── vitest.config.ts
```

### Lo que espero al final

1. Que pueda hacer `npm install && npm run build && npm run dev` y ver la app funcionando
2. Que los tests pasen con `npm test`
3. Que los componentes Lit funcionen correctamente dentro de las páginas Angular
4. Que el estado sea reactivo — marcar un hábito como completado actualice inmediatamente la UI
5. Todos los artefactos del framework: PRD, Tech Spec, Implementation Plan, VERIFICATION_LOG.md con evidencia real, PROJECT_STATUS.md y Review

---

**Nota para el LLM:** Este proyecto usa Angular + Lit. Comprueba si existe un domain profile en `.github/domains/` para este stack antes de empezar. Si existe, cárgalo y aplica sus reglas de integración, pitfalls y verification commands. Si no existe, créalo como parte de la fase de Tech Spec.
