# Tarea 01 — Módulo de Autenticación & Acceso (Login / Registro)

> **Módulo:** `features/auth`  
> **Prioridad:** Alta (Bloqueante para el acceso a la plataforma)  
> **Objetivo:** Implementar la interfaz de inicio de sesión, registro de nuevos usuarios y recuperación de contraseña con validación reactiva, gestión de sesión basada en Signals y consumo del servicio de API mockeado.

---

## 1. Alcance Funcional y Pantallas

### 1.1 Pantalla de Login (`/auth/login` o `/login`)
* **Campos del Formulario:**
  * Correo Electrónico (`email`): Requerido, formato de email válido.
  * Contraseña (`password`): Requerida, mínimo 8 caracteres.
  * Recordar Sesión (`rememberMe`): Booleano opcional.
* **Acciones:**
  * Botón *"Iniciar Sesión"* (con estado `loading` y spinner).
  * Botón de acceso rápido SSO *"Continuar con Google"* (simulado).
  * Enlace a *"¿Olvidaste tu contraseña?"* (abre modal o vista de recuperación).
  * Enlace a *"¿No tienes cuenta? Regístrate gratis"*.

### 1.2 Pantalla de Registro (`/auth/register` o `/register`)
* **Campos del Formulario:**
  * Nombre Completo o Empresa (`fullName`): Requerido, min 3 caracteres.
  * Correo Electrónico Corporativo (`email`): Requerido, formato válido.
  * Contraseña (`password`): Requerido, min 8 caracteres (al menos 1 letra y 1 número).
  * Confirmación de Contraseña (`confirmPassword`): Debe coincidir con `password`.
  * Aceptación de Términos (`termsAccepted`): Checkbox obligatorio.
* **Acciones:**
  * Botón *"Crear Cuenta"* (con estado `loading`).
  * Enlace a *"¿Ya tienes cuenta? Inicia Sesión"*.

### 1.3 Modal de Recuperación de Contraseña (`/auth/recover-password`)
* Campo de correo para solicitar enlace de restablecimiento con mensaje de éxito simulado.

---

## 2. Especificación Visual y Maquetación (UI Mockup)

```text
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│                    ┌──────────────────┐                     │
│                    │ [LOGO MAIL-IA]   │ (assets/logo.jpeg)  │
│                    │     56x56px      │                     │
│                    └──────────────────┘                     │
│                          MAIL-IA                            │
│           Plataforma Inteligente de Email Marketing         │
│                                                             │
│         ┌─────────────────────────────────────────┐         │
│         │ Correo Electrónico                      │         │
│         │ [ usuario@empresa.com                 ] │         │
│         │                                         │         │
│         │ Contraseña                              │         │
│         │ [ •••••••••••••                       ] │         │
│         │                                         │         │
│         │ [x] Recordarme    ¿Olvidaste tu clave?  │         │
│         │                                         │         │
│         │ ┌─────────────────────────────────────┐ │         │
│         │ │        Iniciar Sesión               │ │         │
│         │ └─────────────────────────────────────┘ │         │
│         │                                         │         │
│         │ ───────────── o continúa con ────────── │         │
│         │                                         │         │
│         │ ┌─────────────────────────────────────┐ │         │
│         │ │ [G] Iniciar con Google              │ │         │
│         │ └─────────────────────────────────────┘ │         │
│         └─────────────────────────────────────────┘         │
│                                                             │
│             ¿No tienes cuenta? Regístrate gratis            │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

* **Fondo de Pantalla:** `--color-bg-base` (`#0B1120`).
* **Tarjeta de Formulario:** `--color-bg-surface` (`#131B2E`), borde `1px solid #1E2536`, `border-radius: 16px`, `padding: 32px`.
* **Inputs:** Fondo `#0B1120`, borde `#1E2536`, texto `#FFFFFF`, placeholder `#8A93A6`, focus border `#5B7CFA`.
* **Botón Principal:** Fondo `#5B7CFA`, texto `#FFFFFF`, hover `#4A6CF7`, `border-radius: 8px`.

---

## 3. Arquitectura del Componente y Servicios

```text
LoginComponent (Dumb UI / Reactive Form)
      │
      ▼ (Llama a método login())
AuthStateService (Smart State Service con Signals)
      │
      ▼ (Inyecta AuthApiClient)
MockAuthApiService (Simula retardo de 500ms y valida credenciales)
```

### 3.1 Credenciales de Prueba Preconfiguradas (Mock)
* **Email:** `demo@mail-ia.com`
* **Contraseña:** `Password123!`
* **Usuario Simulado Retornado:**
  ```json
  {
    "id": "usr-demo-001",
    "email": "demo@mail-ia.com",
    "fullName": "Carlos Mendoza",
    "companyName": "TechCorp Solutions",
    "role": "MARKETING_LEAD",
    "avatarUrl": "https://api.dicebear.com/7.x/avataaars/svg?seed=Carlos"
  }
  ```

---

## 4. Requisitos de Pruebas Obligatorias (DoD de la Tarea)

El agente debe implementar y ejecutar las siguientes pruebas antes de dar por terminada la tarea:

### 4.1 Pruebas Unitarias (`login.component.spec.ts` & `auth-state.service.spec.ts`)
1. **Formulario Inválido:** Comprobar que el botón esté deshabilitado o muestre errores si el email no es válido o la contraseña está vacía.
2. **Llamada al Servicio:** Verificar que al enviar credenciales válidas se invoque `AuthStateService.login()`.
3. **Manejo de Errores:** Simular error de credenciales incorrectas y verificar que se muestre el mensaje de error en la UI (`#EF4444`).
4. **Estado de Carga:** Verificar que el spinner se muestre mientras `isLoading()` sea `true`.

### 4.2 Pruebas E2E con Playwright (`e2e/auth-flow.spec.ts`)
1. Navegar a `/login`.
2. Completar email `demo@mail-ia.com` y password `Password123!`.
3. Hacer clic en "Iniciar Sesión".
4. Verificar redirección automática a `/campaigns/new` o `/dashboard`.
5. Probar intento de login con credenciales erróneas y verificar visualización del mensaje de error sin redirección.

---

## 5. Checklist de Entrega para el Agente

- [ ] Formulario reactivo de Login implementado sin uso de `any`.
- [ ] Formulario reactivo de Registro implementado con validación de coincidencia de contraseñas.
- [ ] Logotipo oficial (`assets/images/logo.jpeg`) renderizado correctamente en el encabezado del formulario.
- [ ] `AuthStateService` implementado con Signals (`currentUser`, `isAuthenticated`, `isLoading`, `error`).
- [ ] `MockAuthApiService` configurado e inyectado mediante token de interfaz.
- [ ] Redirección post-login a `/campaigns/new`.
- [ ] Pruebas unitarias aprobadas (>80% cobertura en auth).
- [ ] Pruebas E2E de Playwright ejecutadas y aprobadas.
- [ ] `tsc --noEmit` y `ng lint` sin errores.
