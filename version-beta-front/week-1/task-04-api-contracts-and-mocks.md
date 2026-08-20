# Tarea 04 — Contratos de API, Modelos de Datos & Mock Services

> **Módulo:** `core/models`, `core/api`, `core/mocks`  
> **Prioridad:** Bloqueante (Base de tipado y desacoplamiento para todas las tareas de la Semana 1)  
> **Objetivo:** Definir todos los DTOs, interfaces de dominio, contratos de API REST y servicios mock listos para usar en Angular, garantizando que el frontend esté 100% preparado para conectarse a NestJS sin modificar componentes.

---

## 1. Mapeo de Endpoints REST (Frontend Mock ➔ Backend NestJS)

| Endpoint | Método HTTP | Propósito | Request DTO | Response DTO |
| :--- | :--- | :--- | :--- | :--- |
| `/api/auth/login` | `POST` | Iniciar sesión | `LoginRequestDto` | `LoginResponseDto` |
| `/api/auth/register` | `POST` | Registro de empresa/usuario | `RegisterRequestDto` | `RegisterResponseDto` |
| `/api/auth/me` | `GET` | Obtener perfil activo | — | `UserProfileDto` |
| `/api/auth/recover` | `POST` | Solicitar recuperación clave | `{ email: string }` | `{ success: boolean; message: string }` |
| `/api/campaigns/ai-assist` | `POST` | Enviar prompt al motor de IA | `AiPromptRequestDto` | `AiPromptResponseDto` |
| `/api/campaigns/draft` | `POST` | Guardar borrador de campaña | `SaveDraftRequestDto` | `CampaignDto` |
| `/api/campaigns/:id` | `GET` | Obtener campaña por ID | — | `CampaignDto` |
| `/api/campaigns/recent` | `GET` | Listar campañas recientes | — | `CampaignDto[]` |

---

## 2. Contratos de Tipos e Interfaces TypeScript

### 2.1 Modelos de Autenticación & Usuario (`core/models/auth.models.ts`)
```typescript
export interface UserProfileDto {
  id: string;
  email: string;
  fullName: string;
  companyName: string;
  role: 'ADMIN' | 'MARKETING_LEAD' | 'EDITOR';
  avatarUrl?: string;
  createdAt: string;
}

export interface LoginRequestDto {
  email: string;
  password: string;
  rememberMe?: boolean;
}

export interface LoginResponseDto {
  token: string;
  expiresIn: number;
  user: UserProfileDto;
}

export interface RegisterRequestDto {
  fullName: string;
  companyName: string;
  email: string;
  password: string;
}

export interface RegisterResponseDto {
  token: string;
  user: UserProfileDto;
  message: string;
}
```

### 2.2 Modelos de Campaña & Chat IA (`core/models/campaign.models.ts`)
```typescript
export type CampaignStatus = 'DRAFT' | 'SCHEDULED' | 'PROCESSING' | 'SENT' | 'FAILED';

export interface CampaignMetricsDto {
  totalRecipients: number;
  deliveredCount: number;
  openedCount: number;
  clickedCount: number;
  bouncedCount: number;
  unsubscribedCount: number;
}

export interface CampaignDraft {
  id: string;
  name: string;
  subject: string;
  templateHtml: string;
  plainText: string;
  updatedAt: string;
}

export interface CampaignDto {
  id: string;
  userId: string;
  name: string;
  subject: string;
  senderName: string;
  replyToEmail: string;
  templateHtml: string;
  status: CampaignStatus;
  scheduledAt?: string;
  sentAt?: string;
  audienceIds: string[];
  categoryIds: string[];
  metrics: CampaignMetricsDto;
  createdAt: Date;
  updatedAt: Date;
}

export interface ChatMessage {
  id: string;
  sender: 'USER' | 'AI';
  content: string;
  timestamp: Date;
  metadata?: {
    actionType?: 'TONE_CHANGE' | 'CTA_INSERT' | 'PALETTE_CHANGE' | 'IMAGE_ATTACH';
  };
}

export interface AiPromptRequestDto {
  campaignId?: string;
  prompt: string;
  currentHtml?: string;
  contextParams?: {
    tone?: 'FORMAL' | 'CASUAL' | 'URGENT' | 'MINIMAL';
    includeOffer?: boolean;
    brandColorHex?: string;
  };
}

export interface AiPromptResponseDto {
  messageId: string;
  replyText: string;
  draft: CampaignDraft;
}

export interface SaveDraftRequestDto {
  id?: string;
  name: string;
  subject: string;
  templateHtml: string;
}
```

---

## 3. Fixtures y Datos Estáticos Simulados (`core/mocks/mock-fixtures.ts`)

```typescript
import { UserProfileDto } from '../models/auth.models';
import { CampaignDto } from '../models/campaign.models';

export const MOCK_USER: UserProfileDto = {
  id: 'usr-001',
  email: 'demo@mail-ia.com',
  fullName: 'Carlos Mendoza',
  companyName: 'TechCorp Solutions',
  role: 'MARKETING_LEAD',
  avatarUrl: 'https://api.dicebear.com/7.x/avataaars/svg?seed=Carlos',
  createdAt: '2026-01-15T10:00:00Z'
};

export const MOCK_INITIAL_HTML = `
<!DOCTYPE html>
<html>
<head>
  <meta charset="utf-8">
  <style>
    body { font-family: 'Helvetica Neue', Helvetica, Arial, sans-serif; background-color: #f1f5f9; margin: 0; padding: 24px; }
    .card { max-width: 580px; margin: 0 auto; background: #ffffff; border-radius: 12px; overflow: hidden; box-shadow: 0 4px 12px rgba(0,0,0,0.06); }
    .header { background: #0B1120; color: #ffffff; padding: 32px 24px; text-align: center; }
    .header h1 { margin: 0; font-size: 22px; letter-spacing: -0.5px; }
    .body { padding: 32px 24px; color: #334155; line-height: 1.6; font-size: 15px; }
    .cta-container { text-align: center; margin: 28px 0; }
    .cta-btn { display: inline-block; background-color: #5B7CFA; color: #ffffff !important; text-decoration: none; padding: 14px 32px; border-radius: 8px; font-weight: 600; }
    .footer { background: #f8fafc; padding: 20px; text-align: center; font-size: 12px; color: #94a3b8; border-top: 1px solid #e2e8f0; }
  </style>
</head>
<body>
  <div class="card">
    <div class="header">
      <h1>🚀 Novedades Exclusivas de Verano</h1>
    </div>
    <div class="body">
      <p>Hola <strong>{{nombre}}</strong>,</p>
      <p>Nos complace presentarte nuestra nueva colección diseñada para potenciar los resultados de tu negocio.</p>
      <p>Disfruta de un <strong>20% de descuento</strong> en todos nuestros servicios utilizando el código promocional en tu próximo pedido.</p>
      <div class="cta-container">
        <a href="https://mail-ia.com" class="cta-btn">Aprovechar Descuento</a>
      </div>
    </div>
    <div class="footer">
      <p>© 2026 TechCorp Solutions. Todos los derechos reservados.</p>
      <p>Si no deseas recibir más correos, haz <a href="#" style="color:#5B7CFA;">clic aquí</a>.</p>
    </div>
  </div>
</body>
</html>
`;

export const MOCK_CAMPAIGN_TEMPLATES: CampaignDto[] = [
  {
    id: 'cmp-001',
    userId: 'usr-001',
    name: 'Campaña Lanzamiento Verano',
    subject: 'Aprovecha 20% de descuento en tu próxima suscripción',
    senderName: 'Carlos de MAIL-IA',
    replyToEmail: 'carlos@mail-ia.com',
    templateHtml: MOCK_INITIAL_HTML,
    status: 'DRAFT',
    audienceIds: ['aud-01'],
    categoryIds: ['cat-01'],
    metrics: {
      totalRecipients: 1250,
      deliveredCount: 1240,
      openedCount: 680,
      clickedCount: 290,
      bouncedCount: 10,
      unsubscribedCount: 2
    },
    createdAt: new Date(),
    updatedAt: new Date()
  }
];
```

---

## 4. Implementación de Servicios Mock Intercambiables

Los servicios mock (`MockAuthApiService` y `MockCampaignApiService`) deben implementar exactamente las interfaces `AuthApiClient` y `CampaignApiClient`.

### 4.1 Simulación de Errores y Casos Límite
Para facilitar pruebas de interfaz, los servicios mock deben permitir la simulación de errores configurando flags o mediante parámetros especiales:
* **Credenciales inválidas:** Si el password no es `Password123!`, retornar `throwError(() => new Error('Credenciales incorrectas'))`.
* **Retardo de red:** Todas las respuestas deben encadenarse con `.pipe(delay(400 - 800))` para comprobar el comportamiento de spinners y botones deshabilitados en los componentes.
