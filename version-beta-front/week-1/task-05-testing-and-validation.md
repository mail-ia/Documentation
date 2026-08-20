# Tarea 05 — Plan de Pruebas, Validación y Criterios de Aceptación (Testing Suite)

> **Módulo:** `testing` (Unit & E2E Suites)  
> **Prioridad:** Obligatoria / Calidad  
> **Objetivo:** Definir las especificaciones de pruebas unitarias y pruebas de integración End-to-End (Playwright) que el agente o desarrollador debe implementar, ejecutar y validar antes de dar por finalizada la Semana 1.

---

## 1. Estrategia de Pruebas de la Semana 1

```text
┌────────────────────────────────────────────────────────┐
│             Nivel 3: Pruebas E2E (Playwright)          │
│       • Flujo Completo de Login                        │
│       • Navegación por Sidebar & Layout                │
│       • Creación de Correo con IA & Live Preview       │
├────────────────────────────────────────────────────────┤
│          Nivel 2: Pruebas de Componentes Dumb          │
│       • Renderizado de inputs, outputs y eventos UI    │
│       • Validación de estilos y clases activas         │
├────────────────────────────────────────────────────────┤
│          Nivel 1: Pruebas Unitarias de Servicios       │
│       • AuthStateService & MockAuthApiService          │
│       • AIChatStateService & MockCampaignApiService    │
│       • Reactividad con Signals (>80% Cobertura)       │
└────────────────────────────────────────────────────────┘
```

---

## 2. Especificación de Pruebas Unitarias

### 2.1 Pruebas de Autenticación (`auth-state.service.spec.ts`)
```typescript
describe('AuthStateService', () => {
  let service: AuthStateService;
  let mockApi: MockAuthApiService;

  beforeEach(() => {
    TestBed.configureTestingModule({
      providers: [
        AuthStateService,
        { provide: AUTH_API_CLIENT, useClass: MockAuthApiService }
      ]
    });
    service = TestBed.inject(AuthStateService);
    mockApi = TestBed.inject(AUTH_API_CLIENT) as MockAuthApiService;
  });

  it('debe iniciar con estado no autenticado', () => {
    expect(service.isAuthenticated()).toBeFalse();
    expect(service.currentUser()).toBeNull();
  });

  it('debe autenticar exitosamente con credenciales válidas', async () => {
    await service.login({ email: 'demo@mail-ia.com', password: 'Password123!' });
    expect(service.isAuthenticated()).toBeTrue();
    expect(service.currentUser()?.email).toBe('demo@mail-ia.com');
    expect(service.error()).toBeNull();
  });

  it('debe manejar error al ingresar credenciales incorrectas', async () => {
    try {
      await service.login({ email: 'demo@mail-ia.com', password: 'wrong' });
    } catch {
      expect(service.isAuthenticated()).toBeFalse();
      expect(service.error()).toBeTruthy();
    }
  });

  it('debe limpiar la sesión al invocar logout()', async () => {
    await service.login({ email: 'demo@mail-ia.com', password: 'Password123!' });
    service.logout();
    expect(service.isAuthenticated()).toBeFalse();
    expect(service.currentUser()).toBeNull();
  });
});
```

### 2.2 Pruebas del Editor de Chat & Live Preview (`ai-chat-state.service.spec.ts`)
```typescript
describe('AIChatStateService', () => {
  let service: AIChatStateService;

  beforeEach(() => {
    TestBed.configureTestingModule({
      providers: [
        AIChatStateService,
        { provide: CAMPAIGN_API_CLIENT, useClass: MockCampaignApiService }
      ]
    });
    service = TestBed.inject(AIChatStateService);
  });

  it('debe enviar un prompt y agregar mensajes de usuario e IA', async () => {
    expect(service.messages().length).toBe(0);
    const promise = service.submitPrompt('Crear correo con descuento');
    
    expect(service.isGenerating()).toBeTrue();
    await promise;

    expect(service.isGenerating()).toBeFalse();
    expect(service.messages().length).toBe(2);
    expect(service.messages()[0].sender).toBe('USER');
    expect(service.messages()[1].sender).toBe('AI');
    expect(service.currentDraft()).not.toBeNull();
    expect(service.currentDraft()?.templateHtml).toContain('MAIL-IA Campaign');
  });

  it('debe cambiar la pestaña activa correctamente', () => {
    expect(service.activeTab()).toBe('PREVIEW');
    service.setTab('HTML');
    expect(service.activeTab()).toBe('HTML');
  });

  it('debe cambiar el dispositivo de preview correctamente', () => {
    expect(service.previewDevice()).toBe('DESKTOP');
    service.setDevice('MOBILE');
    expect(service.previewDevice()).toBe('MOBILE');
  });
});
```

---

## 3. Especificación de Pruebas End-to-End (Playwright)

### 3.1 Suite de Autenticación (`e2e/auth-flow.spec.ts`)
```typescript
import { test, expect } from '@playwright/test';

test.describe('Flujo de Autenticación', () => {
  test('debe mostrar el logo oficial y permitir login exitoso', async ({ page }) => {
    await page.goto('/login');
    
    // Validar presencia del logo oficial
    const logo = page.locator('img[alt="MAIL-IA Logo"]');
    await expect(logo).toBeVisible();

    // Llenar formulario
    await page.fill('input[type="email"]', 'demo@mail-ia.com');
    await page.fill('input[type="password"]', 'Password123!');
    await page.click('button:has-text("Iniciar Sesión")');

    // Validar redirección a /campaigns/new
    await expect(page).toHaveURL(/.*campaigns\/new/);
    await expect(page.locator('text=Carlos Mendoza')).toBeVisible();
  });

  test('debe mostrar error con credenciales incorrectas', async ({ page }) => {
    await page.goto('/login');
    await page.fill('input[type="email"]', 'demo@mail-ia.com');
    await page.fill('input[type="password"]', 'ClaveInvalida999');
    await page.click('button:has-text("Iniciar Sesión")');

    await expect(page.locator('.error-message, [role="alert"]')).toBeVisible();
    await expect(page).toHaveURL(/.*login/);
  });
});
```

### 3.2 Suite de Navegación & Shell (`e2e/navigation-flow.spec.ts`)
```typescript
import { test, expect } from '@playwright/test';

test.describe('Navegación y Shell Principal', () => {
  test.beforeEach(async ({ page }) => {
    // Login inicial
    await page.goto('/login');
    await page.fill('input[type="email"]', 'demo@mail-ia.com');
    await page.fill('input[type="password"]', 'Password123!');
    await page.click('button:has-text("Iniciar Sesión")');
    await page.waitForURL(/.*campaigns\/new/);
  });

  test('debe mostrar la sidebar con logo y ruta activa', async ({ page }) => {
    const sidebarLogo = page.locator('aside img');
    await expect(sidebarLogo).toBeVisible();

    const activeLink = page.locator('aside a.active');
    await expect(activeLink).toContainText('Crear Campaña');
  });

  test('debe abrir y cerrar drawer en resolución móvil', async ({ page }) => {
    await page.setViewportSize({ width: 375, height: 667 });
    
    // Sidebar debe estar oculta inicialmente
    const sidebar = page.locator('aside');
    await expect(sidebar).toHaveClass(/closed|hidden|-translate-x/);

    // Clic en botón hamburguesa
    await page.click('button[aria-label="Abrir Menú"]');
    await expect(sidebar).toBeVisible();

    // Clic en backdrop para cerrar
    await page.click('.backdrop');
    await expect(sidebar).not.toBeVisible();
  });
});
```

### 3.3 Suite de Chat IA & Live Preview (`e2e/chat-preview-flow.spec.ts`)
```typescript
import { test, expect } from '@playwright/test';

test.describe('Chat IA & Live Preview', () => {
  test.beforeEach(async ({ page }) => {
    await page.goto('/login');
    await page.fill('input[type="email"]', 'demo@mail-ia.com');
    await page.fill('input[type="password"]', 'Password123!');
    await page.click('button:has-text("Iniciar Sesión")');
    await page.waitForURL(/.*campaigns\/new/);
  });

  test('debe enviar prompt y actualizar iframe de preview', async ({ page }) => {
    const promptInput = page.locator('textarea[placeholder*="Describe el correo"]');
    await promptInput.fill('Crear promoción de calzado deportivo');
    await page.click('button[aria-label="Enviar prompt"]');

    // Validar mensaje del usuario en el chat
    await expect(page.locator('.message-user')).toContainText('Crear promoción de calzado deportivo');

    // Validar respuesta de la IA
    await expect(page.locator('.message-ai')).toBeVisible({ timeout: 5000 });

    // Validar que el iframe cargue contenido
    const iframe = page.frameLocator('iframe.preview-iframe');
    await expect(iframe.locator('body')).not.toBeEmpty();
  });

  test('debe alternar pestañas Preview, HTML y selector Mobile', async ({ page }) => {
    // Generar diseño inicial
    await page.click('button.chip-button:first-child');
    await page.waitForTimeout(1000);

    // Cambiar a pestaña HTML
    await page.click('button:has-text("HTML")');
    await expect(page.locator('pre code, .code-editor')).toBeVisible();

    // Cambiar a dispositivo Mobile
    await page.click('button[aria-label="Vista Móvil"]');
    const container = page.locator('.preview-container');
    await expect(container).toHaveClass(/mobile-view|w-\[375px\]/);
  });
});
```

---

## 4. Comandos de Validación para el Quality Gate

El agente debe ejecutar y verificar que todos los comandos finalicen con éxito:

```bash
# 1. Type check
npx tsc --noEmit

# 2. Linter
npm run lint

# 3. Pruebas Unitarias
npm run test -- --watch=false --browsers=ChromeHeadless

# 4. Pruebas End-to-End
npx playwright test

# 5. Build de Producción
npm run build
```
