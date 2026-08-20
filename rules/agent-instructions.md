# Instrucciones y Protocolo Operativo para el Agente IA de Desarrollo

> **Destinatario:** Agentes de Inteligencia Artificial (Antigravity, Claude Code, Cursor, Windsurf, Copilot, etc.) y Desarrolladores Frontend.  
> **Repositorio Oficial:** `https://github.com/mail-ia/mail-web.git`  
> **Estrategia de Trabajo:** Fork + Gitflow desde `main` + Conventional Commits.  
> **Objetivo:** Establecer el protocolo riguroso de implementación, gestión de ramas, validación, pruebas obligatorias y criterios de aceptación (Definition of Done) antes de cerrar cualquier tarea.

---

## 1. Repositorio, Fork y Convención de Ramas (Gitflow)

1. **Origen del Repositorio:**
   * El repositorio base es [`https://github.com/mail-ia/mail-web.git`](https://github.com/mail-ia/mail-web.git).
   * El agente o desarrollador debe trabajar sobre un **Fork** de este repositorio.
   * `upstream` debe apuntar al repositorio oficial `https://github.com/mail-ia/mail-web.git`.

2. **Creación de Ramas (Gitflow):**
   * Toda rama de funcionalidad se crea **a partir de `main`** actualizado:
     ```bash
     git checkout main
     git pull upstream main
     git checkout -b feature/week-1-task-<num>-<nombre-corto>
     ```
   * Ejemplos de ramas de Semana 1:
     * `feature/week-1-task-01-login-auth`
     * `feature/week-1-task-02-layout-navigation`
     * `feature/week-1-task-03-chat-ai-live-preview`

3. **Commits Estándar (Conventional Commits):**
   * Todo commit debe seguir el formato: `<type>(<scope>): <description>` (ej. `feat(auth): add reactive login form`, `test(chat): add e2e playwright test`).
   * Tipos permitidos: `feat`, `fix`, `test`, `refactor`, `style`, `docs`, `chore`.
   * Para más detalles, consultar la [Guía de Gitflow y Commits](./git-workflow.md).

---

## 2. Mandamientos No Negociables del Código

Todo código generado debe cumplir sin excepciones con las siguientes restricciones:

1. **Tipado Estricto (0% `any`):**
   * Queda terminantemente prohibido el uso de `any`.
   * Todo modelo, DTO, payload, evento, estado de Signal y respuesta de servicio debe tener una interfaz o tipo explícito.
   * Usar `unknown` con type narrowing solo si la estructura externa es variable.

2. **Complejidad Ciclomática Máxima: 4:**
   * Ningún método o función debe exceder una complejidad ciclomática de 4.
   * Descomponer lógica con múltiples condicionales en funciones puras y unitarias (ej. `validateInput()`, `transformPayload()`, `handleSuccess()`).

3. **Límite de Tamaño de Archivo: 200 – 300 Líneas:**
   * Archivos de más de 300 líneas indican violación del principio de responsabilidad única (SRP).
   * Dividir componentes grandes en Smart Container + Dumb Presentational Components.

4. **Nombres Descriptivos sin Abreviaciones:**
   * ❌ Prohibido: `const usr = ...`, `const btn = ...`, `const res = ...`
   * ✅ Obligatorio: `const authenticatedUser = ...`, `const submitButton = ...`, `const campaignResponse = ...`

5. **Aislamiento de la IA en Frontend:**
   * El cliente frontend **jamás** debe invocar APIs externas de IA (Groq, OpenAI, Anthropic, OpenRouter) directamente.
   * Toda interacción conversacional pasa a través del servicio de abstracción (`CampaignApiService` / `AIChatService`), el cual se conecta al backend o a su adaptador mock correspondiente.

---

## 3. Flujo de Trabajo Operativo del Agente

Cuando se asigne una tarea de desarrollo, el agente debe seguir obligatoriamente este orden secuencial:

```text
┌────────────────────────────────────────────────────────┐
│ 1. Fork & Rama Gitflow a partir de main                │
└──────────────────────────┬─────────────────────────────┘
                           │
                           ▼
┌────────────────────────────────────────────────────────┐
│ 2. Análisis de Requisitos & Contratos de Datos (DTOs) │
└──────────────────────────┬─────────────────────────────┘
                           │
                           ▼
┌────────────────────────────────────────────────────────┐
│ 3. Implementación de Servicios & Mocks API             │
└──────────────────────────┬─────────────────────────────┘
                           │
                           ▼
┌────────────────────────────────────────────────────────┐
│ 4. Creación de Componentes (Dumb UI + Smart Containers)│
└──────────────────────────┬─────────────────────────────┘
                           │
                           ▼
┌────────────────────────────────────────────────────────┐
│ 5. Creación y Ejecución de Pruebas Unitarias (TDD)     │
└──────────────────────────┬─────────────────────────────┘
                           │
                           ▼
┌────────────────────────────────────────────────────────┐
│ 6. Ejecución de Pruebas E2E (Playwright)               │
└──────────────────────────┬─────────────────────────────┘
                           │
                           ▼
┌────────────────────────────────────────────────────────┐
│ 7. Verificación del Quality Gate (Build + Lint + Tests)│
└──────────────────────────┬─────────────────────────────┘
                           │
                           ▼
┌────────────────────────────────────────────────────────┐
│ 8. Commit Convencional & Entrega con Evidencia         │
└────────────────────────────────────────────────────────┘
```

---

## 4. Protocolo de Pruebas Obligatorio (Definition of Done - DoD)

> [!CRITICAL]
> **Ninguna tarea se considerará finalizada ni entregada hasta que todas las pruebas hayan sido ejecutadas y aprobadas exitosamente.**
> El agente debe presentar la evidencia de la ejecución de pruebas en su reporte final.

### 4.1 Pruebas Unitarias Requeridas
* **Servicios de Estado & API Mocks:** Cobertura mínima del **80%**. Deben verificarse escenarios de éxito, fallo, simulación de retardo y actualización reactiva de Signals.
* **Componentes Dumb:** Verificar que rendericen correctamente los datos pasados por `input()` y que emitan los eventos correctos en `output()`.
* **Validadores de Formularios:** Cobertura del 100% en reglas de validación (email válido, longitud de contraseñas, campos requeridos).

### 4.2 Pruebas End-to-End (Playwright)
Para cada funcionalidad de la Semana 1, deben existir y ejecutarse pruebas E2E automatizadas:
* **Flujo de Login:** Autenticación con credenciales válidas, rechazo con credenciales erróneas, navegación a `/campaigns/new`.
* **Flujo de Navegación:** Renderizado de la Sidebar, presencia del logotipo, colapso responsivo y navegación entre rutas activas.
* **Flujo de Chat IA:** Envío de prompt, visualización de mensajes en el chat, selección de chips rápidos y actualización del Live Preview.

---

## 5. Checklist de Validación del Quality Gate

Antes de emitir el reporte final o abrir PR, el agente debe ejecutar localmente los siguientes comandos y validar su salida:

```bash
# 1. Verificación de Tipos TypeScript (Cero errores)
npx tsc --noEmit

# 2. Análisis Estático y Linter
npm run lint

# 3. Ejecución de Pruebas Unitarias en modo CI
npm run test -- --watch=false --browsers=ChromeHeadless

# 4. Ejecución de Pruebas E2E con Playwright
npx playwright test

# 5. Compilación del Bundle de Producción
npm run build
```

---

## 6. Formato del Reporte de Entrega del Agente

Al finalizar una tarea, el agente debe generar un reporte estructurado con las siguientes secciones:

1. **Resumen de la Tarea:** Descripción clara de lo implementado.
2. **Rama y Commits:** Nombre de la rama Gitflow y lista de Conventional Commits realizados.
3. **Archivos Creados / Modificados:** Lista de rutas absolutas con links markdown.
4. **Evidencia de Pruebas Unitarias:** Resumen de suites ejecutadas y porcentaje de cobertura.
5. **Evidencia de Pruebas E2E:** Estado de los tests de Playwright (Passed / Failed).
6. **Verificación de Calidad:** Confirmación de que `tsc`, `lint` y `build` finalizaron con código de salida `0`.
7. **Capturas o Descripción de Comportamiento Visual:** Validación de responsividad y paleta de colores.
