# Plan de automatización n8n para video diario multiplataforma

Este plan define cómo implementar la automatización completa en n8n para crear y publicar videos diarios en TikTok, Instagram Reels, YouTube Shorts y Facebook Reels, con un panel web simple para controlar estilo, tono, duración y visual. No incluye código; es una guía operacional con estado por niveles.

## Estado por niveles (checklist operativo)
- Nivel 0 — Preparación n8n
  - ⬜ 0.1 Confirmar si hay acceso al ordenador local del usuario con n8n (requiere respuesta del usuario).
  - ⬜ 0.2 Priorizar uso de n8n local si existe.
  - ⬜ 0.3 Si no hay n8n local, instalar n8n local (Docker o npm) con mínimos pasos.
  - ⬜ 0.4 Si no es posible local, usar n8n Cloud (free trial) sin exceder presupuesto.
  - ⬜ 0.5 Abrir workspace y crear workflow base.
- Nivel 1 — Diseño funcional + presupuesto
  - ⬜ 1.1 Fijar meta: 3 videos/día/plataforma (12 diarios).
  - ⬜ 1.2 Definir modo Auto infinito y Manual puntual.
  - ⬜ 1.3 Definir panel con controles (tipo, tono, estilo visual, duración, cantidad diaria, plataformas, pausa/reanudar).
  - ⬜ 1.4 Definir criterio de calidad mínima y lógica de regeneración/salto.
  - ⬜ 1.5 Seleccionar herramientas baratas/gratis para tendencias, guion, voz, video, publicación.
  - ⬜ 1.6 Presupuesto mensual ≤ 50 USD.
- Nivel 2 — Panel simple con persistencia
  - ⬜ 2.1 Elegir panel web mínimo (HTML sencillo + Supabase free tier para persistencia).
  - ⬜ 2.2 Conectar panel → n8n para actualizar config y disparar workflow.
  - ⬜ 2.3 Verificar que cambios persisten (ej. educativo → gracioso) para siguientes ciclos.
- Nivel 3 — Núcleo workflow auto infinito
  - ⬜ 3.1 Scheduler diario (America/Bogota) con ventanas horarias.
  - ⬜ 3.2 Módulo tendencias (fuentes gratuitas).
  - ⬜ 3.3 Selección y scoring de ideas.
  - ⬜ 3.4 Guion: 3 guiones/día/plataforma (o base + adaptaciones).
  - ⬜ 3.5 Producción: hook 1–2s, ritmo alto, subtítulos grandes, loop.
  - ⬜ 3.6 Adaptación por plataforma (descripción/hashtags/ratio/cover).
  - ⬜ 3.7 Publicación: programar en TikTok/IG Reels/YouTube Shorts/Facebook Reels.
  - ⬜ 3.8 Logging con links y errores.
- Nivel 4 — Integraciones de publicación (logins)
  - ⬜ 4.1 Conectar TikTok (pedir toma de control para login).
  - ⬜ 4.2 Conectar Instagram Reels.
  - ⬜ 4.3 Conectar YouTube Shorts.
  - ⬜ 4.4 Conectar Facebook Reels.
  - ⬜ 4.5 Publicar prueba privada/no listada o real con consentimiento.
- Nivel 5 — Modo Manual
  - ⬜ 5.1 Botón “Crear 1 video ahora” en panel.
  - ⬜ 5.2 Inputs manuales: tema + estilo + duración + plataformas.
  - ⬜ 5.3 Generar y publicar ese video puntual.
  - ⬜ 5.4 Opción: aplicar config como nueva infinita.
- Nivel 6 — Control de calidad/seguridad
  - ⬜ 6.1 Gate de calidad previo (hook fuerte, ritmo, subtítulos legibles).
  - ⬜ 6.2 Filtro de temas bloqueados y copyright.
  - ⬜ 6.3 Modo seguro: parar y pedir confirmación si riesgo.
- Nivel 7 — Métricas y mejora
  - ⬜ 7.1 Guardar métricas básicas por post.
  - ⬜ 7.2 Ajuste automático de duración/hook/horarios.
  - ⬜ 7.3 Reporte diario/semanal en panel.

## Decisiones propuestas (prioridad gratis/barato, ≤ 50 USD/mes)
- n8n: preferir local (Docker compose mínimo) para costo 0; fallback n8n Cloud starter solo si necesario.
- Persistencia de configuración: Supabase free tier (tabla config + tabla logs) o Data Stores de n8n si está disponible.
- Panel simple: HTML estático (hosteado local o en Supabase storage) que escribe/lee config vía API REST de Supabase y dispara un Webhook de n8n (“Guardar y ejecutar infinito”, “Pausar”, “Ejecutar ahora”). Sin que el usuario toque n8n.
- LLM para guiones: OpenRouter con modelos económicos (ej. Llama 3.1 8B/70B según calidad) con límites; estimado < 10 USD/mes para 12 guiones/día + adaptaciones.
- Tendencias: fuentes gratuitas (YouTube Trends RSS, Google News RSS, Reddit RSS temático, scraping ligero con RSSHub o Apify free runs).
- TTS: OpenAI TTS “tts-1” (~0.015 USD/min) → 12 videos x 1 min x 30 días ≈ 5.4 USD/mes; alternativa eSpeak local si hay recorte de presupuesto.
- Generación visual: imágenes libres (Pexels/Unsplash API gratis) + subtítulos quemados y composición con ffmpeg local (nodo Execute Command).
- Subtítulos: generación en n8n con srt/vtt y quemado en ffmpeg; tipografía grande y high contrast.
- Publicación:
  - TikTok: Official Content Posting API (gratis; requiere app y login).
  - Instagram/Facebook Reels: Graph API (ig_user/media + publish; gratis; requiere Business Manager).
  - YouTube Shorts: YouTube Data API v3 (upload; gratis dentro de cuotas).
  - Programación: n8n scheduler + colas con redis opcional (local gratis).
- Logging y métricas: Supabase tables (posts, attempts, metrics) y panel que lea últimos registros y estado de scheduler.

## Flujo resumido (auto infinito)
1) Scheduler (timezone America/Bogota) genera lote diario con ventanas configurables.  
2) Tendencias → ranking → selecciona 3 ideas x plataforma.  
3) Guion base + adaptación por plataforma (hook 1–2s, CTA, loop).  
4) Producción: TTS + assets libres + ffmpeg para armar vertical 9:16, subtítulos grandes, cover por plataforma.  
5) Publicación programada por plataforma; reintentos en caso de error; logging de links/errores.  
6) Métricas (cuando APIs lo permitan) → ajuste automático (duración/horarios/hook).  
7) Panel: permite guardar config persistente, pausar, ejecutar ahora, crear video puntual (modo manual).

## Presupuesto mensual estimado (12 videos/día)
- n8n local: 0 USD (o n8n Cloud starter < 30 USD si se usa).
- LLM (OpenRouter económico): ~10 USD.
- TTS (OpenAI tts-1, 360 min/mes): ~5.4 USD.
- Hosting/panel + Supabase free tier: 0 USD.
- APIs sociales oficiales: 0 USD (solo esfuerzo de login).
- Margen para imprevistos: ~34 USD sobrantes si n8n es local; ~4 USD si se usa n8n Cloud starter.

## Entregables finales previstos (sin código)
- Panel web simple con botones Guardar y ejecutar infinito / Pausar / Ejecutar ahora y campos de configuración (tipo, tono, estilo visual, duración, cantidad diaria, plataformas activas).  
- Workflow n8n con scheduler diario, generación, producción, adaptación y publicación con logging y reintentos.  
- Guía de uso: cómo abrir panel, cambiar configuración, ver estado y re-login si se desconecta una red, solicitar consentimiento antes de la primera publicación pública.
