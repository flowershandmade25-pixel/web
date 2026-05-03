# 📋 Resumen del Proyecto: Flowers Handmade · Sistema v1.14

## 🎯 Descripción general

Sistema web tipo SPA (single-page app) en un solo archivo HTML (`index.html`) para **Flowers Handmade**, un negocio de flores eternas hechas a mano ubicado en Loja, Ecuador. Eslogan: *"Detalles que perduran"*.

**Versión actual:** v1.14.0 · Build 2026-05-03 10:00

---

## 🏗️ Arquitectura

- **Frontend puro:** HTML + CSS + JavaScript vanilla (sin frameworks)
- **Persistencia dual:**
  - `localStorage` como respaldo local (clave `fh_v6`, `fh_saved_catalogs`, `fh_saved_proformas`, `fh_saved_publications`)
  - **Supabase** como backend en la nube (URL: `owfetilyxgvkrtdopskz.supabase.co`)
- **Autenticación:** Supabase Auth con login bloqueante (sin login no se puede usar la app)
- **Datos por usuario:** todo se filtra por `user.id` en lugar de un session_id global
- **Sincronización:** automática con badge visual de estado (Conectado / Sin conexión / Verificando)
- **Tablas Supabase:** `app_settings`, `catalogs`, `proformas`, `post_drafts`
- **PDFs:** generados con jsPDF
- **Imágenes/Posts:** generados en `<canvas>` HTML5 con edición visual interactiva
- **Responsive:** mobile-first desde v1.14, totalmente usable en celular

---

## 🔐 Autenticación

- Pantalla de login bloqueante full-screen al abrir la app
- Solo login con email + password (sin signup, las cuentas las crea el admin desde Supabase)
- Botón "Cerrar sesión" en la barra superior
- Cada usuario ve solo SUS datos (productos, catálogos, proformas, publicaciones)

---

## ⚙️ Modal central "Datos del negocio"

Botón en la barra superior. Centraliza todos los datos del negocio en un solo lugar:

**Datos públicos:**
- WhatsApp, Redes sociales, Dirección, Slogan
- Texto de envíos (editable, antes hardcodeado)
- Sitio web / Email comercial

**Datos fiscales (proformas):**
- Razón social, RUC/CI, Teléfono, Dirección, Email

Sincroniza con los inputs de cada pestaña. Persistencia automática en Supabase y localStorage.

---

## 📑 Las 4 pestañas (tabs) del sistema

### 1. Productos — Banco maestro
- Catálogo central de productos del negocio
- Campos: nombre, descripción, categoría (tag), SKU, precio, descuento %, precio mayoreo, cantidad mínima, foto
- 15 categorías predefinidas con íconos emoji
- Edición/eliminación con confirmación, filtros, contador de stock

### 2. Catálogos — Generador de PDF
- Selecciona productos vía modal "picker"
- Configuración de portada: título, mensaje, fecha, vigencia, público, moneda
- Color customizer completo (9 variables) y 7 paletas predefinidas
- 6 estilos de tarjeta: classic, horizontal, poster, minimal, price, bold
- Override por producto (personalizar tarjeta individual)
- Generación de PDF con 1-6 productos por página, vertical/horizontal
- Guardado/carga de catálogos completos

### 3. Proformas — Cotizaciones
- Datos proveedor (Flowers Handmade pre-cargado) y cliente
- Tabla de items: cantidad, precio unitario, descuento, total
- Cálculo automático: subtotal, descuento global, IVA, total
- Generación de PDF profesional con bandas decorativas y firmas
- Guardado/carga de proformas

### 4. Publicaciones — Posts para redes sociales

**5 plantillas:**
- Producto destacado, Carrusel, Frase, Promo/Oferta, **Collage** (2-10 fotos)

**8 paletas de color:** Nude, Rosa, Crema, Sage, Azul, Negro, Dorado, Lila

**Destinos:** Instagram, Facebook, WhatsApp (afecta caption y formato)

**Edición visual completa (todos los elementos editables con mouse/touch):**

Todo se edita arrastrando directamente en el lienzo:

| Acción | Resultado |
|---|---|
| Sin tocar nada | Borde casi invisible (rosa muy tenue) |
| Hover sobre elemento | Borde rosa palo punteado + mini etiqueta arriba |
| **Click** sobre elemento | Selecciona (borde azul + tirador + etiqueta) |
| **Drag** desde el cuerpo | Mueve el elemento |
| **Drag** desde el tirador (esquina inferior derecha) | Redimensiona |
| **Rueda del mouse** | Redimensiona también |
| **Doble click** | Resetea ese elemento |
| **Click en zona vacía** | Deselecciona |
| **Botón 👁️‍🗨️ Vista limpia** | Oculta todos los bordes para ver el resultado real |

**Elementos editables (TODOS con mouse/touch):**
- Logo
- Foto principal del producto
- Fotos lifestyle (1-4, individualmente movibles)
- Bloque inferior (dirección, envíos, CTA, @handle) — bbox calculado del contenido real
- Celdas del collage
- **TODOS los textos de las 5 plantillas:**
  - Destacado: subtítulo, título, tag (chip), descripción, precio (chip)
  - Carrusel: título principal, subtítulo
  - Frase: frase principal, título del producto
  - Promo: badge "OFERTA" rotado, "¡Aprovecha hoy!", título, precio anterior, precio actual
  - Collage: título, subtítulo

**Solapamientos resueltos:** cuando un texto y una foto se cruzan, gana siempre el elemento más pequeño (el texto sobre la foto).

**Click en texto en el lienzo** → auto-selecciona en el panel "Ajustar textos" para edición fina con sliders.

---

## 📱 Responsive / Mobile (v1.14)

App completamente usable en celular y tablet:

- **Topbar reorganizado**: badges en línea inferior con padding apropiado
- **Tabs scrolleables horizontalmente**: no se rompen ni encogen
- **Layout una columna**: en Publicaciones, el canvas va PRIMERO para verlo sin scroll
- **Canvas adaptable**: 100% del ancho, máx 540px, centrado
- **Inputs táctiles**: 16px mínimo de fuente (previene zoom auto en iOS), altura ≥38px
- **Sliders cómodos**: thumb 24px, fácil de agarrar con el dedo
- **Botones grandes**: altura mínima 38px (target táctil estándar)
- **Modales adaptados**: 95vw/90vh, no desbordan
- **Grillas adaptadas**: productos/colores/plantillas en 2 columnas (1 en celular angosto)
- **Tablas con scroll**: horizontales cuando no entran (proformas)
- **Touch action `manipulation`**: scroll vertical normal de la página, drag solo cuando se arrastra un elemento

---

## 🎨 Panel "Bloque inferior" (footer)

Control del bloque de info del negocio (dirección + envíos + CTA + handle):
- Tamaño: 5 niveles (Muy pequeño 60% → Muy grande 150%)
- Posición horizontal/vertical (sliders -30% a +30%)
- Mostrar/Ocultar bloque entero
- Bbox de selección coincide exactamente con el contenido real pintado
- Aplica a TODAS las plantillas

---

## 📝 Panel "Ajustar textos"

Control granular de 9 textos (ampliado en v1.12):
- Título, Subtítulo, Precio (actual), Precio anterior (Promo), Frase, Etiqueta de oferta (Promo), "¡Aprovecha hoy!" (Promo), Descripción corta, Categoría/Tag
- 5 niveles de tamaño (XS/S/M/L/XL)
- Offsets X/Y individuales
- Botón "Reset todos"
- Sincroniza automáticamente cuando se hace click en un texto del lienzo

---

## 🔧 Sistema universal "Mis Items"

Modal único reutilizado para gestionar guardados de las 3 secciones:
- Mis Catálogos / Mis Proformas / Mis Publicaciones
- Funciones: Guardar actual, Cargar por ID, Eliminar
- Límite: 50 items por tipo

---

## 🎨 Diseño y UI

- **Tipografías:** Cormorant Garamond (serif), Jost (sans-serif)
- **Paleta principal:** tonos bark (#3d2118), nude, cream, rose (#c47e6e), gold (#b8935a)
- **Logo embebido** como base64 (JPEG comprimido ~15KB), pre-cargado en caché para que aparezca en descargas
- **Foto principal pre-cargada** en caché (`pubPhotoImg`) para render sincrónico — esto garantiza que las fotos lifestyle queden ENCIMA de la principal
- Toasts para feedback, modales con overlay, confirmaciones para destructivos
- Auto-guardado con debounce y badge visual ("save dot")
- Modo "Vista limpia" para ver el resultado sin guías de edición

---

## 🔌 Estado actual del archivo

El archivo `index.html` está en la versión **v1.14.0**, ubicado en `/mnt/project/index.html` o donde lo tengas en tu proyecto. Tiene **6986 líneas** y pesa ~678 KB (incluye logo embebido).

### Configuración Supabase requerida

Para que la app funcione, necesitas tener configuradas estas tablas en Supabase con RLS (Row Level Security):

```sql
ALTER TABLE app_settings ENABLE ROW LEVEL SECURITY;
ALTER TABLE catalogs ENABLE ROW LEVEL SECURITY;
ALTER TABLE proformas ENABLE ROW LEVEL SECURITY;
ALTER TABLE post_drafts ENABLE ROW LEVEL SECURITY;

CREATE POLICY "users_own_data" ON app_settings
  FOR ALL USING (session_id = auth.uid()::text)
  WITH CHECK (session_id = auth.uid()::text);

CREATE POLICY "users_own_data" ON catalogs
  FOR ALL USING (session_id = auth.uid()::text)
  WITH CHECK (session_id = auth.uid()::text);

CREATE POLICY "users_own_data" ON proformas
  FOR ALL USING (session_id = auth.uid()::text)
  WITH CHECK (session_id = auth.uid()::text);

CREATE POLICY "users_own_data" ON post_drafts
  FOR ALL USING (session_id = auth.uid()::text)
  WITH CHECK (session_id = auth.uid()::text);
```

### Crear cuentas de usuario

Desde el panel de Supabase: **Authentication → Users → Add user → Create new user**
- Email + Password (mínimo 6 caracteres)
- Marcar "Auto Confirm User"

---

## 📜 Changelog completo

### v1.14.0 (actual) · 2026-05-03
- **FIX**: el bbox del Bloque inferior ahora coincide exacto con el contenido pintado (antes estaba estimado y no alineaba con los textos visibles)
- **FIX**: el logo del negocio ahora aparece en la imagen descargada (antes era async y no llegaba al PNG). Logo + foto principal pre-cacheados con `logoImgCache` y `pubPhotoImg`
- **MOBILE**: app rediseñada para celular y tablet con media queries `@media (max-width: 900px)` y `@media (max-width: 480px)`. Topbar reorganizado, tabs scrolleables, panel lateral en columna, canvas al 100% del ancho, inputs y botones con tamaño táctil apropiado, sliders con thumb más grande

### v1.13.0
- FIX: líneas guía de edición no aparecen al descargar imagen
- FIX: fotos lifestyle ahora se dibujan ENCIMA de la foto principal (antes la foto principal cargaba async y se pintaba después, tapándolas)
- FIX: imagen no se ve borrosa al abrir Publicaciones por primera vez (canvas arranca a 1080x1080 desde el inicio, no 540x540 escalado)
- `setPubPhotoSrc()` para precarga sincrónica de la foto principal

### v1.12.0
- TODOS los textos de las 5 plantillas ahora son editables con mouse/touch
- Cuadros de edición rediseñados: borde tenue en reposo, hover marcado con mini etiqueta, seleccionado azul con tirador
- Modo "Vista limpia" toggleable (botón ojo) para ocultar todos los bordes
- Solapamientos resueltos: el elemento más pequeño gana sobre el más grande (textos sobre fotos)
- Click en texto del lienzo → auto-sincroniza con el panel "Ajustar textos"
- Nuevas opciones de texto: precio anterior, "¡Aprovecha hoy!"

### v1.11.0
- Edición visual unificada: arrastrar logo, foto principal, fotos lifestyle (1-4), footer, textos y celdas de collage directamente en el lienzo
- Click → seleccionar (borde azul + tirador de redimensión + etiqueta)
- Drag para mover, tirador o rueda del mouse para redimensionar, doble click para resetear
- Fotos lifestyle ahora individualmente movibles y redimensionables
- Persistencia de transforms de lifestyle en publicaciones guardadas

### v1.10.0
- Modal centralizado "⚙️ Datos del negocio" en topbar
- Texto de envíos editable (antes hardcodeado)
- Campo "Sitio web / Email comercial" agregado

### v1.9.0
- Bloque inferior controlable (tamaño 5 niveles, offset X/Y, ocultar)

### v1.8.0
- Drag interactivo en collage (mouse/touch + zoom + dblclick)
- Panel "Ajustar textos" para 7 elementos (XS/S/M/L/XL + offsets)
- Campos "Descripción corta" y "Categoría / Tag"

### v1.7.0
- Sistema de autenticación con Supabase Auth (login bloqueante)
- Datos separados por usuario (filtrados por user.id)
- Solo login (no signup desde la app)

### v1.6.0
- Logo movible/redimensionable, foto movible/redimensionable
- Destino IG/FB/WA, orientación, fit, plantilla collage
- Footer no-overflow

---

## 📝 Para continuar en otro chat

Copia este resumen al inicio del nuevo chat junto con:

1. **El archivo `index.html` completo** del proyecto (versión v1.14.0)
2. **Los cambios específicos** que quieres hacer (qué pestaña, qué funcionalidad, qué corregir o agregar)

### Pendientes/ideas para futuras versiones

- Plantillas adicionales (Stories verticales 1080x1920 con su layout propio, Reels covers)
- Exportar publicación como video corto (animación sutil del precio o del badge)
- Sistema de plantillas guardadas por el usuario (no solo publicaciones individuales)
- Gestos de pellizco (pinch-to-zoom) para redimensionar en touch
- Soporte de orientación landscape en celular para edición más cómoda
- Atajos de teclado (Esc para deseleccionar, Delete para reset, flechas para mover)
