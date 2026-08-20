# Guía de Flujo de Trabajo Git — Fork, Gitflow y Commits Estándar

> **Repositorio Oficial (Upstream):** `https://github.com/mail-ia/mail-web.git`  
> **Estrategia de Trabajo:** Forking Workflow + Gitflow + Conventional Commits  
> **Objetivo:** Estandarizar el ciclo de desarrollo, la nomenclatura de ramas y el historial de cambios para todo el frontend de MAIL-IA.

---

## 1. Estrategia de Repositorio & Fork

El repositorio central oficial es [`https://github.com/mail-ia/mail-web.git`](https://github.com/mail-ia/mail-web.git). Ningún desarrollador o agente debe hacer push directo sobre las ramas protegidas del repositorio oficial.

### 1.1 Configuración Inicial del Entorno

```bash
# 1. Realizar Fork del repositorio desde GitHub hacia tu cuenta/organización

# 2. Clonar tu repositorio forkeado localmente
git clone https://github.com/<tu-usuario-o-agente>/mail-web.git
cd mail-web

# 3. Configurar el repositorio oficial como 'upstream'
git remote add upstream https://github.com/mail-ia/mail-web.git

# 4. Verificar configuración de remotos
git remote -v
# origin    https://github.com/<tu-usuario-o-agente>/mail-web.git (fetch)
# origin    https://github.com/<tu-usuario-o-agente>/mail-web.git (push)
# upstream  https://github.com/mail-ia/mail-web.git (fetch)
# upstream  https://github.com/mail-ia/mail-web.git (push)
```

### 1.2 Sincronización Continua con Upstream

Antes de iniciar cualquier nueva rama o tarea, sincronizar siempre con `upstream/main`:

```bash
git checkout main
git fetch upstream
git merge upstream/main
git push origin main
```

---

## 2. Estrategia de Ramas (Gitflow)

Todas las ramas de trabajo deben partir de `main` (o `develop` si está activa) y seguir la nomenclatura estandarizada:

```text
main (Producción / Versión Estable)
  │
  ├── feature/week-1-task-01-login-auth
  ├── feature/week-1-task-02-layout-navigation
  ├── feature/week-1-task-03-chat-ai-live-preview
  ├── feature/week-1-task-04-api-contracts-mocks
  └── feature/week-1-task-05-testing-suite
```

### 2.1 Convención de Nombres de Ramas

| Tipo de Rama | Formato de Nomenclatura | Propósito | Ejemplo |
| :--- | :--- | :--- | :--- |
| **Feature (Semana 1)** | `feature/week-1-task-<num>-<nombre-corto>` | Nuevas funcionalidades del sprint | `feature/week-1-task-01-login-auth` |
| **Feature General** | `feature/<modulo>-<descripcion>` | Funcionalidades específicas | `feature/auth-reactive-form` |
| **Bugfix** | `fix/<modulo>-<descripcion>` | Corrección de defectos detectados | `fix/chat-iframe-sandbox-scroll` |
| **Refactor** | `refactor/<modulo>-<descripcion>` | Mejoras de código sin cambio funcional | `refactor/sidebar-signals-cleanup` |
| **Hotfix** | `hotfix/<descripcion-urgente>` | Parche crítico directo a producción | `hotfix/auth-token-expiration` |

### 2.2 Creación de una Rama de Trabajo

```bash
# Asegurarse de estar en main actualizado
git checkout main
git pull upstream main

# Crear y cambiar a la rama de la tarea específica
git checkout -b feature/week-1-task-01-login-auth
```

---

## 3. Estándar de Commits (Conventional Commits v1.0.0)

Todo mensaje de confirmación (*commit*) debe seguir estrictamente la especificación de **Conventional Commits**:

```text
<tipo>(<alcance opcional>): <descripción concisa en imperativo>

[cuerpo opcional con detalles técnicos y justificación]

[pie opcional con referencias a tareas / tickets]
```

### 3.1 Tipos de Commit Permitidos

| Tipo | Cuándo Usarlo | Ejemplo |
| :--- | :--- | :--- |
| **`feat`** | Nueva funcionalidad para el usuario | `feat(auth): implement reactive login form with signal state` |
| **`fix`** | Corrección de un error o bug | `fix(preview): resolve responsive width calculation on mobile toggle` |
| **`docs`** | Cambios exclusivos en documentación | `docs(readme): add setup and testing instructions for week 1` |
| **`style`** | Ajustes visuales, CSS o formato (sin afectar lógica) | `style(layout): update sidebar background to claude-dark #131B2E` |
| **`refactor`** | Reestructuración de código sin alterar comportamiento | `refactor(chat): extract prompt chips into dumb presentational component` |
| **`test`** | Adición o corrección de pruebas unitarias o E2E | `test(e2e): add playwright tests for user login and auth redirection` |
| **`chore`** | Configuración de build, linters, scripts o dependencias | `chore(deps): configure playwright and mock api providers` |

### 3.2 Reglas para los Mensajes de Commit
1. Usar modo imperativo y minúsculas en la primera línea (ej. `add`, `implement`, `fix`, `refactor`).
2. No colocar punto final en la primera línea.
3. Longitud máxima del asunto: **72 caracteres**.
4. Explicar el *por qué* y no solo el *qué* en el cuerpo del commit si el cambio es complejo.

---

## 4. Ejemplos de Commits por Tarea (Semana 1)

### Tarea 01: Login & Autenticación
```bash
git commit -m "feat(auth): add reactive login and register forms"
git commit -m "feat(auth): implement AuthStateService with signals and mock provider"
git commit -m "test(auth): add unit tests for login validation and auth state"
```

### Tarea 02: Layout & Navegación
```bash
git commit -m "feat(layout): create app shell with claude-style dark sidebar"
git commit -m "feat(layout): include official brand logo in sidebar and header"
git commit -m "feat(layout): implement responsive mobile drawer with backdrop"
git commit -m "test(layout): add unit and e2e navigation tests"
```

### Tarea 03: Chat IA & Live Preview
```bash
git commit -m "feat(chat): implement split screen 45/55 layout for campaign editor"
git commit -m "feat(chat): add quick prompt adjustment chips and message bubbles"
git commit -m "feat(preview): integrate secure iframe sandbox with responsive tabs"
git commit -m "test(chat): add playwright suite for prompt submission and preview update"
```

---

## 5. Ciclo de Validación y Apertura de Pull Request (PR)

Antes de realizar `git push` y abrir un Pull Request hacia `upstream/main`:

```bash
# 1. Ejecutar Quality Gate local obligatorio
npx tsc --noEmit
npm run lint
npm run test -- --watch=false --browsers=ChromeHeadless
npx playwright test
npm run build

# 2. Subir la rama a tu fork (origin)
git push -u origin feature/week-1-task-01-login-auth

# 3. Crear Pull Request desde GitHub:
# Base repository: mail-ia/mail-web (branch: main)
# Head repository: <tu-usuario>/mail-web (branch: feature/week-1-task-01-login-auth)
```
