# PROMPT
Actúa como Lead Fullstack Developer y Arquitecto de Soluciones Cloud. Mi objetivo es que diseñes un Plan de Implementación Maestro para una aplicación multiplataforma (Steam Clone) usando Flutter y Firebase.

🛠️ STACK Y HERRAMIENTAS
Framework: Flutter (Dart).

Backend: Firebase (Standard Edition) en Modo Prueba.

Base de Datos: Cloud Firestore (siguiendo el esquema de 15 tablas SQL adjunto).

Estado: Provider.

IDE: VS Code / Antigravity (Firebase Studio).

Auth: Email/Password y Anonymous.

📊 ESQUEMA SQL DE REFERENCIA (CONTEXTO DE DATOS)
-- =============================================================
-- BASE DE DATOS: bdsteam
-- DESCRIPCIÓN: Plataforma de distribución digital de videojuegos
-- ENFOQUE: Comercial, Catálogo y Biblioteca.
-- =============================================================

-- 1. Gestión de Publicadores
CREATE TABLE publishers (
    publisher_id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    country_code CHAR(2),
    website_url VARCHAR(255),
    is_active BOOLEAN DEFAULT true,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- 2. Gestión de Usuarios (Core)
CREATE TABLE users (
    user_id SERIAL PRIMARY KEY,
    email VARCHAR(150) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    display_name VARCHAR(50) NOT NULL,
    country_code CHAR(2) NOT NULL,
    currency_code CHAR(3) DEFAULT 'USD',
    is_active BOOLEAN DEFAULT true,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- 3. Sesiones de Usuario
CREATE TABLE user_sessions (
    session_id SERIAL PRIMARY KEY,
    user_id INT NOT NULL REFERENCES users(user_id) ON DELETE CASCADE,
    token_hash VARCHAR(255) UNIQUE NOT NULL,
    device_info TEXT,
    ip_address VARCHAR(45),
    expires_at TIMESTAMP NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- 4. Monederos Virtuales (Comercial)
CREATE TABLE user_wallets (
    wallet_id SERIAL PRIMARY KEY,
    user_id INT UNIQUE NOT NULL REFERENCES users(user_id),
    current_balance DECIMAL(12, 2) DEFAULT 0.00,
    last_updated TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- 5. Catálogo de Videojuegos
CREATE TABLE games (
    game_id SERIAL PRIMARY KEY,
    publisher_id INT NOT NULL REFERENCES publishers(publisher_id),
    title VARCHAR(200) NOT NULL,
    description TEXT,
    release_date DATE,
    age_rating VARCHAR(10),
    is_active BOOLEAN DEFAULT true,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- 6. Categorías / Géneros
CREATE TABLE categories (
    category_id SERIAL PRIMARY KEY,
    name VARCHAR(50) UNIQUE NOT NULL,
    slug VARCHAR(50) UNIQUE NOT NULL
);

-- 7. Relación N:M Juegos-Categorías
CREATE TABLE game_categories (
    game_id INT NOT NULL REFERENCES games(game_id),
    category_id INT NOT NULL REFERENCES categories(category_id),
    PRIMARY KEY (game_id, category_id)
);

-- 8. Plataformas (Windows, MacOS, Linux, etc.)
CREATE TABLE platforms (
    platform_id SERIAL PRIMARY KEY,
    name VARCHAR(50) NOT NULL,
    os_family VARCHAR(50)
);

-- 9. Relación N:M Juegos-Plataformas (Compatibilidad técnica)
CREATE TABLE game_platforms (
    game_id INT NOT NULL REFERENCES games(game_id),
    platform_id INT NOT NULL REFERENCES platforms(platform_id),
    min_requirements JSONB,
    rec_requirements JSONB,
    PRIMARY KEY (game_id, platform_id)
);

-- 10. Ediciones de Juegos (SKUs)
CREATE TABLE game_editions (
    edition_id SERIAL PRIMARY KEY,
    game_id INT NOT NULL REFERENCES games(game_id),
    edition_name VARCHAR(100) NOT NULL, -- Ej: 'Standard', 'Deluxe', 'GOTY'
    description TEXT,
    is_bundle BOOLEAN DEFAULT false
);

-- 11. Precios y Ofertas (Regionalización)
CREATE TABLE prices (
    price_id SERIAL PRIMARY KEY,
    edition_id INT NOT NULL REFERENCES game_editions(edition_id),
    country_code CHAR(2) NOT NULL,
    amount DECIMAL(10, 2) NOT NULL,
    discount_pct INT DEFAULT 0,
    valid_from TIMESTAMP NOT NULL,
    valid_until TIMESTAMP NOT NULL,
    is_active BOOLEAN DEFAULT true
);

-- 12. Métodos de Pago
CREATE TABLE payment_methods (
    payment_method_id SERIAL PRIMARY KEY,
    user_id INT NOT NULL REFERENCES users(user_id),
    provider VARCHAR(50) NOT NULL, -- 'Stripe', 'PayPal', 'Wallet'
    provider_token VARCHAR(255),
    last_four CHAR(4),
    is_default BOOLEAN DEFAULT false,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- 13. Órdenes de Compra (Cabecera)
CREATE TABLE orders (
    order_id SERIAL PRIMARY KEY,
    user_id INT NOT NULL REFERENCES users(user_id),
    payment_method_id INT NOT NULL REFERENCES payment_methods(payment_method_id),
    total_amount DECIMAL(10, 2) NOT NULL,
    currency_code CHAR(3) NOT NULL,
    status VARCHAR(20) DEFAULT 'completed', -- 'pending', 'completed', 'refunded'
    transaction_ref VARCHAR(100) UNIQUE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- 14. Detalle de la Orden (Items)
CREATE TABLE order_items (
    order_item_id SERIAL PRIMARY KEY,
    order_id INT NOT NULL REFERENCES orders(order_id),
    edition_id INT NOT NULL REFERENCES game_editions(edition_id),
    unit_price DECIMAL(10, 2) NOT NULL,
    discount_applied DECIMAL(10, 2) DEFAULT 0.00
);

-- 15. Licencias y Biblioteca (Propiedad del Usuario)
CREATE TABLE licenses (
    license_id SERIAL PRIMARY KEY,
    user_id INT NOT NULL REFERENCES users(user_id),
    edition_id INT NOT NULL REFERENCES game_editions(edition_id),
    order_item_id INT NOT NULL REFERENCES order_items(order_item_id),
    license_key VARCHAR(100) UNIQUE NOT NULL,
    status VARCHAR(20) DEFAULT 'active', -- 'active', 'revoked'
    acquired_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- =============================================================
-- ÍNDICES PARA OPTIMIZACIÓN
-- =============================================================

CREATE INDEX idx_games_title ON games(title);
CREATE INDEX idx_prices_lookup ON prices(edition_id, country_code, is_active);
CREATE INDEX idx_licenses_user ON licenses(user_id);
CREATE INDEX idx_orders_user_date ON orders(user_id, created_at DESC);
CREATE INDEX idx_user_sessions_token ON user_sessions(token_hash);

📋 TAREAS REQUERIDAS EN EL PLAN:
1. Estrategia de UX (Experiencia de Usuario):

Define el "User Journey": desde el acceso anónimo para explorar el catálogo hasta el proceso de registro forzoso al intentar comprar.

Describe cómo se manejará la persistencia de la sesión y la transición de cuenta anónima a cuenta real sin perder los datos del carrito o preferencias.

2. Diseño de UI (Interfaz de Usuario):

Enumera los componentes atómicos necesarios para replicar la estética de Steam (Dark theme, Game Cards, botones de acción, etc.).

Listado detallado de pantallas principales: Store (Catálogo), Game Details, User Library, Wallet Management y Checkout.

3. Arquitectura de Base de Datos (Mapeo SQL a NoSQL):

Explica detalladamente cómo transformar las 15 tablas relacionales a una estructura de Colecciones y Subcolecciones en Firestore.

Define qué datos se deben "desnormalizar" (duplicar) para optimizar las lecturas y minimizar costos en Firebase.

Explica cómo se gestionarán las relaciones N:M (categorías y plataformas) en un entorno NoSQL.

4. Infraestructura y Dependencias:

Describe el proceso de vinculación entre VS Code y Antigravity para la gestión visual de datos.

Enumera las librerías necesarias para el proyecto (Auth, Firestore, Provider, etc.) explicando la responsabilidad técnica de cada una dentro del ecosistema de la app.

5. Estructura de Proyecto (Clean Architecture):

Define el árbol de directorios organizando el código en capas de Datos, Dominio y Presentación.

6. Roadmap de Implementación (4 Fases):

Fase 1 (Cimientos): Setup de Firebase, Auth y navegación base.

Fase 2 (Catálogo): Implementación de la visualización de juegos, precios regionales y filtros.

Fase 3 (Comercial): Lógica del Wallet, transacciones y validación de compras.

Fase 4 (Biblioteca): Generación de licencias y acceso a juegos adquiridos.

Mantén un tono técnico, directo y profesional. Enfócate en la viabilidad técnica y la escalabilidad del modelo de datos.

Por qué este prompt corregido es el mejor para ti:
Enfoque en la Desnormalización: Al pasar de SQL a Firestore, el mayor error es intentar usar Firestore como si fuera SQL. Este prompt obliga a la IA a explicarte cómo duplicar datos estratégicamente para que la app sea rápida.

Antigravity como Eje: Al mencionarlo como herramienta de gestión, el plan considerará cómo vas a monitorear las transacciones de los "Monederos" y "Licencias" visualmente.

Cero Ruido de Código: Al prohibir el código, obtendrás una explicación mucho más profunda sobre la lógica de negocio (por ejemplo, cómo validar que un usuario no compre dos veces el mismo juego).

UX de Compra: Se centra en el flujo de dinero (Wallet/Checkout), que es el corazón de tu esquema de base de datos.

# RESPUESTA
# 📐 MASTER IMPLEMENTATION PLAN: STEAM CLONE (Flutter + Firebase)
**Rol:** Lead Fullstack Developer & Cloud Solutions Architect  
**Enfoque:** Escalabilidad, optimización de costos Firestore, integridad transaccional, UX sin fricción.

---

## 1. 🧭 ESTRATEGIA DE UX (USER JOURNEY)

### Flujo Principal
1. **Acceso Anónimo:** Al abrir la app, se autentica automáticamente con `Firebase Anonymous Auth`. No se solicita registro. El usuario recibe un `UID` persistente que actúa como llave primaria local.
2. **Exploración y Carrito:** Navegación libre por el catálogo, aplicación de filtros, consulta de precios regionales y adición de juegos al carrito. Los datos se almacenan temporalmente en caché local vinculada al `UID` anónimo.
3. **Trigger de Conversión:** Al presionar "Comprar" o acceder a "Wallet", se activa la vista de registro/login. Se presenta un banner explícito: *"Tu carrito y preferencias se vincularán a tu cuenta permanente"*.
4. **Vinculación (Account Linking):** Se ejecuta `linkWithCredential` combinando la cuenta anónima con Email/Password. Firebase mantiene el mismo `UID`, garantizando que las preferencias, carrito y estado de la sesión no se pierdan.
5. **Checkout y Biblioteca:** Tras la validación del pago, se genera la licencia y se actualiza la biblioteca del usuario. La sesión persiste mediante refresh tokens nativos de Firebase Auth.

### Gestión de Sesión y Persistencia
- **Sesiones (`user_sessions` en SQL):** Se elimina la tabla relacional. Firebase Auth gestiona automáticamente tokens de acceso, refresh tokens, expiración y revocación. Solo se almacenan metadatos mínimos en Firestore si es necesario para auditoría.
- **Persistencia de Carrito/Preferencias:** Se utiliza `hive` o `shared_preferences` durante la fase anónima. Tras el `link`, un proceso de migración sincroniza el estado local con los documentos de Firestore bajo el `UID` unificado, limpiando la caché local.
- **Recuperación de Sesión:** El estado de autenticación se expone vía `StreamProvider`. Al reiniciar la app, se verifica el token local; si es válido, se restaura el contexto de usuario y se sincronizan datos críticos (saldo, licencias).

---

## 2. 🎨 DISEÑO DE UI (COMPONENTES ATÓMICOS Y PANTALLAS)

### Sistema de Diseño (Atomic Architecture)
- **Átomos:** `IconPlatform`, `BadgeAgeRating`, `SkeletonBlock`, `DividerThemed`, `TypographyScale`.
- **Moléculas:** `PriceTag` (manejo de descuento, moneda regional, formato), `GameCard` (miniatura, título, badges, precio), `PlatformRow` (iconos compatibles), `FilterChipGroup`.
- **Organismos:** `FeaturedCarousel`, `CatalogGrid`, `CartSummaryCard`, `PaymentMethodSelector`, `LicenseStatusIndicator`.

### Pantallas Principales
| Pantalla | Responsabilidad UX/UI | Componentes Clave |
|----------|----------------------|-------------------|
| **Store (Catálogo)** | Descubrimiento, navegación por géneros, ordenamiento por precio/popularidad. | `FeaturedCarousel`, `FilterBar`, `InfiniteScrollGrid`, `SearchOverlay`. |
| **Game Details** | Conversión. Información técnica, multimedia, selector de ediciones, precio regional dinámico. | `MediaGallery`, `EditionSelector`, `SystemRequirementsPanel`, `ActionButtonsGroup`. |
| **User Library** | Acceso post-compra. Estado de posesión, gestión de licencias, filtros por plataforma. | `LibraryGrid`, `StatusBadge`, `EmptyStateLibrary`, `QuickFilterTabs`. |
| **Wallet Management** | Gestión de fondos virtuales (modo prueba), historial de movimientos, configuración regional. | `BalanceHeader`, `TopUpMockPanel`, `TransactionHistoryList`, `CurrencySelector`. |
| **Checkout** | Validación comercial. Resumen, método de pago, confirmación de términos, recibo. | `OrderSummary`, `PaymentFlowStepper`, `ReceiptModal`, `SecurityDisclaimer`. |

---

## 3. 🗄️ ARQUITECTURA DE BASE DE DATOS (SQL → FIRESTORE)

### Principio Rector: Lecturas Optimizadas, Escrituras Controladas
Firestore no soporta `JOIN`. Se aplica **desnormalización estratégica** para evitar lecturas múltiples por documento. Los datos que cambian poco (títulos, nombres de categorías, requisitos) se embeben. Los datos transaccionales se aíslan en colecciones independientes.

### Mapeo Conceptual (Colecciones & Subcolecciones)
| Entidad SQL | Estructura Firestore | Justificación Arquitectónica |
|-------------|----------------------|------------------------------|
| `users` | `users/{uid}` (doc) | Perfil, `country_code`, `currency_code`, `wallet_balance`. |
| `user_sessions` | ❌ Eliminada | Gestionada nativamente por Firebase Auth tokens. |
| `publishers` | `publishers/{id}` (doc) | Referencia cruzada. `publisher_name` se desnormaliza en `games`. |
| `games` | `games/{id}` (doc) | `title`, `description`, `publisher_name`, `age_rating`, `is_active`, `category_slugs[]`, `platform_slugs[]`, `base_price_map{country:amount}`. |
| `categories` / `game_categories` | `categories/{slug}` + array en `games` | Relación N:M resuelta con `where('categories', arrayContains: 'action')`. |
| `platforms` / `game_platforms` | `platforms/{id}` + array en `games` | Embebido `platform_slugs[]` + `requirements` (map nested). |
| `game_editions` + `prices` | `games/{id}/editions/{id}` (subcol) o array embebido | Para catálogo, se embebe `edition_name` y `current_price` en el doc padre. Los precios históricos/válidos se mantienen en subcolección solo si es necesario auditoría. |
| `user_wallets` | Campo `wallet_balance` en `users/{uid}` | Lectura O(1). Actualizaciones vía transacciones atómicas. |
| `payment_methods` | `users/{uid}/payment_methods/{id}` (subcol) | Aislamiento por usuario. No requiere índices globales. |
| `orders` + `order_items` | `orders/{id}` (doc) + `items[]` (array de objetos embebidos) | Desnormalización total de items al momento de compra. Evita que cambios futuros de precio rompan el historial. |
| `licenses` | `users/{uid}/licenses/{id}` (subcol) | `edition_id`, `license_key`, `status`, `acquired_at`, `order_ref`. Acceso directo por usuario. |

### Estrategia de N:M y Consultas
- **Categorías/Plataformas:** Se almacenan como arrays de strings (`['rpg', 'action']`) en el documento `game`. Las consultas usan `arrayContains` y `arrayContainsAny` (máx. 30 valores).
- **Índices Compuestos:** Se configuran en `firestore.indexes.json`:
  - `games`: `is_active`, `category_slugs`, `base_price_map.*` (para ordenamiento regional).
  - `orders`: `user_id`, `created_at` (descendente).
  - `licenses`: `user_id`.
- **Seguridad:** Reglas estrictas para impedir lectura cruzada de wallets, modificación de precios y escritura directa en `licenses`. Solo Cloud Functions pueden emitir licencias y debitar fondos.

---

## 4. ⚙️ INFRAESTRUCTURA Y DEPENDENCIAS

### Integración VS Code + Antigravity (Firebase Studio)
- **Flujo de Desarrollo Local:** 
  1. Inicializar Emuladores (`firebase emulators:start`) desde terminal VS Code.
  2. Configurar Flutter para apuntar a `localhost` (Auth, Firestore, Functions).
  3. Utilizar Antigravity para inspección visual en tiempo real de documentos, validación de estructura de arrays/maps, y simulación de reglas de seguridad sin tocar la consola web.
  4. Exportar snapshots de datos de prueba desde Antigravity para CI/CD local.
- **Ventaja:** Detección temprana de desnormalizaciones incorrectas, validación de índices y debugging de transacciones sin costos reales.

### Stack de Librerías (Responsabilidades Técnicas)
| Paquete | Responsabilidad en el Ecosistema |
|---------|----------------------------------|
| `firebase_core` + `firebase_auth` | Inicialización, gestión de ciclo de vida de sesión, flujo anónimo→link, manejo de errores de auth. |
| `cloud_firestore` | Capa de datos. Consultas paginadas, listeners en tiempo real, transacciones atómicas, offline persistence. |
| `provider` | Gestión de estado UI. `ChangeNotifier` para AuthState, CartState, WalletState, CatalogState. Separa lógica de presentación de widgets. |
| `go_router` | Navegación declarativa. Guards de autenticación, deep linking a juegos/ediciones, manejo de rutas protegidas (Checkout/Library). |
| `intl` | Formateo de monedas (USD/EUR/etc), fechas, y números según `country_code` del usuario. |
| `crypto` + `uuid` | Generación segura de `license_key` (SHA-256 + salt) y UUIDs v4 para ordenes y sesiones internas. |
| `firebase_functions` (recomendado) | Validación server-side de precios, atomicidad de compra, generación de licencias, prevención de doble-gasto. |

---

## 5. 📁 ESTRUCTURA DE PROYECTO (CLEAN ARCHITECTURE)

```
lib/
├── core/                 # Constantes, tema, errores, utilidades, rutas, config de Firebase
├── data/                 # CAPA DE DATOS
│   ├── datasources/      # Firestore clients, Auth wrappers, Emulator connectors
│   ├── models/           # DTOs (fromJson/toJson), mapeo exacto a docs Firestore
│   └── repositories/     # Implementación concreta de interfaces de dominio
├── domain/               # CAPA DE NEGOCIO (0 dependencias de Flutter/Firebase)
│   ├── entities/         # Objetos puros: User, Game, Order, License, CartItem
│   ├── repositories/     # Interfaces abstractas (contracts)
│   └── usecases/         # Casos de uso: GetCatalog, LinkAccount, ExecutePurchase, FetchLibrary
└── presentation/         # CAPA DE UI
    ├── providers/        # StateNotifiers/ChangeNotifiers que orquestan UseCases
    ├── pages/            # Pantallas completas (Store, Details, Library, Wallet, Checkout)
    ├── widgets/          # Componentes reutilizables (atómicos y organismos)
    └── state/            # Enums de carga, error handlers, validadores de formulario
```
**Flujo de Datos:** UI → Provider → UseCase → Domain Entity ← Repository ← DataSource ← Firestore. Las reglas de inyección de dependencias se resuelven vía `Provider` en la raíz del árbol de widgets.

---

## 6. 🗺️ ROADMAP DE IMPLEMENTACIÓN (4 FASES)

### 🔹 Fase 1: Cimientos (Semana 1-2)
- Configuración de proyecto Flutter, integración Firebase CLI, setup de emuladores locales.
- Implementación de Auth: flujo anónimo, registro Email/Password, `linkWithCredential`.
- Estructura Clean Architecture, routing con guards, tema Dark base.
- Providers base: `AuthProvider`, `ThemeNotifier`. Validación de conexión a emuladores.

### 🔹 Fase 2: Catálogo (Semana 3-4)
- Despliegue de esquema Firestore en emuladores. Seed de datos de prueba vía Antigravity.
- Repositorio de catálogo: paginación (`startAfterDocument`), resolución de precios por región.
- Filtros funcionales: `arrayContains` para categorías/plataformas, ordenamiento por precio.
- UI: `StoreScreen`, `GameDetailsScreen`, sistema de carga progresiva y cache local.

### 🔹 Fase 3: Comercial (Semana 5-6)
- Lógica de Wallet (modo prueba): depósito simulado, validación de saldo.
- Gestión de carrito: persistencia local → sincronización post-auth.
- Checkout atómico: uso de Cloud Functions o Firestore Transactions para:
  1. Validar precio actual vs precio en carrito.
  2. Verificar saldo/método.
  3. Debitar, crear documento `orders`, marcar items como comprados.
- UI: `CheckoutScreen`, `WalletScreen`, historial de transacciones, recibos.

### 🔹 Fase 4: Biblioteca y Cierre (Semana 7-8)
- Generación de `licenses` post-compra exitosa (subcolección `users/{uid}/licenses`).
- `UserLibraryScreen`: consulta optimizada, estados (active/revoked), filtros.
- Optimización: índices compuestos, reglas de seguridad (`match` strict, `allow read/write if request.auth.uid == resource.data.user_id`), offline cache tuning.
- QA end-to-end: pruebas de concurrencia (doble compra), pérdida de sesión, migración de carrito, validación de precios regionales. Preparación para despliegue a producción (modo prueba → modo estable).

---

## 📌 NOTAS ARQUITECTÓNICAS CLAVE
1. **Cero Join, Máxima Velocidad:** La desnormalización de `publisher_name`, `category_slugs` y `base_price` en el documento `game` reduce lecturas de ~5 docs a 1 por item del catálogo. El costo de escritura aumenta marginalmente, pero el costo de lectura (la operación más frecuente) se minimiza drásticamente.
2. **Integridad Transaccional:** Nunca confiar en validación cliente para precios o saldo. El checkout debe ejecutarse en una transacción Firestore o Cloud Function para garantizar atomicidad y evitar condiciones de carrera.
3. **Escalabilidad del Modelo:** Las subcolecciones (`users/{uid}/licenses`, `users/{uid}/payment_methods`) garantizan particionamiento natural por usuario, optimizando índices y evitando documentos >1MB. Las colecciones raíz (`games`, `orders`, `publishers`) permanecen planas para queries globales.

Este plan garantiza un ciclo de desarrollo predecible, costos Firestore controlados, y una base escalable para evolucionar a funciones avanzadas (logros, comunidad, multijugador) sin reestructurar la capa de datos.
