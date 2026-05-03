# 🌸 Flowers Handmade — Sistema Completo

> *Detalles que perduran*

Sistema web para **Flowers Handmade**, negocio de flores eternas hechas a mano en Loja, Ecuador. Compuesto por **dos aplicaciones independientes** que comparten el mismo backend Supabase y la misma cuenta de usuario.

**Versión actual:** v1.15.0 · Build 2026-05-03

---

## 📦 Las dos aplicaciones

### 1. `index.html` — App principal (catálogos, proformas, publicaciones)
SPA cliente-pesada para crear materiales comerciales: catálogos PDF, proformas profesionales y publicaciones para redes sociales.

### 2. `admin.html` — Panel administrativo (inventario, ventas, clientes) · **NUEVO**
Panel tipo app móvil para gestionar el negocio en el día a día: registrar ventas en segundos, controlar stock, identificar clientes frecuentes y ver utilidad real.

Ambas apps comparten:
- Mismo proyecto Supabase (`owfetilyxgvkrtdopskz.supabase.co`)
- Misma autenticación (storage key `flowers_handmade_session_v7`) → si inicias sesión en una, la otra te reconoce automáticamente
- Mismos datos de usuario (filtrados por `auth.uid()`)

---

## 🏗️ Arquitectura general

- **Frontend puro:** HTML + CSS + JavaScript vanilla (sin frameworks)
- **Backend:** Supabase (Auth + Postgres con RLS)
- **Persistencia:** Supabase como fuente de verdad + localStorage como respaldo offline
- **Autenticación:** Supabase Auth con login bloqueante (sin login no se puede usar)
- **Datos por usuario:** todo se filtra por `user.id` mediante Row Level Security
- **PDFs:** generados con jsPDF (sólo en `index.html`)
- **Imágenes/Posts:** generados en `<canvas>` HTML5 con edición visual interactiva
- **Responsive:** mobile-first, totalmente usable en celular

---

## 🗄️ Estructura de Supabase

### Tablas existentes (de `index.html`)
| Tabla | Propósito |
|---|---|
| `app_settings` | Productos del banco maestro + config del negocio (1 fila por usuario, datos en JSON) |
| `catalogs` | Catálogos PDF guardados (snapshots completos) |
| `proformas` | Proformas/cotizaciones guardadas |
| `post_drafts` | Publicaciones para redes sociales guardadas |

### Tablas nuevas (para `admin.html`)
| Tabla | Propósito |
|---|---|
| `products` | Inventario real (con stock y costo) |
| `customers` | Clientes con totales acumulados |
| `sales` | Historial de ventas con utilidad calculada |

### SQL para crear las tablas nuevas

Pega esto en el **SQL Editor** de Supabase y ejecútalo una sola vez:

```sql
-- ────────────────────────────────────────────────────────────
-- FLOWERS HANDMADE · ADMIN PANEL · ESQUEMA DE BASE DE DATOS
-- Crea: products, customers, sales (con RLS por usuario)
-- NO modifica las tablas existentes del catálogo público
-- ────────────────────────────────────────────────────────────

CREATE TABLE IF NOT EXISTS products (
  id          BIGSERIAL PRIMARY KEY,
  user_id     UUID NOT NULL,
  name        TEXT NOT NULL,
  price       NUMERIC(10,2) DEFAULT 0,
  cost        NUMERIC(10,2) DEFAULT 0,
  stock       INTEGER DEFAULT 0,
  tag         TEXT,
  photo       TEXT,
  legacy_id   BIGINT,
  created_at  TIMESTAMPTZ DEFAULT NOW()
);

CREATE TABLE IF NOT EXISTS customers (
  id                BIGSERIAL PRIMARY KEY,
  user_id           UUID NOT NULL,
  name              TEXT NOT NULL,
  phone             TEXT,
  total_purchases   NUMERIC(10,2) DEFAULT 0,
  last_purchase_at  TIMESTAMPTZ,
  created_at        TIMESTAMPTZ DEFAULT NOW()
);

CREATE TABLE IF NOT EXISTS sales (
  id              BIGSERIAL PRIMARY KEY,
  user_id         UUID NOT NULL,
  product_id      BIGINT REFERENCES products(id)  ON DELETE SET NULL,
  product_name    TEXT,
  customer_id     BIGINT REFERENCES customers(id) ON DELETE SET NULL,
  customer_name   TEXT,
  quantity        INTEGER NOT NULL,
  price           NUMERIC(10,2) NOT NULL,
  total           NUMERIC(10,2) NOT NULL,
  profit          NUMERIC(10,2) DEFAULT 0,
  payment_method  TEXT,
  created_at      TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX IF NOT EXISTS idx_products_user      ON products(user_id);
CREATE INDEX IF NOT EXISTS idx_products_user_tag  ON products(user_id, tag);
CREATE INDEX IF NOT EXISTS idx_customers_user     ON customers(user_id);
CREATE INDEX IF NOT EXISTS idx_sales_user_date    ON sales(user_id, created_at DESC);
CREATE INDEX IF NOT EXISTS idx_sales_product      ON sales(product_id);
CREATE INDEX IF NOT EXISTS idx_sales_customer     ON sales(customer_id);

ALTER TABLE products  ENABLE ROW LEVEL SECURITY;
ALTER TABLE customers ENABLE ROW LEVEL SECURITY;
ALTER TABLE sales     ENABLE ROW LEVEL SECURITY;

DROP POLICY IF EXISTS "users_own_products"  ON products;
DROP POLICY IF EXISTS "users_own_customers" ON customers;
DROP POLICY IF EXISTS "users_own_sales"     ON sales;

CREATE POLICY "users_own_products" ON products
  FOR ALL USING (user_id = auth.uid()) WITH CHECK (user_id = auth.uid());

CREATE POLICY "users_own_customers" ON customers
  FOR ALL USING (user_id = auth.uid()) WITH CHECK (user_id = auth.uid());

CREATE POLICY "users_own_sales" ON sales
  FOR ALL USING (user_id = auth.uid()) WITH CHECK (user_id = auth.uid());
```

### Políticas RLS para tablas existentes (referencia, ya configuradas)

```sql
ALTER TABLE app_settings ENABLE ROW LEVEL SECURITY;
ALTER TABLE catalogs     ENABLE ROW LEVEL SECURITY;
ALTER TABLE proformas    ENABLE ROW LEVEL SECURITY;
ALTER TABLE post_drafts  ENABLE ROW LEVEL SECURITY;

CREATE POLICY "users_own_data" ON app_settings
  FOR ALL USING (session_id = auth.uid()::text)
  WITH CHECK (session_id = auth.uid()::text);

-- (idéntico para catalogs, proformas, post_drafts)
```

### Diagrama de relaciones (admin)

```
┌─────────────────┐         ┌─────────────────┐
│    products     │         │    customers    │
│─────────────────│         │─────────────────│
│ id (PK)         │         │ id (PK)         │
│ user_id         │         │ user_id         │
│ name, tag       │         │ name, phone     │
│ price, cost     │         │ total_purchases │
│ stock           │         │ last_purchase_at│
│ photo, legacy_id│         └────────┬────────┘
└────────┬────────┘                  │
         │                           │
         │     ┌─────────────────┐   │
         │     │     sales       │   │
         │     │─────────────────│   │
         └─────┤ product_id (FK) │   │
               │ customer_id (FK)├───┘
               │ quantity, price │
               │ total, profit   │
               │ payment_method  │
               │ created_at      │
               └─────────────────┘
```

---

## 🔐 Autenticación

- Pantalla de login bloqueante full-screen al abrir cualquiera de las dos apps
- Solo login con email + password (sin signup, las cuentas las crea el admin desde Supabase)
- Las dos apps comparten sesión: si inicias sesión en `index.html` y abres `admin.html`, ya estás dentro
- Cada usuario ve solo SUS datos en todas las tablas

### Crear cuentas de usuario
Desde el panel de Supabase: **Authentication → Users → Add user → Create new user**
- Email + Password (mínimo 6 caracteres)
- Marcar "Auto Confirm User"

---

# 📘 PARTE 1: `index.html` — App principal

## ⚙️ Modal central "Datos del negocio"

Botón en la barra superior. Centraliza todos los datos del negocio en un solo lugar:

**Datos públicos:** WhatsApp, Redes sociales, Dirección, Slogan, Texto de envíos, Sitio web / Email comercial

**Datos fiscales (proformas):** Razón social, RUC/CI, Teléfono, Dirección, Email

Sincroniza con los inputs de cada pestaña. Persistencia automática en Supabase y localStorage.

## 📑 Las 4 pestañas (tabs)

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
- **5 plantillas:** Producto destacado, Carrusel, Frase, Promo/Oferta, Collage (2-10 fotos)
- **8 paletas de color:** Nude, Rosa, Crema, Sage, Azul, Negro, Dorado, Lila
- **Destinos:** Instagram, Facebook, WhatsApp (afecta caption y formato)
- **Edición visual completa:** todo se edita arrastrando directamente en el lienzo (logo, fotos, textos, footer, celdas de collage)
- **Solapamientos resueltos:** cuando un texto y una foto se cruzan, gana siempre el elemento más pequeño
- **Botón 👁️‍🗨️ Vista limpia** oculta todos los bordes para ver el resultado real

## 🎨 Panel "Bloque inferior" (footer)
Control del bloque de info del negocio (dirección + envíos + CTA + handle): tamaño 5 niveles, posición H/V, mostrar/ocultar, bbox que coincide exactamente con el contenido pintado.

## 📝 Panel "Ajustar textos"
Control granular de 9 textos: Título, Subtítulo, Precio, Precio anterior, Frase, Etiqueta de oferta, "¡Aprovecha hoy!", Descripción corta, Categoría/Tag. 5 niveles de tamaño + offsets X/Y individuales.

## 🔧 Sistema universal "Mis Items"
Modal único reutilizado para gestionar guardados: Mis Catálogos / Mis Proformas / Mis Publicaciones. Funciones: Guardar actual, Cargar por ID, Eliminar. Límite: 50 items por tipo.

---

# 📗 PARTE 2: `admin.html` — Panel administrativo · NUEVO

Panel tipo app móvil con tabbar inferior fija, FAB flotante para nueva venta, modales bottom-sheet con handle. Diseñado para uso diario rápido en celular.

## 🚀 Primera vez: migración inteligente

Al entrar por primera vez, si la tabla `products` está vacía pero detecta productos en `app_settings.data.products`, aparece un **banner dorado** con el conteo y un botón **"Importar productos al inventario"**. Mapea:

| Origen (`app_settings`) | Destino (`products`) |
|---|---|
| `name` | `name` |
| `priceAfterDiscount` o `price` | `price` |
| `tag` | `tag` |
| `photo` (base64) | `photo` |
| `id` | `legacy_id` (referencia) |
| — | `stock = 0` (lo completas tú) |
| — | `cost = 0` (lo completas tú) |

Después de importar, el banner desaparece y `app_settings` queda intacto (`index.html` sigue funcionando exactamente igual).

## 📊 Vistas

### Dashboard
- Tile destacado: ventas totales del periodo (hoy/semana/mes), número de ventas, ticket promedio
- Utilidad del periodo + % de margen
- Total de productos + alerta de stock bajo
- Total de clientes + cliente top
- Producto más vendido del periodo
- Atajos rápidos: nueva venta, ir a inventario, historial, exportar CSV
- Lista de las últimas 5 ventas

### Ventas
- Filtros: hoy / semana / mes / todo
- Resumen del periodo: total, utilidad, número de ventas
- Botón exportar CSV
- Lista completa con producto, cliente, método de pago, fecha y utilidad
- Eliminar venta individual (no restaura stock automáticamente)

### Inventario
- Buscador por nombre o categoría
- Chips de filtro por tag
- Lista con foto/icono, precio, costo, stock
- **Alerta visual** cuando stock < 3: badge rojo "Te quedan X" o "0 — agotado"
- Acciones por producto: ajustar stock, editar, eliminar
- Modal de stock con botones rápidos ±1 y ±5
- Modal de producto completo (nombre, categoría, stock, precio, costo, foto reducida automáticamente a 600px max)

### Clientes
- Buscador por nombre o teléfono
- Ranking automático por gasto total con badges 🥇🥈🥉 para los top 3
- Lista con teléfono, total gastado, última compra
- Detalle del cliente: stats + historial completo de sus compras
- Eliminar cliente (las ventas conservan el snapshot del nombre)

## 💰 Registro de venta (en segundos)

FAB flotante (＋) en cualquier vista. Al confirmar:

1. **Cliente:** si se ingresó nombre/teléfono, busca por teléfono → busca por nombre → si no existe lo crea
2. **Venta:** inserta en `sales` con snapshot del nombre del producto y del cliente
3. **Stock:** descuenta del producto automáticamente
4. **Cliente:** suma el total a `total_purchases` y actualiza `last_purchase_at`
5. **Utilidad:** calcula `(price - cost) * quantity` y la guarda
6. **Aviso:** si el stock quedó bajo el umbral (3), muestra alerta amarilla

## 🎨 Diseño

- Misma paleta que `index.html`: bark `#3d2118`, nude `#e8d5c4`, cream `#fdf8f3`, rose `#c47e6e`, gold `#b8935a`
- Tipografías Cormorant Garamond (serif, displays) + Jost (sans, body)
- Mobile-first con tabbar inferior fija (4 pestañas)
- Inputs táctiles 16px (sin zoom auto en iOS), targets ≥42px
- Modales tipo bottom-sheet con handle visible
- En desktop (≥720px) el tabbar pasa a la parte superior y el contenido se centra a 920px

## 📤 Exportar CSV

Botón en Dashboard y Ventas. Genera CSV con BOM UTF-8 (compatible con Excel/Numbers) con columnas: Fecha, Producto, Cantidad, Precio Unit, Total, Utilidad, Cliente, Teléfono, Método.

## 🔌 Modo offline / respaldo

Cada operación exitosa guarda una copia de productos, ventas y clientes en localStorage (clave `fh_admin_backup_v1`). Si Supabase falla al cargar, el panel muestra los datos del respaldo para que sigas trabajando.

---

## 🛠️ Despliegue / cómo usarlo

1. **Crear las tablas nuevas** en Supabase (SQL de arriba) — solo la primera vez
2. **Crear cuentas de usuario** desde Supabase Auth — solo cuando agregas a alguien al equipo
3. **Subir** `index.html` y `admin.html` a tu hosting (cualquier servidor estático funciona: Netlify, Vercel, GitHub Pages, etc.)
4. **Compartir las URLs** con quien corresponda:
   - `https://tu-dominio.com/` — para crear catálogos, proformas y publicaciones
   - `https://tu-dominio.com/admin.html` — para gestionar inventario y ventas

---

## 📜 Changelog

### v1.15.0 · 2026-05-03 — Admin panel
- **NUEVO:** `admin.html` — panel administrativo independiente con login, dashboard, inventario, ventas y clientes
- **NUEVO:** 3 tablas en Supabase: `products`, `customers`, `sales` con RLS por usuario
- **NUEVO:** migración automática desde `app_settings.data.products` con un click (sin duplicación)
- **NUEVO:** registro de ventas en segundos con descuento automático de stock y cálculo de utilidad
- **NUEVO:** alertas de stock bajo (umbral configurable, default 3 unidades)
- **NUEVO:** ranking de clientes por gasto total con badges oro/plata/bronce
- **NUEVO:** exportar ventas a CSV con BOM UTF-8 (compatible Excel)
- `index.html` no se tocó — sigue funcionando idéntico

### v1.14.0 · 2026-05-03
- FIX: el bbox del Bloque inferior coincide exacto con el contenido pintado
- FIX: el logo del negocio aparece en la imagen descargada (logo + foto principal pre-cacheados)
- MOBILE: app rediseñada para celular y tablet (`@media max-width: 900px` y `480px`)

### v1.13.0
- FIX: líneas guía de edición no aparecen al descargar imagen
- FIX: fotos lifestyle ahora se dibujan ENCIMA de la foto principal
- FIX: imagen no se ve borrosa al abrir Publicaciones por primera vez
- `setPubPhotoSrc()` para precarga sincrónica

### v1.12.0
- TODOS los textos de las 5 plantillas son editables con mouse/touch
- Cuadros de edición rediseñados: borde tenue, hover marcado, seleccionado azul con tirador
- Modo "Vista limpia" toggleable
- Solapamientos resueltos: el elemento más pequeño gana sobre el más grande
- Click en texto del lienzo → auto-sincroniza con panel "Ajustar textos"
- Nuevas opciones: precio anterior, "¡Aprovecha hoy!"

### v1.11.0
- Edición visual unificada: arrastrar logo, foto principal, fotos lifestyle (1-4), footer, textos, celdas de collage
- Click → seleccionar (borde azul + tirador + etiqueta)
- Drag para mover, tirador o rueda para redimensionar, doble click para resetear
- Persistencia de transforms de lifestyle en publicaciones guardadas

### v1.10.0
- Modal centralizado "⚙️ Datos del negocio" en topbar
- Texto de envíos editable
- Campo "Sitio web / Email comercial"

### v1.9.0
- Bloque inferior controlable (tamaño 5 niveles, offset X/Y, ocultar)

### v1.8.0
- Drag interactivo en collage
- Panel "Ajustar textos" para 7 elementos
- Campos "Descripción corta" y "Categoría / Tag"

### v1.7.0
- Sistema de autenticación con Supabase Auth (login bloqueante)
- Datos separados por usuario (filtrados por user.id)

### v1.6.0
- Logo movible/redimensionable, foto movible/redimensionable
- Destino IG/FB/WA, orientación, fit, plantilla collage

---

## 🚀 Pendientes / ideas futuras

### Para `index.html`
- Plantillas adicionales (Stories verticales 1080x1920, Reels covers)
- Exportar publicación como video corto
- Sistema de plantillas guardadas por el usuario
- Pinch-to-zoom para redimensionar en touch
- Soporte de orientación landscape en celular
- Atajos de teclado (Esc, Delete, flechas)

### Para `admin.html`
- Reportes mensuales con gráficas (Chart.js)
- Detección automática de productos a reponer (predicción simple por velocidad de venta)
- Compartir recibo de venta por WhatsApp (auto-generar mensaje)
- Múltiples productos por venta (carrito)
- Categorías de gasto (no solo ingresos): registrar costos operativos
- Modo "caja del día": resumen para cerrar la jornada
- Sincronización con productos de `app_settings` cuando se editan en `index.html`

---

## 📝 Para continuar en otro chat

Copia este README al inicio del nuevo chat junto con:
1. **El archivo afectado** (`index.html` o `admin.html`) — versión actual
2. **Los cambios específicos** que quieres hacer (qué archivo, qué pestaña, qué funcionalidad)

---

## ⚙️ Configuración técnica

```
Supabase URL:     https://owfetilyxgvkrtdopskz.supabase.co
Storage key:      flowers_handmade_session_v7  (compartido entre las dos apps)
localStorage keys:
  - fh_v6                 → estado actual de index.html
  - fh_saved_catalogs     → catálogos guardados
  - fh_saved_proformas    → proformas guardadas
  - fh_saved_publications → publicaciones guardadas
  - fh_admin_backup_v1    → respaldo del admin
```
