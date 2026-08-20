# Tarea 03 — Chat de Creación Asistida por IA & Live Preview (Pantalla Insignia)

> **Módulo:** `features/campaigns/chat-editor`  
> **Ruta:** `/campaigns/new`  
> **Prioridad:** Crítica (El corazón y elemento diferenciador del producto)  
> **Objetivo:** Implementar la interfaz de creación conversacional dividida en dos paneles: Panel izquierdo (45%) para chatear con la IA con chips de ajuste rápido, y Panel derecho (55%) para la previsualización en vivo en tiempo real de la plantilla generada con pestañas de Preview, HTML y Texto Plano.

---

## 1. Arquitectura de Pantalla (Split Screen 45% / 55%)

La pantalla se distribuye en dos áreas de trabajo sincronizadas reactivamente mediante Signals:

```text
┌──────────────────────────────────────────────┬─────────────────────────────────────────────────┐
│        PANEL DE CHAT CONVERSACIONAL (45%)    │           PANEL DE LIVE PREVIEW (55%)           │
├──────────────────────────────────────────────┼─────────────────────────────────────────────────┤
│ [Historial de Conversación]                  │ [Topbar Preview]                                │
│                                              │ Tabs: [◉ Preview] [<> HTML] [≡ Texto]           │
│ 🤖 IA: ¡Hola Carlos! ¿Qué campaña deseas     │ Vista: [🖥 Desktop] [📱 Mobile]  [Guardar] [✈]  │
│        crear hoy?                            ├─────────────────────────────────────────────────┤
│                                              │                                                 │
│ 👤 Tú: Diseña un correo para oferta de       │  ┌───────────────────────────────────────────┐  │
│        verano con 20% de descuento.          │  │ [Iframe Sandbox Seguro]                   │  │
│                                              │  │                                           │  │
│ 🤖 IA: He generado un diseño moderno con un  │  │   ┌───────────────────────────────────┐   │  │
│        banner vibrante y botón de compra.    │  │   │  MAIL-IA Campaign                 │   │  │
│                                              │  │   ├───────────────────────────────────┤   │  │
│ [Chips de Ajuste Rápido]                     │  │   │  OFERTA EXCLUSIVA DE VERANO       │   │  │
│ [✦ Más formal] [🏷 20% OFF] [🎨 Paleta Azul] │  │   │  20% de descuento en catálogo     │   │  │
│                                              │  │   │  [ Comprar Ahora (CTA) ]          │   │  │
├──────────────────────────────────────────────┤  │   └───────────────────────────────────┘   │  │
│ [Input Textarea: "Describe tu correo..."]    │  │                                           │  │
│ [📎 Adjuntar Imagen]         [➤ Enviar Prompt]│  └───────────────────────────────────────────┘  │
└──────────────────────────────────────────────┴─────────────────────────────────────────────────┘
```

---

## 2. Especificación Detallada de Componentes

### 2.1 Panel Izquierdo: Conversación con la IA (45% Ancho)
* **Contenedor de Mensajes (`MessageListComponent`):**
  * Burbujas de chat diferenciadas:
    * **Usuario:** Alineada a la derecha, fondo `#5B7CFA`, texto blanco.
    * **Asistente IA:** Alineada a la izquierda, fondo `#131B2E`, borde `1px solid #1E2536`, avatar de IA.
  * **Indicador de Generación / Escribiendo:** Animación de 3 puntos o barra de progreso cuando `isGenerating()` sea `true`.
  * Auto-scroll automático hacia el último mensaje.
* **Barra de Chips de Sugerencia Rápida (`PromptChipsComponent`):**
  * Chips interactivos de un solo clic:
    * *"Hacer más formal y corporativo"*
    * *"Insertar botón de llamada a la acción (CTA)"*
    * *"Cambiar paleta a colores oscuros"*
    * *"Agregar código de cupón VERANO20"*
    * *"Referenciar recursos de marca / logo"*
  * Al hacer clic en un chip, se envía automáticamente como prompt o se inserta en el input.
* **Caja de Entrada de Prompt (`PromptInputComponent`):**
  * Textarea auto-expandible con placeholder *"Describe el correo que quieres crear o pide modificaciones..."*.
  * Botón para adjuntar imagen o recurso (simulación de selector de archivo).
  * Botón de envío con icono de flecha/avión. Deshabilitado si el input está vacío o `isGenerating()` es `true`.
  * Atajo de teclado: `Enter` para enviar, `Shift + Enter` para salto de línea.

### 2.2 Panel Derecho: Live Preview en Vivo (55% Ancho)
* **Barra de Herramientas Superior (`PreviewToolbarComponent`):**
  * **Selector de Pestañas:**
    1. **`Preview` (Visual):** Renderizado del correo en tiempo real.
    2. **`HTML` (Código):** Visualizador del HTML generado con botón *"Copiar Código"*.
    3. **`Texto Plano` (Fallback):** Versión texto sin formato del correo.
  * **Selector de Dispositivo:**
    * **Desktop:** Contenedor de `600px` centrado con sombra.
    * **Mobile:** Contenedor de `375px` con marco simulado de smartphone.
  * **Botones de Acción:**
    * *"Guardar Borrador"* (almacena el estado en el servicio mock).
    * *"Continuar al Envío"* (abre modal de confirmación o redirige a `/campaigns/send`).
* **Contenedor de Visualización Seguro (`LivePreviewSandboxComponent`):**
  * **Seguridad Obligatoria:** Uso de `<iframe sandbox="allow-same-origin" [srcdoc]="currentDraft().templateHtml"></iframe>`.
  * Aislamiento total de estilos para que el CSS del correo no afecte la aplicación.
  * Estado vacío (*Empty State*) elegante con ilustración cuando aún no se ha generado ninguna plantilla.

---

## 3. Gestión de Estado con Signals (`AIChatStateService`)

El servicio de estado administra la reactividad de la pantalla:

```typescript
@Injectable({ providedIn: 'root' })
export class AIChatStateService {
  // Signals de estado
  readonly messages = signal<ChatMessage[]>([]);
  readonly currentDraft = signal<CampaignDraft | null>(null);
  readonly isGenerating = signal<boolean>(false);
  readonly activeTab = signal<'PREVIEW' | 'HTML' | 'PLAIN_TEXT'>('PREVIEW');
  readonly previewDevice = signal<'DESKTOP' | 'MOBILE'>('DESKTOP');

  // Computed signals
  readonly hasDraft = computed(() => this.currentDraft() !== null);
  readonly formattedHtml = computed(() => this.currentDraft()?.templateHtml ?? '');
  readonly plainTextContent = computed(() => this.currentDraft()?.plainText ?? '');

  // Acciones
  async submitPrompt(prompt: string): Promise<void>;
  setTab(tab: 'PREVIEW' | 'HTML' | 'PLAIN_TEXT'): void;
  setDevice(device: 'DESKTOP' | 'MOBILE'): void;
  saveCurrentDraft(): Promise<void>;
}
```

---

## 4. Requisitos de Pruebas Obligatorias (DoD de la Tarea)

El agente debe implementar y ejecutar las siguientes pruebas antes de dar por terminada la tarea:

### 4.1 Pruebas Unitarias (`chat-editor.component.spec.ts` & `ai-chat-state.service.spec.ts`)
1. **Envío de Mensajes:** Comprobar que al enviar un prompt se agregue el mensaje del usuario y se active `isGenerating(true)`.
2. **Actualización de Plantilla:** Verificar que tras la respuesta mock de la IA, el Signal `currentDraft` se actualice y se renderice el nuevo HTML.
3. **Cambio de Pestañas:** Verificar que al alternar entre `Preview`, `HTML` y `Texto Plano` cambie la visualización correspondiente.
4. **Cambio de Dispositivo:** Comprobar que al cambiar a `MOBILE`, el contenedor reduzca su ancho a `375px`.
5. **Chips de Ajuste:** Verificar que al hacer clic en un chip de sugerencia se ejecute la solicitud correspondiente.

### 4.2 Pruebas E2E con Playwright (`e2e/chat-ai-preview.spec.ts`)
1. Iniciar sesión y navegar a `/campaigns/new`.
2. Escribir un prompt en el textarea (*"Campaña de lanzamiento con descuento"*) y presionar Enviar.
3. Validar que aparezca la burbuja del usuario y el indicador de carga.
4. Validar que se reciba la respuesta de la IA y el iframe de Live Preview renderice el contenido.
5. Hacer clic en la pestaña `HTML` y validar que se visualice el bloque de código fuente.
6. Hacer clic en el selector `Mobile` y validar la reducción de dimensiones.
7. Hacer clic en el chip rápido *"Hacer más formal"* y validar que se genere una actualización del correo.

---

## 5. Checklist de Entrega para el Agente

- [ ] Split screen 45% / 55% maquetado con la paleta de colores oficial.
- [ ] Burbujas de mensajes de usuario e IA con timestamps e iconos.
- [ ] Barra de chips de sugerencias rápidas operativas.
- [ ] Textarea con envío mediante tecla `Enter` y botón `➤`.
- [ ] Iframe sandbox seguro para Live Preview sin uso de `bypassSecurityTrustHtml`.
- [ ] Pestañas funcionales: `Preview`, `HTML` y `Texto Plano`.
- [ ] Toggle responsivo Desktop / Mobile en el preview.
- [ ] `MockCampaignApiService` conectado mediante Inyección de Dependencias.
- [ ] Pruebas unitarias aprobadas (>80% cobertura).
- [ ] Pruebas E2E de Playwright ejecutadas y aprobadas.
- [ ] Cero uso de `any` y archivos < 300 líneas.
