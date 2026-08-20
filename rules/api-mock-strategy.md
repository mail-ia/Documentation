# Estrategia de Mocking y Arquitectura de Conexión a API

> **Contexto:** En esta fase inicial, el backend NestJS se encuentra en desarrollo paralelo.  
> **Objetivo:** Construir una arquitectura frontend 100% preparada para el consumo de API REST real, utilizando adaptadores mock intercambiables mediante Inyección de Dependencias.  
> **Regla de Oro:** Los componentes visuales y los servicios de aplicación **jamás** deben contener datos estáticos en código duro (*hardcoded*). Deben consumir siempre las interfaces de API.

---

## 1. Patrón de Adaptadores de API (Port & Adapter en Frontend)

Para desacoplar el frontend del estado de desarrollo del backend, se implementa el patrón de abstracción de servicios de API:

```text
┌────────────────────────────────────────────────────────┐
│                   Angular Component                    │
│            (ej. ChatEditorComponent / LoginComponent)   │
└───────────────────────────┬────────────────────────────┘
                            │ (Inyecta servicio de aplicación)
                            ▼
┌────────────────────────────────────────────────────────┐
│               Application State Service                │
│            (ej. AuthStateService / AIChatStateService) │
└───────────────────────────┬────────────────────────────┘
                            │ (Inyecta contrato de API)
                            ▼
┌────────────────────────────────────────────────────────┐
│                API Contract Interface                  │
│       (ej. AuthApiClient / CampaignApiClient)          │
└───────────────┬────────────────────────┬───────────────┘
                │                        │
  [useMock: true]                        [useMock: false]
                ▼                        ▼
┌───────────────────────────────┐ ┌──────────────────────────────┐
│       Mock API Adapter        │ │       Http API Adapter       │
│  (Simula retardos, fixtures,  │ │  (Realiza llamadas HttpClient│
│   y respuestas dinámicas)     │ │   hacia el backend NestJS)   │
└───────────────────────────────┘ └──────────────────────────────┘
```

---

## 2. Definición de Contratos e Interfaces de API

Todos los métodos de comunicación con el backend deben tiparse con interfaces de petición (Request DTO) y respuesta (Response DTO):

### 2.1 Contrato de Autenticación (`auth-api.interface.ts`)
```typescript
import { Observable } from 'rxjs';
import { 
  LoginRequestDto, 
  LoginResponseDto, 
  RegisterRequestDto, 
  RegisterResponseDto,
  UserProfileDto 
} from '../models/auth.models';

export interface AuthApiClient {
  login(credentials: LoginRequestDto): Observable<LoginResponseDto>;
  register(data: RegisterRequestDto): Observable<RegisterResponseDto>;
  logout(): Observable<void>;
  getCurrentUser(): Observable<UserProfileDto>;
  recoverPassword(email: string): Observable<{ success: boolean; message: string }>;
}
```

### 2.2 Contrato de Campañas e IA (`campaign-api.interface.ts`)
```typescript
import { Observable } from 'rxjs';
import { 
  AiPromptRequestDto, 
  AiPromptResponseDto, 
  CampaignDto, 
  CreateCampaignDto 
} from '../models/campaign.models';

export interface CampaignApiClient {
  sendAiPrompt(request: AiPromptRequestDto): Observable<AiPromptResponseDto>;
  getCampaignById(id: string): Observable<CampaignDto>;
  saveDraft(campaign: Partial<CreateCampaignDto>): Observable<CampaignDto>;
  listRecentCampaigns(): Observable<CampaignDto[]>;
}
```

---

## 3. Implementación del Adaptador Mock (`MockCampaignApiService`)

El adaptador mock debe emular el comportamiento real de red, incluyendo latencia realista y generación dinámica de plantillas simuladas:

```typescript
import { Injectable } from '@angular/core';
import { Observable, of, delay, throwError } from 'rxjs';
import { CampaignApiClient } from './campaign-api.interface';
import { AiPromptRequestDto, AiPromptResponseDto, CampaignDto } from '../models/campaign.models';
import { MOCK_CAMPAIGN_TEMPLATES } from '../mocks/mock-fixtures';

@Injectable({ providedIn: 'root' })
export class MockCampaignApiService implements CampaignApiClient {
  
  sendAiPrompt(request: AiPromptRequestDto): Observable<AiPromptResponseDto> {
    // Simula retardo de inferencia de IA (800ms)
    const simulatedResponse: AiPromptResponseDto = {
      messageId: crypto.randomUUID(),
      replyText: this.generateAiReply(request.prompt),
      draft: {
        id: request.campaignId ?? crypto.randomUUID(),
        name: 'Campaña Asistida por IA',
        subject: 'Descubre nuestra nueva colección exclusiva',
        templateHtml: this.generateSimulatedHtml(request.prompt),
        plainText: 'Descubre nuestra nueva colección exclusiva con descuentos de hasta el 20%.',
        updatedAt: new Date().toISOString()
      }
    };

    return of(simulatedResponse).pipe(delay(800));
  }

  getCampaignById(id: string): Observable<CampaignDto> {
    const fixture = MOCK_CAMPAIGN_TEMPLATES.find(c => c.id === id) ?? MOCK_CAMPAIGN_TEMPLATES[0];
    return of(fixture).pipe(delay(300));
  }

  saveDraft(campaign: Partial<CampaignDto>): Observable<CampaignDto> {
    const saved: CampaignDto = {
      id: campaign.id ?? crypto.randomUUID(),
      userId: 'user-123',
      name: campaign.name ?? 'Borrador sin título',
      subject: campaign.subject ?? '',
      senderName: 'MAIL-IA Marketing',
      replyToEmail: 'noreply@empresa.com',
      templateHtml: campaign.templateHtml ?? '<p>Contenido base</p>',
      status: 'DRAFT',
      audienceIds: [],
      categoryIds: [],
      metrics: {
        totalRecipients: 0,
        deliveredCount: 0,
        openedCount: 0,
        clickedCount: 0,
        bouncedCount: 0,
        unsubscribedCount: 0
      },
      createdAt: new Date(),
      updatedAt: new Date()
    };
    return of(saved).pipe(delay(400));
  }

  listRecentCampaigns(): Observable<CampaignDto[]> {
    return of(MOCK_CAMPAIGN_TEMPLATES).pipe(delay(300));
  }

  private generateAiReply(prompt: string): string {
    if (prompt.toLowerCase().includes('formal')) {
      return 'He ajustado el tono a un estilo corporativo y formal, destacando el valor comercial.';
    }
    if (prompt.toLowerCase().includes('oferta') || prompt.toLowerCase().includes('descuento')) {
      return 'He añadido un banner de descuento del 20% con un botón de llamada a la acción prominente.';
    }
    return 'He generado la estructura inicial de la plantilla basada en tus indicaciones. Puedes previsualizarla en el panel derecho.';
  }

  private generateSimulatedHtml(prompt: string): string {
    return `
      <!DOCTYPE html>
      <html>
      <head>
        <meta charset="utf-8">
        <style>
          body { font-family: 'Helvetica Neue', Arial, sans-serif; background-color: #f4f5f7; margin: 0; padding: 20px; }
          .container { max-width: 600px; margin: 0 auto; background: #ffffff; border-radius: 8px; overflow: hidden; box-shadow: 0 2px 8px rgba(0,0,0,0.05); }
          .header { background: #0B1120; color: #ffffff; padding: 30px; text-align: center; }
          .content { padding: 30px; color: #333333; line-height: 1.6; }
          .cta-btn { display: inline-block; background: #5B7CFA; color: #ffffff; text-decoration: none; padding: 12px 28px; border-radius: 6px; font-weight: bold; margin-top: 20px; }
          .footer { background: #f8fafc; padding: 20px; text-align: center; font-size: 12px; color: #8A93A6; }
        </style>
      </head>
      <body>
        <div class="container">
          <div class="header">
            <h1 style="margin:0; font-size: 24px;">MAIL-IA Campaign</h1>
          </div>
          <div class="content">
            <h2>Propuesta Generada para tu Campaña</h2>
            <p>Este correo ha sido diseñado automáticamente en base a tu indicación: <em>"${prompt}"</em>.</p>
            <p>Aprovecha esta oportunidad para conectar con tu audiencia con alta tasa de entregabilidad.</p>
            <center>
              <a href="#" class="cta-btn">Ver Promoción Exclusiva</a>
            </center>
          </div>
          <div class="footer">
            <p>Has recibido este correo porque estás suscrito a nuestra lista oficial.</p>
            <p><a href="#" style="color: #8A93A6;">Darse de baja</a></p>
          </div>
        </div>
      </body>
      </html>
    `;
  }
}
```

---

## 4. Configuración de Inyección de Dependencias (Cero Fricción al Migrar)

En el archivo de configuración de proveedores (`app.config.ts`), la inyección se resuelve mediante tokens:

```typescript
import { ApplicationConfig, InjectionToken } from '@angular/core';
import { provideHttpClient } from '@angular/common/http';
import { environment } from '../environments/environment';
import { AuthApiClient } from './core/api/auth-api.interface';
import { MockAuthApiService } from './core/mocks/mock-auth-api.service';
import { HttpAuthApiService } from './core/api/http-auth-api.service';
import { CampaignApiClient } from './core/api/campaign-api.interface';
import { MockCampaignApiService } from './core/mocks/mock-campaign-api.service';
import { HttpCampaignApiService } from './core/api/http-campaign-api.service';

export const AUTH_API_CLIENT = new InjectionToken<AuthApiClient>('AUTH_API_CLIENT');
export const CAMPAIGN_API_CLIENT = new InjectionToken<CampaignApiClient>('CAMPAIGN_API_CLIENT');

export const appConfig: ApplicationConfig = {
  providers: [
    provideHttpClient(),
    {
      provide: AUTH_API_CLIENT,
      useClass: environment.useMockApi ? MockAuthApiService : HttpAuthApiService
    },
    {
      provide: CAMPAIGN_API_CLIENT,
      useClass: environment.useMockApi ? MockCampaignApiService : HttpCampaignApiService
    }
  ]
};
```

---

## 5. Migración al Backend NestJS

Cuando el backend NestJS esté desplegado:
1. Se implementa `HttpCampaignApiService` utilizando `HttpClient` apuntando a `environment.apiUrl` (`/api/campaigns`).
2. Se cambia `environment.useMockApi: false`.
3. **Ningún componente de Angular requerirá modificación alguna.**
