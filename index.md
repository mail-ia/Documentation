# Índice Maestro de Documentación — MAIL-IA Platform

> **Bienvenido a la base de conocimiento y directrices de ingeniería de MAIL-IA.**  
> Este documento sirve como mapa de navegación para Agentes de Inteligencia Artificial y Desarrolladores. A continuación se resume en pocas palabras el propósito y contenido de cada archivo.

---

## 🗺️ Mapa de Archivos y Contenidos

```text
Documentation/
├── index.md                                   # (Este archivo) Índice maestro y resumen de documentación
├── README.md                                  # Guía de inicio rápido para agentes
├── assets/
│   └── logo.jpeg                              # Logotipo oficial de la plataforma MAIL-IA
├── context/
│   ├── business.md                            # Contexto estratégico, modelo de negocio y plan Fast-Track 21 días
│   └── developer.md                           # Arquitectura técnica global (NestJS + Angular), DDD y Clean Architecture
├── rules/
│   ├── angular-rules.md                       # Estándares de Angular (v18/19+), Standalone, Signals, Smart/Dumb pattern
│   ├── agent-instructions.md                  # Protocolo operativo para el agente IA, límites de complejidad y Quality Gate
│   ├── security-and-design.md                 # Seguridad XSS (Iframe Sandbox), paleta Claude y pautas de diseño UI/UX
│   └── api-mock-strategy.md                   # Estrategia de adaptadores Mock vs HTTP para desacoplamiento total del backend
└── version-beta-front/
    └── week-1/
        ├── resume.md                          # Resumen ejecutivo y hoja de ruta de la Semana 1 (Beta 21 días)
        ├── task-01-login-auth.md              # Tarea 1: Login, Registro, validaciones y AuthStateService
        ├── task-02-layout-navigation.md       # Tarea 2: Shell maestro, Sidebar con logotipo, navegación y responsive drawer
        ├── task-03-chat-ai-live-preview.md    # Tarea 3: Chat de creación IA (45%) + Live Preview en tiempo real (55%)
        ├── task-04-api-contracts-and-mocks.md # Tarea 4: DTOs, interfaces TypeScript y Mock Services listos para usar
        └── task-05-testing-and-validation.md  # Tarea 5: Plan de pruebas unitarias y Playwright E2E obligatorias (DoD)
```

---

## 📚 Resumen de Archivos por Categoría

### 1. Contexto del Proyecto (`context/`)
* **[`context/business.md`](./context/business.md):** Describe el problema de negocio, la propuesta de valor SaaS B2B, los planes de precios ($0 a $199+), márgenes brutos (>85%), análisis de competidores y el cronograma de lanzamiento Fast-Track de 21 días (3 semanas).
* **[`context/developer.md`](./context/developer.md):** Define la arquitectura global del sistema (Monolito Modular desacoplado), el stack oficial (NestJS + Neon PostgreSQL + Prisma + Redis/BullMQ + Angular), separación estricta de repositorios y modelos de dominio centrales.

### 2. Reglas de Ingeniería & Frontend (`rules/`)
* **[`rules/angular-rules.md`](./rules/angular-rules.md):** Guía técnica para desarrollo en Angular moderno (v18/19+): uso obligatorio de Standalone Components, reactividad con Signals (`signal`, `computed`, `input`, `output`), nuevo Control Flow (`@if`, `@for`, `@let`), inyección con `inject()`, y separación Smart Services vs Dumb Components.
* **[`rules/agent-instructions.md`](./rules/agent-instructions.md):** Reglas estrictas para el Agente IA: Prohibición total de `any`, complejidad ciclomática máxima de 4, límite de 200-300 líneas por archivo, y el **protocolo obligatorio de ejecución de pruebas antes de dar por terminada cualquier tarea**.
* **[`rules/security-and-design.md`](./rules/security-and-design.md):** Protocolo de seguridad frontend (tratamiento de HTML de IA como no confiable, renderizado en iframe sandbox, cero secretos en cliente) y sistema de diseño visual (paleta Claude `#0B1120`, `#131B2E`, `#5B7CFA` y pautas del logo).
* **[`rules/api-mock-strategy.md`](./rules/api-mock-strategy.md):** Arquitectura de adaptadores de API (Port & Adapter en frontend): cómo estructurar servicios y modelos para operar con datos mock durante la fase sin backend, permitiendo conectar NestJS posteriormente mediante Inyección de Dependencias sin tocar la UI.

### 3. Tareas de la Semana 1 — Versión Beta 21 Días (`version-beta-front/week-1/`)
* **[`version-beta-front/week-1/resume.md`](./version-beta-front/week-1/resume.md):** Visión general y objetivos de la Semana 1, matriz de dependencias entre tareas y criterios globales de éxito.
* **[`version-beta-front/week-1/task-01-login-auth.md`](./version-beta-front/week-1/task-01-login-auth.md):** Especificación de las vistas de Login, Registro y Recuperación de contraseña, validaciones reactivas, integración con `MockAuthApiService` y pruebas requeridas.
* **[`version-beta-front/week-1/task-02-layout-navigation.md`](./version-beta-front/week-1/task-02-layout-navigation.md):** Especificación del Layout principal, Sidebar fija oscura con el logotipo oficial (`assets/logo.jpeg`), navegación activa, perfil de usuario y drawer responsivo para móviles.
* **[`version-beta-front/week-1/task-03-chat-ai-live-preview.md`](./version-beta-front/week-1/task-03-chat-ai-live-preview.md):** Especificación de la pantalla insignia `/campaigns/new` (Split screen 45% chat conversacional con chips rápidos y 55% Live Preview seguro con pestañas Preview/HTML/Texto y selector Desktop/Mobile).
* **[`version-beta-front/week-1/task-04-api-contracts-and-mocks.md`](./version-beta-front/week-1/task-04-api-contracts-and-mocks.md):** Contratos completos en TypeScript (DTOs de Auth, Campañas, Chat, Plantillas) y fixtures de datos simulados para desarrollo inmediato.
* **[`version-beta-front/week-1/task-05-testing-and-validation.md`](./version-beta-front/week-1/task-05-testing-and-validation.md):** Plan detallado de pruebas unitarias (Jasmine/Jest) y pruebas End-to-End (Playwright) que deben ejecutarse y aprobarse como parte del Definition of Done.

### 4. Activos & Recursos (`assets/`)
* **[`assets/logo.jpeg`](./assets/logo.jpeg):** Logotipo corporativo de MAIL-IA copiado desde `recourses/logo.jpeg` para su referencia visual e integración en componentes frontend (`src/assets/images/logo.jpeg`).
