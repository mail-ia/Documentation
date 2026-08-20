# Guía de Seguridad y Sistema de Diseño (UI/UX)

> **Plataforma:** AI Email Platform (MAIL-IA)  
> **Propósito:** Directrices de seguridad en cliente y especificación del sistema de diseño visual inspirado en la estética minimalista de Claude (Anthropic).

---

## 1. Directrices de Seguridad Frontend

### 1.1 Prevención de XSS y Renderizado de HTML Generado por IA
El HTML producido por modelos de Inteligencia Artificial debe ser tratado obligatoriamente como **contenido no confiable**.

* **Prohibición de `bypassSecurityTrustHtml`:**
  * Queda estrictamente prohibido el uso directo de `DomSanitizer.bypassSecurityTrustHtml()` sobre plantillas o fragmentos de código sin sanitización previa.
  * ❌ Prohibido: `<div [innerHTML]="sanitizer.bypassSecurityTrustHtml(aiHtml)"></div>`
* **Estrategia de Renderizado Seguro (Live Preview):**
  * La previsualización de la plantilla de correo debe realizarse dentro de un **`<iframe>` aislado con atributo `sandbox`**:
  ```html
  <iframe
    class="preview-iframe"
    sandbox="allow-same-origin"
    [srcdoc]="sanitizedEmailHtml()"
    title="Email Template Live Preview">
  </iframe>
  ```
  * Esto previene la ejecución arbitraria de scripts maliciosos y aísla los estilos CSS del correo respecto a los estilos de la aplicación.

### 1.2 Gestión de Tokens y Autenticación
* **Entorno Mock:** Almacenar el token de sesión simulado en un Signal en memoria dentro de `AuthStateService` con persistencia opcional en `sessionStorage`.
* **Entorno Producción:** La autenticación se realiza mediante cookies `HttpOnly` y `Secure`, mitigando el riesgo de robo de tokens vía JavaScript.
* **Cierre de Sesión Seguro:** Limpieza inmediata del estado reactivo y redirección inmediata a `/login`.

### 1.3 Cero Secretos en el Cliente (*Zero Secrets*)
* Ninguna clave de API (Groq, OpenRouter, Resend, Postmark, AWS, Neon DB) debe existir en `environment.ts` ni en el código fuente de Angular.
* Todas las llamadas a servicios de IA o despacho de emails se canalizan exclusivamente a través del backend NestJS (`/api/...`).

---

## 2. Sistema de Diseño Visual (UI / UX)

El lenguaje visual de MAIL-IA sigue los principios de **claridad, alto espacio en blanco (whitespace), tipografía pulida y minimalismo oscuro**, inspirado en aplicaciones de IA conversacional de primer nivel (estilo Claude).

### 2.1 Paleta de Colores Oficial

```text
┌───────────────────────────────┬──────────────┬──────────────────────────────────────────┐
│ Rol / Superficie              │ Código Hex   │ Uso en Interfaz                          │
├───────────────────────────────┼──────────────┼──────────────────────────────────────────┤
│ Background Base (60%)         │ #0B1120      │ Fondo principal de pantalla y layout     │
│ Superficie de Tarjetas (30%)  │ #131B2E      │ Sidebar, cards, burbujas de chat, inputs │
│ Bordes & Separadores (10%)    │ #1E2536      │ Bordes de paneles, líneas divisorias     │
│ Acento Primario / Acción      │ #5B7CFA      │ Botones principales, estados activos     │
│ Acento Hover                  │ #4A6CF7      │ Hover en botones y enlaces interactivos  │
│ Texto Principal               │ #FFFFFF      │ Títulos, texto clave, inputs activos     │
│ Texto Secundario / Muted      │ #8A93A6      │ Subtítulos, labels, placeholders, fechas │
│ Éxito                         │ #10B981      │ Estados completados, badges verdes       │
│ Advertencia                   │ #F59E0B      │ Alertas de límite de cuota o advertencia │
│ Error                         │ #EF4444      │ Mensajes de error en formularios         │
└───────────────────────────────┴──────────────┴──────────────────────────────────────────┘
```

### 2.2 Variables CSS Globales
Estas variables deben definirse en `src/styles.css` o `styles/variables.css`:

```css
:root {
  --color-bg-base: #0B1120;
  --color-bg-surface: #131B2E;
  --color-border: #1E2536;
  --color-accent: #5B7CFA;
  --color-accent-hover: #4A6CF7;
  --color-text-primary: #FFFFFF;
  --color-text-secondary: #8A93A6;
  --color-success: #10B981;
  --color-warning: #F59E0B;
  --color-error: #EF4444;

  --font-family-base: 'Inter', system-ui, -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
  --radius-sm: 6px;
  --radius-md: 10px;
  --radius-lg: 16px;
  --radius-full: 9999px;

  --transition-fast: 150ms ease;
  --transition-normal: 250ms ease;
}
```

---

## 3. Guía de Uso del Logotipo Oficial

El activo oficial del logotipo de MAIL-IA se encuentra ubicado en:
* **Ruta de documentación:** `Documentation/assets/logo.jpeg`
* **Ruta del repositorio:** `recourses/logo.jpeg`
* **Ruta de destino en Angular:** `src/assets/images/logo.jpeg`

### 3.1 Pautas de Implementación del Logo en la UI

1. **Pantalla de Login / Registro (`/login`):**
   * Ubicación: Centrado superior en la tarjeta de autenticación.
   * Dimensiones: Ancho de `56px` a `64px`, con `border-radius: 12px` y sombra sutil (`box-shadow: 0 4px 20px rgba(91, 124, 250, 0.15)`).
   * Acompañado del título: **MAIL-IA** (Font-weight: 700, 24px) y el subtítulo *"Plataforma Inteligente de Email Marketing"*.

2. **Sidebar de Navegación Principal:**
   * Ubicación: Esquina superior izquierda de la barra lateral.
   * Dimensiones: Alto de `36px` a `40px`, manteniendo la proporción de aspecto.
   * Comportamiento: Clic en el logo redirige siempre al inicio (`/campaigns/new` o `/dashboard`).

3. **Favicon y PWA:**
   * Generar versiones cuadradas (`32x32`, `192x192`, `512x512`) optimizadas a partir del logo original.

---

## 4. Tipografía y Espaciado

* **Familia Tipográfica:** `Inter` o fuentes de sistema sans-serif limpias.
* **Escala Tipográfica:**
  * **H1 (Títulos de Página):** `24px` / `1.3` / Semibold (`600`)
  * **H2 (Secciones & Paneles):** `18px` / `1.4` / Medium (`500`)
  * **Body / Mensajes de Chat:** `14px` / `1.5` / Regular (`400`)
  * **Labels / Meta / Badges:** `12px` / `1.4` / Medium (`500`)
* **Espaciado y Padding:**
  * Márgenes consistentes basados en múltiplos de 4px (`8px`, `16px`, `24px`, `32px`).
  * Alto uso de espacios vacíos para evitar saturación cognitiva.
