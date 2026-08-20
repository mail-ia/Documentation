# Guía y Reglas de Arquitectura Frontend — Angular

> **Versión de Referencia:** Angular 18+ / 19+ (Modern Angular)  
> **Enfoque:** Standalone Components, Signals Reactivos, Clean Code, Smart/Dumb Pattern y Cero Acoplamiento.  
> **Objetivo:** Definir las directrices no negociables para la construcción del frontend de MAIL-IA.

---

## 1. Versiones y Características Modernas de Angular

La aplicación debe desarrollarse bajo el estándar moderno de Angular, adoptando las siguientes capacidades:

| Característica | Estándar Obligatorio | Justificación |
| :--- | :--- | :--- |
| **Arquitectura de Componentes** | **Standalone Components** (`standalone: true`) | Eliminación de `NgModule` innecesarios, carga diferida simplificada y menor boilerplate. |
| **Gestión de Estado Reactivo** | **Signals** (`signal()`, `computed()`, `effect()`, `input()`, `output()`, `model()`) | Reactividad granular sin sobrecarga de Zone.js, mejor rendimiento y sintaxis declarativa. |
| **Sintaxis de Control Flow** | **Built-in Control Flow** (`@if`, `@else`, `@for`, `@switch`, `@let`) | Reemplazo total de directivas estructurales legadas (`*ngIf`, `*ngFor`). |
| **Inyección de Dependencias** | **Función `inject()`** | Inyección declarativa y tipada, eliminando constructores sobrecargados. |
| **Carga Diferida (Lazy Loading)** | **Router con `loadComponent` / `loadChildren`** | Bundles optimizados y carga bajo demanda por feature. |
| **Estrategia de Detección de Cambios** | **`ChangeDetectionStrategy.OnPush`** | Obligatorio en todos los componentes para maximizar rendimiento. |

---

## 2. Patrón de Componentes: Smart Services vs Dumb Components

### 2.1 Dumb Components (Componentes de Presentación)
* **Responsabilidad Única:** Renderizar la interfaz de usuario, capturar interacción y emitir eventos.
* **Reglas:**
  * **Prohibido:** No deben contener lógica de negocio, cálculos complejos ni inyectar `HttpClient`.
  * **Comunicación:** Reciben datos exclusivamente mediante `input()` / `input.required()` y notifican acciones mediante `output()`.
  * **Detección de Cambios:** Siempre `changeDetection: ChangeDetectionStrategy.OnPush`.

```typescript
// Ejemplo de Dumb Component
import { Component, ChangeDetectionStrategy, input, output } from '@angular/core';

@Component({
  selector: 'app-prompt-chip',
  standalone: true,
  changeDetection: ChangeDetectionStrategy.OnPush,
  template: `
    <button 
      type="button"
      class="chip-button"
      [disabled]="disabled()"
      (click)="selected.emit(label())">
      {{ label() }}
    </button>
  `,
  styles: [`
    .chip-button {
      background-color: #131B2E;
      border: 1px solid #1E2536;
      color: #8A93A6;
      padding: 6px 12px;
      border-radius: 9999px;
      cursor: pointer;
      transition: all 0.2s ease;
    }
    .chip-button:hover:not(:disabled) {
      border-color: #5B7CFA;
      color: #FFFFFF;
    }
  `]
})
export class PromptChipComponent {
  readonly label = input.required<string>();
  readonly disabled = input<boolean>(false);
  readonly selected = output<string>();
}
```

### 2.2 Smart Services (Servicios de Aplicación & Estado)
* **Responsabilidad:** Coordinar el flujo de datos, gestionar el estado con Signals, orquestar llamadas a la API y transformar DTOs a modelos de vista.
* **Reglas:**
  * Deben ser `providedIn: 'root'` o provistos a nivel de Feature Route.
  * Exponen el estado como Signals de solo lectura (`readonly state = computed(...)` o `asReadonly()`).
  * Los métodos públicos ejecutan mutaciones controladas del estado.

```typescript
// Ejemplo de Smart Service
import { Injectable, signal, computed, inject } from '@angular/core';
import { CampaignApiService } from '@core/api/campaign-api.service';
import { ChatMessage, CampaignDraft } from '@core/models/campaign.models';

@Injectable({ providedIn: 'root' })
export class AIChatStateService {
  private readonly campaignApi = inject(CampaignApiService);

  // Estado interno privado
  private readonly _messages = signal<ChatMessage[]>([]);
  private readonly _currentDraft = signal<CampaignDraft | null>(null);
  private readonly _isLoading = signal<boolean>(false);

  // Estado público de solo lectura
  readonly messages = this._messages.asReadonly();
  readonly currentDraft = this._currentDraft.asReadonly();
  readonly isLoading = this._isLoading.asReadonly();
  readonly hasDraft = computed(() => this._currentDraft() !== null);

  async sendMessage(promptText: string): Promise<void> {
    const userMessage: ChatMessage = {
      id: crypto.randomUUID(),
      sender: 'USER',
      content: promptText,
      timestamp: new Date()
    };
    
    this._messages.update(msgs => [...msgs, userMessage]);
    this._isLoading.set(true);

    try {
      const response = await this.campaignApi.sendAiPrompt(promptText);
      const assistantMessage: ChatMessage = {
        id: response.messageId,
        sender: 'AI',
        content: response.replyText,
        timestamp: new Date()
      };
      
      this._messages.update(msgs => [...msgs, assistantMessage]);
      this._currentDraft.set(response.draft);
    } finally {
      this._isLoading.set(false);
    }
  }
}
```

---

## 3. Estructura de Directorios del Proyecto

El código del frontend debe organizarse siguiendo esta estructura limpia y escalable:

```text
src/app/
├── core/                         # Singleton services, interceptors, guards, API contracts
│   ├── auth/                     # Lógica y guards de autenticación
│   │   ├── auth.guard.ts
│   │   ├── auth.interceptor.ts
│   │   └── auth-state.service.ts
│   ├── api/                      # Interfaces y adaptadores de API
│   │   ├── api-config.ts
│   │   ├── auth-api.service.ts
│   │   └── campaign-api.service.ts
│   ├── mocks/                    # Implementaciones mock y fixtures estáticos
│   │   ├── mock-auth-api.service.ts
│   │   ├── mock-campaign-api.service.ts
│   │   └── mock-fixtures.ts
│   └── models/                   # Interfaces, tipos y DTOs globales
│       ├── auth.models.ts
│       ├── campaign.models.ts
│       └── user.models.ts
│
├── shared/                       # Componentes Dumb, pipes, directivas y UI Kit
│   ├── ui/                       # Componentes visuales genéricos
│   │   ├── button/
│   │   ├── input/
│   │   ├── modal/
│   │   ├── chip/
│   │   └── spinner/
│   ├── pipes/                    # Pipes puros (ej. safe-html, date-format)
│   └── directives/               # Directivas de interacción
│
├── features/                     # Módulos de funcionalidad (Smart containers)
│   ├── auth/                     # Pantallas de Login y Registro
│   │   ├── login/
│   │   ├── register/
│   │   └── auth.routes.ts
│   ├── layout/                   # Shell de la aplicación
│   │   ├── shell/
│   │   ├── sidebar/
│   │   └── header/
│   └── campaigns/                # Creación conversacional con IA
│       ├── chat-editor/          # Split panel (Chat 45% + Preview 55%)
│       ├── components/           # Subcomponentes específicos de campaña
│       │   ├── message-list/
│       │   ├── prompt-input/
│       │   └── live-preview/
│       └── campaigns.routes.ts
│
├── assets/                       # Recursos estáticos (logos, iconos, fuentes)
│   └── images/
│       └── logo.jpeg             # Logo oficial de la plataforma
│
└── environments/                 # Configuración de entornos y flags de mock
    ├── environment.ts            # Entorno de desarrollo / mock
    └── environment.prod.ts       # Entorno de producción
```

---

## 4. Estándares de Código y Buenas Prácticas

### 4.1 Tipado Estricto (Prohibición Total de `any`)
* **Regla:** Ninguna variable, parámetro de función, observable o DTO puede tiparse como `any`.
* **Alternativas:** Usar interfaces específicas, tipos unión (`'DRAFT' | 'SENT'`) o `unknown` con type guards si el contenido es dinámico.

### 4.2 Límite de Tamaño de Archivo
* Los componentes y servicios deben mantenerse entre **150 y 300 líneas**.
* Si un componente supera este límite, debe subdividirse en Dumb Components más pequeños o delegar lógica a un servicio.

### 4.3 Complejidad Ciclomática Máxima: 4
* Ninguna función o método debe superar una complejidad ciclomática de 4.
* Evitar anidaciones profundas de `if-else` o `switch` masivos. Descomponer en funciones puras auxiliares.

### 4.4 Nomenclatura Estándar
* **Componentes:** `PascalCaseComponent` (ej. `ChatEditorComponent`, `SidebarNavComponent`).
* **Archivos:** `kebab-case.extension.ts` (ej. `chat-editor.component.ts`, `campaign-api.service.ts`).
* **Signals:** Nombres descriptivos en camelCase (`isLoading`, `messages`, `currentDraft`).
* **Outputs:** Nombres de eventos en tiempo presente o pasado (`submitted`, `canceled`, `itemSelected`).

---

## 5. Control Flow Moderno (Sintaxis Obligatoria)

❌ **Prohibido (Sintaxis Legada):**
```html
<div *ngIf="isLoading; else contentTpl">Cargando...</div>
<ng-template #contentTpl>
  <div *ngFor="let item of items; trackBy: trackById">{{ item.name }}</div>
</ng-template>
```

✅ **Obligatorio (Sintaxis Moderna):**
```html
@if (isLoading()) {
  <app-spinner label="Generando diseño..." />
} @else {
  @for (message of messages(); track message.id) {
    <app-chat-message-bubble [message]="message" />
  } @empty {
    <div class="empty-state">No hay mensajes aún. ¡Comienza a chatear con la IA!</div>
  }
}
```
