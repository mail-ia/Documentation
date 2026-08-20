# Tarea 02 — Layout Principal y Menú de Navegación (Sidebar & Shell)

> **Módulo:** `features/layout` (Shell, Sidebar, Header)  
> **Prioridad:** Alta (Estructura base para todas las pantallas del producto)  
> **Objetivo:** Construir el layout principal de la aplicación, incorporando la barra lateral fija (Sidebar) con el logotipo corporativo, enlaces de navegación, indicador de ruta activa, perfil de usuario y comportamiento responsivo (Drawer móvil).

---

## 1. Alcance Funcional y Componentes

### 1.1 Estructura del Shell (`AppShellComponent`)
El shell actúa como contenedor maestro de la aplicación autenticada:

```text
┌─────────────────────────────────────────────────────────────────────────────┐
│ [Logo] MAIL-IA   │ Topbar: [Título de Sección]         [+ Nueva Campaña] [Avatar]│
├──────────────────┼──────────────────────────────────────────────────────────┤
│                  │                                                          │
│  [✦] Crear con IA│                                                          │
│  [⊞] Dashboard   │                                                          │
│  [❏] Plantillas  │               <router-outlet>                            │
│  [👥] Audiencias │            (Área Principal de Contenido)                 │
│  [✈] Envíos      │                                                          │
│  [📊] Métricas   │                                                          │
│  [⚙] Ajustes     │                                                          │
│                  │                                                          │
├──────────────────┤                                                          │
│ [Avatar] Carlos  │                                                          │
│ TechCorp [Logout]│                                                          │
└──────────────────┴──────────────────────────────────────────────────────────┘
```

### 1.2 Sidebar de Navegación (`SidebarComponent`)
* **Cabecera del Sidebar:**
  * Logotipo oficial cargado desde `assets/images/logo.jpeg` (`Documentation/assets/logo.jpeg`).
  * Altura de `36px` a `40px` con bordes redondeados (`border-radius: 8px`).
  * Texto de marca: **MAIL-IA** en color `#FFFFFF` con badge sutil *"BETA"*.
* **Menú de Enlaces:**
  1. **Crear Campaña (Chat IA):** `/campaigns/new` (Pantalla principal de la Semana 1).
  2. **Dashboard:** `/dashboard` (Placeholder informativo de Semana 2/3).
  3. **Plantillas & Recursos:** `/templates` (Placeholder).
  4. **Contactos & Audiencias:** `/contacts` (Placeholder).
  5. **Programación & Envíos:** `/campaigns/send` (Placeholder).
  6. **Telemetría & Métricas:** `/analytics` (Placeholder).
  7. **Configuración de Marca:** `/settings` (Placeholder).
* **Indicador de Ruta Activa:**
  * La opción activa debe resaltar con fondo `rgba(91, 124, 250, 0.12)`, texto `#5B7CFA` y una barra lateral izquierda de `3px` en color índigo (`#5B7CFA`).
* **Pie del Sidebar (Widget de Usuario):**
  * Avatar circular del usuario autenticado (`currentUser.avatarUrl`).
  * Nombre del usuario y nombre de la empresa (`fullName` / `companyName`).
  * Botón de Cerrar Sesión (`logout()`), que invoca `AuthStateService.logout()` y redirige a `/login`.

### 1.3 Barra Superior (Topbar / Header)
* Título dinámico de la página actual basado en la ruta activa.
* Botón de acción rápida: **"+ Nueva Campaña"** con fondo índigo (`#5B7CFA`).
* Botón hamburguesa (visible solo en resolución móvil < 768px) para desplegar el drawer.

### 1.4 Soporte Responsivo (Mobile Drawer)
* En resoluciones móviles (< 768px), la sidebar se oculta automáticamente.
* Al presionar el botón hamburguesa, la sidebar se desliza como un drawer sobre un fondo oscuro semitransparente (*backdrop* con `backdrop-filter: blur(4px)`).
* Al hacer clic fuera o seleccionar una ruta, el drawer se cierra automáticamente.

---

## 2. Especificación Visual y Estilos

* **Fondo de la Sidebar:** `--color-bg-surface` (`#131B2E`).
* **Borde Derecho:** `1px solid #1E2536`.
* **Ancho de Sidebar:** `260px` fija en escritorio.
* **Tipografía de Enlaces:** `14px`, peso `500`, color inactivo `#8A93A6`, color hover `#FFFFFF`, color activo `#5B7CFA`.
* **Iconos:** Iconos SVG limpios (estilo Lucide / Heroicons) integrados como componentes dumb.

---

## 3. Requisitos de Pruebas Obligatorias (DoD de la Tarea)

El agente debe implementar y ejecutar las siguientes pruebas antes de dar por terminada la tarea:

### 3.1 Pruebas Unitarias (`sidebar.component.spec.ts` & `shell.component.spec.ts`)
1. **Renderizado de Enlaces:** Verificar que los 7 ítems de navegación se rendericen con sus respectivas rutas.
2. **Presencia del Logotipo:** Comprobar que el elemento `<img>` del logo exista y apunte a `assets/images/logo.jpeg`.
3. **Muestra de Datos de Usuario:** Verificar que el nombre del usuario y su empresa provengan del Signal `currentUser` de `AuthStateService`.
4. **Emisión de Logout:** Verificar que al hacer clic en el botón de logout se ejecute el método de cierre de sesión.

### 3.2 Pruebas E2E con Playwright (`e2e/navigation-flow.spec.ts`)
1. Iniciar sesión y validar que el shell y la sidebar se muestren.
2. Verificar que la ruta `/campaigns/new` tenga la clase de estilo activa.
3. Hacer clic en otros enlaces del menú y verificar la navegación y actualización del título.
4. En viewport móvil (`375x667`), comprobar que la sidebar esté oculta, abrirla con el botón hamburguesa y validar que se cierre al hacer tap en el backdrop.
5. Hacer clic en "Cerrar Sesión" y verificar la redirección a `/login`.

---

## 4. Checklist de Entrega para el Agente

- [ ] Shell maestro (`AppShellComponent`) estructurado con Sidebar, Header y `<router-outlet>`.
- [ ] Sidebar implementada con el logotipo corporativo (`assets/images/logo.jpeg`).
- [ ] Enlace a `/campaigns/new` destacado como acción principal.
- [ ] Indicador de ruta activa (`routerLinkActive`) con estilos índigo.
- [ ] Tarjeta de usuario con avatar y botón de logout funcional.
- [ ] Drawer responsivo para dispositivos móviles con animación suave.
- [ ] Pruebas unitarias de componentes de navegación aprobadas.
- [ ] Pruebas E2E en escritorio y móvil aprobadas en Playwright.
- [ ] Cero uso de `any` y cumplimiento de límites de líneas (<300 líneas por archivo).
