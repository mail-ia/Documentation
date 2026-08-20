# Contexto de Proyecto para Agente de IA — Desarrollo & Arquitectura Técnica
> **Plataforma:** AI Email Platform (MAIL-IA)  
> **Propósito:** Contexto integral y directrices de ingeniería para Agentes de Inteligencia Artificial de desarrollo de software (Cursor, Claude Code, Antigravity, GitHub Copilot, Windsurf, etc.).  
> **Objetivo:** Garantizar la construcción de código limpio, modular, tipado estrictamente, altamente testeable y desacoplado según Clean Architecture y DDD.

---

## 1. Visión General del Producto & Arquitectura del Sistema

**MAIL-IA** es una plataforma SaaS B2B de alta disponibilidad para la **creación conversacional, maquetación, despacho asíncrono y telemetría analítica de campañas de correo masivo asistidas por IA**.

### 1.1 Problema que Resuelve
Eliminar la intermediación manual entre los departamentos de marketing y los equipos de desarrollo. Permite a usuarios no técnicos diseñar plantillas HTML de alta calidad conversando con un modelo de IA en lenguaje natural, garantizando entregabilidad con estándares anti-spam (SPF/DKIM/DMARC) e infraestructura desacoplada.

### 1.2 Arquitectura Global (Modular Monolith Desacoplado)
El sistema opera bajo una arquitectura de **Monolito Modular** estructurado con **Clean Architecture** y **Domain-Driven Design (DDD)**.

```text
[ Angular Frontend App ] 
         │  (HTTP / REST / JSON)
         ▼
[ NestJS Backend API Core ]
   ├── Dominio (Entidades, Value Objects, Reglas de Negocio)
   ├── Aplicación (Casos de Uso, Puertos/Interfaces, DTOs)
   └── Infraestructura (Adaptadores Prisma, Neon Postgres, BullMQ, Resend, Groq)
         │
         ├── [ Base de Datos Neon PostgreSQL ] (Vía Prisma ORM)
         ├── [ Cola Redis / BullMQ ] (Procesamiento Asíncrono)
         │         │
         │         ▼
         ├── [ Background Workers ] ──► [ Email Provider: Resend / Postmark ]
         ├── [ AI Gateway Adapter ] ──► [ AI Engine: Groq / OpenRouter ]
         └── [ Storage Adapter ]    ──► [ Cloudflare R2 ]
```

### 1.3 Principio de Separación de Repositorios
1. **Repositorio Frontend:** Aplicación cliente en Angular.
2. **Repositorio Backend:** API NestJS, Casos de Uso, Cola y Workers.
* Ambos repositorios deben compilarse, testearse, versionarse y desplegarse de manera 100% independiente mediante contenedores **Docker**.

---

## 2. Stack Tecnológico Oficial

| Capa / Componente | Tecnología Seleccionada | Justificación & Estándar |
| :--- | :--- | :--- |
| **Backend Core** | **NestJS + TypeScript** | Inyección de dependencias, controladores modulares, Clean Architecture. |
| **Base de Datos** | **PostgreSQL (Neon Serverless)** | Relacional, escalable a cero en inactividad, transacciones seguras. |
| **ORM / Acceso a Datos** | **Prisma ORM** | Modelado estricto, migraciones reproducibles, tipado generado. |
| **Colas & Workers** | **BullMQ / Redis** | Despacho asíncrono no bloqueante de correos y tareas pesadas. |
| **Frontend Web** | **Angular + TypeScript** | Separación forzada de presentación y lógica mediante Smart/Dumb pattern. |
| **Testing Backend** | **Jest** | TDD riguroso, >80% de cobertura en Casos de Uso. |
| **Testing Frontend & E2E** | **Playwright** | Pruebas end-to-end de flujos críticos (login, chat, envío). |
| **Proveedor de Email** | **Resend / Postmark** | Aislado tras interfaz `EmailSender` (SPF, DKIM, DMARC). |
| **Motor de IA** | **Groq / OpenRouter** (Qwen 2.5 / Llama 3.1) | Inferencia ultrarrápida, generación estructurada JSON/HTML, bajo coste. |
| **Storage de Imágenes** | **Cloudflare R2** | Almacenamiento S3-compatible con cero costes de egreso (*zero egress*). |
| **Contenedores** | **Docker & Docker Compose** | Entornos de desarrollo y producción estandarizados. |

---

## 3. Reglas de Ingeniería & Clean Code (No Negociables)

Cualquier código generado por un agente o desarrollador debe cumplir obligatoriamente estas reglas:

### 3.1 Complejidad Ciclomática Máxima: 4
* Ninguna función debe superar una complejidad ciclomática de **4**.
* Si una función acumula múltiples condicionales o bifurcaciones, debe descomponerse en funciones auxiliares puras y enfocadas (ej. `validate()`, `transform()`, `execute()`, `persist()`).

### 3.2 Límite de Tamaño de Archivo: 200 – 300 Líneas
* Los archivos fuente deben mantenerse preferentemente por debajo de **200 a 300 líneas**.
* Un archivo extenso es un indicador directo de violación del Principio de Responsabilidad Única (SRP).

### 3.3 Tipado Estricto — Prohibición Total de `any`
* El uso de `any` está **terminantemente prohibido**.
* Todo parámetro, valor de retorno, DTO, entidad, respuesta HTTP y configuración externa debe contar con un tipo o interfaz explícita.

### 3.4 Nombres Descriptivos sin Abreviaturas Crípticas
* Los identificadores deben comunicar intención clara.
* ❌ Prohibido: `const cList = ...`, `const d = ...`, `const proc = ...`
* ✅ Obligatorio: `const campaignList = ...`, `const deliveryStatus = ...`, `const campaignProcessor = ...`

### 3.5 TDD (Test-Driven Development) & Aislamiento de Pruebas
* Los casos de uso de negocio deben mantener al menos **80% de cobertura unitaria**.
* Las pruebas unitarias **jamás deben conectarse a infraestructura real**. Todas las bases de datos, APIs de email, proveedores de IA y colas deben ser simuladas mediante mocks/stubs de sus interfaces.

---

## 4. Reglas Específicas por Capa

### 4.1 Backend (NestJS & Clean Architecture)
1. **Dirección de Dependencias hacia el Interior:**
    * `Dominio` no conoce `Aplicación`, ni `Infraestructura`, ni `NestJS`, ni `Prisma`.
    * `Aplicación` solo conoce `Dominio` y define `Puertos` (interfaces).
    * `Infraestructura` implementa los `Puertos` e inyecta librerías externas (NestJS, Prisma, Resend, Groq).
2. **Ports and Adapters (Arquitectura Hexagonal):**
   ```typescript
   // Dominio / Aplicación define el puerto:
   export interface CampaignRepository {
     save(campaign: Campaign): Promise<void>;
     findById(id: CampaignId): Promise<Campaign | null>;
   }

   // Infraestructura implementa el adaptador:
   @Injectable()
   export class PrismaCampaignRepository implements CampaignRepository {
     constructor(private readonly prisma: PrismaService) {}
     async save(campaign: Campaign): Promise<void> { /* ... */ }
     async findById(id: CampaignId): Promise<Campaign | null> { /* ... */ }
   }
   ```
3. **Despacho Asíncrono de Correos:**
    * La API HTTP **nunca debe enviar correos masivos de forma sincrónica** dentro del ciclo de vida de la petición.
    * El controlador valida la campaña, crea un `DeliveryJob`, lo encola en BullMQ y retorna un `202 Accepted` inmediato con el ID del trabajo.
    * Los `Workers` consumen la cola en segundo plano, despachan en lotes controlados y actualizan la telemetría.

### 4.2 Frontend (Angular)
1. **Smart Services vs Dumb Components:**
    * **Componentes:** Solo renderizan datos, capturan interacciones del usuario y emiten eventos. Cero lógica de negocio, cero inyecciones directas de `HttpClient` para operaciones complejas.
    * **Servicios:** Gestionan llamadas HTTP, transformación de DTOs a modelos de vista, estado reactivo y orquestación.
2. **Aislamiento Absoluto de la IA:**
    * El frontend **nunca debe invocar directamente a Groq, OpenAI o OpenRouter**.
    * Todo mensaje del chat viaja hacia el Backend (`POST /api/campaigns/ai-assist`), donde se valida la sesión, se sanitiza el prompt y se consulta al adaptador de IA.
3. **Seguridad & Sanitización HTML:**
    * El HTML generado por la IA es **contenido no confiable**.
    * Queda estrictamente prohibido usar `DomSanitizer.bypassSecurityTrustHtml()` o `element.innerHTML` sin un proceso formal de sanitización previa.
    * La previsualización de plantillas debe ejecutarse en un entorno seguro (ej. sandbox iframe o pipeline de sanitización estricto).
4. **Cero Secretos en el Cliente:**
    * Ninguna API Key (Resend, Groq, tokens de BD) debe existir en `environment.ts` ni en el bundle JS del navegador.
    * La autenticación preferente es mediante cookies `HttpOnly` y `Secure` con protección CSRF en endpoints de mutación (`POST`, `PUT`, `DELETE`).

---

## 5. Módulos del Sistema & Flujo de Pantallas (UI Mockups)

El diseño de la aplicación está inspirado en la estética minimalista y espaciosa de **Claude (Anthropic)**:
* **Paleta:** Base 60% Azul Noche (`#0B1120`), Superficies secundarias (`#131B2E`), Bordes sutiles (`#1E2536`), Texto blanco/gris (`#FFFFFF` / `#8A93A6`), Acento Índigo moderado (`#5B7CFA`).
* **Estructura:** Sidebar fija a la izquierda (angosta con SVG line icons) + Área principal amplia con alto whitespace.

### Pantallas Principales:
1. **Login & Onboarding (`/login`):** Acceso limpio con correo/contraseña y soporte SSO (Google/Microsoft).
2. **Dashboard General (`/dashboard`):** 4 KPIs cuantitativos (Campañas Activas, Envíos del Mes, Tasa de Apertura %, Total Contactos), gráfica de rendimiento y accesos rápidos.
3. **Chat de Creación IA (`/campaigns/new` — Pantalla Insignia):**
    * *Panel Izquierdo (45%):* Chat conversacional con la IA, chips de sugerencias rápidas ("Más formal", "Insertar oferta", "Cambiar paleta"), input con soporte para adjuntar imágenes.
    * *Panel Derecho (55%):* Previsualización en vivo en tiempo real con pestañas `Preview`, `HTML` y `Texto Plano`, y botón "Continuar al Envío".
4. **Selector de Plantillas (`/templates`):** Catálogo visual en cuadrícula de diseños históricos, filtros por categoría y modal de subida de plantillas personalizadas.
5. **Biblioteca Multimedia (`/media`):** Gestión de imágenes y recursos alojados en Cloudflare R2 para incrustar en correos.
6. **Configuración de Envío (`/campaigns/:id/send`):** Stepper guiado (Diseño ➔ Destinatarios ➔ Programación ➔ Revisión), selector de listas/categorías, programación inmediata o diferida, y panel sticky de resumen.
7. **Gestión de Contactos & Audiencias (`/contacts`):** Tabla de clientes con importador masivo CSV, filtros por estado y panel deslizable de perfil individual.
8. **Categorías de Clientes (`/categories`):** Etiquetas de segmentación dinámica con colores identificadores.
9. **Seguimiento & Telemetría (`/campaigns/:id/analytics`):** Métricas en tiempo real de aperturas, clics, rebotes, mapa de calor de enlaces y exportación de reportes.
10. **Ajustes & Configuración (`/settings`):** Verificación de dominios (DKIM, SPF, DMARC), gestión de credenciales API, notificaciones y seguridad.

---

## 6. Modelo de Dominio & Entidades Principales

```typescript
// Entidades conceptuales centrales del dominio:

export interface Campaign {
  id: string;
  userId: string;
  name: string;
  subject: string;
  senderName: string;
  replyToEmail: string;
  templateHtml: string;
  templateJsonStructure?: Record<string, unknown>;
  status: 'DRAFT' | 'SCHEDULED' | 'PROCESSING' | 'SENT' | 'FAILED';
  scheduledAt?: Date;
  sentAt?: Date;
  audienceIds: string[];
  categoryIds: string[];
  metrics: CampaignMetrics;
  createdAt: Date;
  updatedAt: Date;
}

export interface CampaignMetrics {
  totalRecipients: number;
  deliveredCount: number;
  openedCount: number;
  clickedCount: number;
  bouncedCount: number;
  unsubscribedCount: number;
}

export interface Contact {
  id: string;
  userId: string;
  email: string;
  fullName: string;
  categoryIds: string[];
  status: 'ACTIVE' | 'UNSUBSCRIBED' | 'BOUNCED';
  customAttributes?: Record<string, string>;
  createdAt: Date;
}

export interface Category {
  id: string;
  userId: string;
  name: string;
  colorHex: string;
  description?: string;
}

export interface Template {
  id: string;
  userId: string;
  name: string;
  thumbnailUrl?: string;
  htmlContent: string;
  jsonStructure: Record<string, unknown>;
  category: string;
  createdAt: Date;
}

export interface DeliveryJob {
  id: string;
  campaignId: string;
  batchIndex: number;
  recipientBatch: Array<{ email: string; name: string }>;
  status: 'QUEUED' | 'SENDING' | 'COMPLETED' | 'RETRYING' | 'FAILED';
  attempts: number;
}
```

---

## 7. Instrucciones Operativas para el Agente IA de Desarrollo

Cuando se te solicite implementar código o refactorizar:
1. **Comienza siempre por los tipos e interfaces del dominio.**
2. **Escribe o planifica primero las pruebas unitarias (TDD) para casos de uso.**
3. **No utilices jamás `any`. Define tipos específicos para cada DTO y respuesta.**
4. **Verifica que ninguna función sobrepase 4 de complejidad ciclomática.**
5. **Mantén los componentes Angular 'tontos' (Dumb) y traslada la lógica a Servicios.**
6. **Asegúrate de que todo servicio externo (email, IA, DB) se acceda mediante abstracciones e inyección de dependencias.**
