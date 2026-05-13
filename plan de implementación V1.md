# PROMPT
"Actúa como un Lead Fullstack Developer y Arquitecto de Soluciones Cloud. Mi objetivo es desarrollar una aplicación multiplataforma (Steam Clone) usando Flutter (Dart) y Firebase.

Restricciones Técnicas y Herramientas:

IDE: VS Code / Antigravity (Firebase Studio).

Base de Datos: Cloud Firestore (NoSQL), pero basada en un esquema relacional proporcionado (SQL de 15 tablas).

Estado: Provider para la gestión de estado.

Auth: Firebase Auth (Email/Password y Anonymous).

Entorno: Firebase Console Edición Estándar, Modo Prueba (sin reglas de seguridad complejas por ahora), sin Analytics.

Información de Base de Datos (Esquema de Referencia):
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

Tarea:
Genera un Plan de Implementación Paso a Paso en formato Markdown. No escribas código de lógica de negocio todavía, concéntrate en el 'roadmap' técnico. El plan debe incluir:

Fase de Preparación de Entorno: Configuración de Flutter, Firebase CLI y Antigravity.

Arquitectura de Datos (Mapeo SQL a NoSQL): Cómo transformar las 15 tablas SQL (Publishers, Games, Licenses, etc.) a Colecciones y Subcolecciones eficientes en Firestore, considerando que es una plataforma comercial.

Configuración de pubspec.yaml: Lista de dependencias esenciales (firebase_core, firebase_auth, cloud_firestore, provider, etc.) con una breve explicación de para qué sirve cada una en este contexto.

Estructura de Carpetas (Clean Architecture): Organización del proyecto (models, providers, services, ui/screens, widgets).

Flujo de Autenticación: Proceso de implementación desde el login anónimo (para ver el catálogo) hasta el registro completo (Email/Pass) para compras.

Estrategia de UI/UX: Listado de pantallas principales necesarias (Store, Library, Game Details, Wallet, Checkout) siguiendo la estética de Steam.

Hitos de Desarrollo (Sprints): División del trabajo en 4 fases (Setup & Auth, Catálogo & Datos, Sistema de Carrito/Ventas, Biblioteca de Usuario).

Por favor, mantén un tono profesional y técnico."

¿Por qué este prompt es efectivo?
Mapeo de Datos: Al pedirle que transforme SQL a NoSQL, obligas a la IA a pensar en cómo manejar los datos de "Precios Regionales" y "Licencias" en Firestore, que es el reto técnico más grande de tu proyecto.

Uso de Antigravity: Al mencionarlo específicamente, el plan incluirá pasos para visualizar y gestionar la base de datos de forma visual.

Diferenciación de Auth: Especifica el flujo "Anónimo vs. Registrado", que es crucial para una tienda donde el usuario primero mira y luego compra.

Enfoque en Estructura: Al prohibir código de lógica pero pedir pubspec.yaml y carpetas, aseguras que los cimientos del proyecto sean sólidos antes de escribir la primera línea de main.dart.

# RESPUESTA
# 📘 Plan de Implementación: Steam Clone (Flutter + Firebase)

## 1. 🛠️ Fase de Preparación de Entorno
| Componente | Acción Técnica |
|------------|----------------|
| **Flutter SDK** | Instalar `>= 3.24` (estable). Configurar PATH y verificar con `flutter doctor -v`. |
| **VS Code** | Instalar extensiones: `Dart`, `Flutter`, `Firebase`, `Error Lens`, `Pubspec Assist`. Activar `formatOnSave` y `flutter outline`. |
| **Firebase CLI** | `npm install -g firebase-tools` → `firebase login` → `firebase init firestore` (modo prueba). |
| **FlutterFire CLI** | Ejecutar `dart pub global activate flutterfire_cli` → `flutterfire configure` para generar `firebase_options.dart` automáticamente. |
| **Antigravity (Firebase Studio)** | Conectar al proyecto creado. Utilizar para: visualización jerárquica de Firestore, carga masiva de datos semilla (seed), validación de estructura de documentos y prueba rápida de reglas en modo `test` antes de la implementación. |
| **Entorno Firebase** | Crear proyecto en Firebase Console. Habilitar `Authentication` (Email/Pass + Anónimo) y `Firestore Database` (modo test: `allow read, write: if true;`). Desactivar `Analytics` y `Crashlytics` para reducir overhead inicial. |

---

## 2. 🗃️ Arquitectura de Datos (Mapeo SQL → NoSQL)
Firestore es orientado a documentos. La estrategia prioriza **lecturas optimizadas**, **denormalización controlada** y **agrupación por ámbito de usuario** para minimizar consultas y preparar el terreno para reglas de seguridad futuras.

| SQL Table | Firestore Equivalente | Estrategia de Mapeo |
|-----------|----------------------|---------------------|
| `publishers` | `publishers/{publisherId}` | Colección raíz. Documentos ligeros con datos estáticos. |
| `users` | `users/{uid}` | La clave es el UID de Firebase Auth. Se omiten `password_hash` y `user_sessions` (gestionados nativamente por Firebase). Se almacenan `displayName`, `countryCode`, `currencyCode`, `isActive`, `createdAt`. |
| `user_sessions` | ❌ No se mapea | Firebase Auth gestiona tokens, refresh tokens y sesiones de forma segura. Eliminar carga innecesaria en Firestore. |
| `user_wallets` | `users/{uid}/wallet/{walletId}` | Subcolección 1:1. Alternativamente, embeber `wallet: {balance, lastUpdated}` dentro del doc `users/{uid}` para acceso directo en perfil. |
| `games` | `games/{gameId}` | Colección raíz. Se denormalizan categorías y plataformas como arrays: `categorySlugs: ["action", "rpg"]`, `platforms: ["windows", "macos"]`. |
| `categories` | `categories/{categoryId}` | Colección raíz para gestión administrativa y filtros globales. |
| `game_categories` | ❌ Se embebe en `games` | Array de referencias o slugs dentro del doc `games`. Evita joins. |
| `platforms` | `platforms/{platformId}` | Colección raíz estática. |
| `game_platforms` | ❌ Se embebe en `games` | Map `platformReqs: {"windows": {"min": {...}, "rec": {...}}, "macos": {...}}`. Optimiza lectura de requisitos por juego. |
| `game_editions` | `games/{gameId}/editions/{editionId}` | Subcolección bajo el juego. 1:N natural. Permite escalar SKUs sin saturar el doc principal. |
| `prices` | ❌ Se embebe en `editions` | Map `regionalPrices: {"US": {amount: 59.99, discount: 10, validFrom, validUntil}, "AR": {...}}`. Permite resolver precios por región con **una sola lectura** al renderizar catálogo. |
| `payment_methods` | `users/{uid}/payment_methods/{pmId}` | Subcolección aislada. `is_default` boolean para UX rápida. |
| `orders` | `users/{uid}/orders/{orderId}` | Subcolección por usuario. Facilita auditoría y consultas `WHERE user_id == currentUid`. |
| `order_items` | ❌ Se embebe en `orders` | Array `items: [{editionId, unitPrice, discountApplied, title}]` dentro del doc `orders/{orderId}`. Evita queries adicionales post-compra. |
| `licenses` | `users/{uid}/library/{licenseId}` | Subcolección `library`. 1:1 con `order_items`. Se incluye `licenseKey`, `status`, `acquiredAt`, `editionId` para renderizado instantáneo de la biblioteca. |

**Índices y Optimización:** Firestore crea índices simples automáticamente. Para consultas compuestas (`prices` por país, `orders` por fecha), se generarán manualmente en la consola o mediante `firebase.json` cuando se salga del modo prueba.

---

## 3. 📦 Configuración de `pubspec.yaml`
```yaml
dependencies:
  flutter:
    sdk: flutter
  
  # 🔥 Firebase Ecosystem
  firebase_core: ^3.6.0           # Inicialización obligatoria del SDK
  firebase_auth: ^5.3.1           # Auth (Email/Pass + Anonymous + Linking)
  cloud_firestore: ^5.4.4         # Cliente NoSQL, streams, transacciones
  
  # 📊 State & Architecture
  provider: ^6.1.2                # Gestión de estado (ChangeNotifier/Consumer)
  equatable: ^2.0.5               # Value equality para modelos y estados
  freezed_annotation: ^2.4.2      # Inmutabilidad y copyWith (build runner)
  
  # 🧩 UI & Utilidades
  cached_network_image: ^3.3.1    # Cacheo eficiente de portadas/assets
  intl: ^0.19.0                   # Formateo de moneda, fechas y localización
  uuid: ^4.4.2                    # Generación de licenseKey y transactionRef
  go_router: ^14.2.0              # Navegación declarativa y deep linking
  
dev_dependencies:
  flutter_test:
    sdk: flutter
  build_runner: ^2.4.12           # Generación de código (freezed/json)
  freezed: ^2.5.7
  json_serializable: ^6.8.0
```
*Nota: Las versiones reflejan el ecosistema estable a 2026. Se recomienda usar `flutter pub get` y verificar compatibilidad con el SDK Flutter instalado.*

---

## 4. 📁 Estructura de Carpetas (Arquitectura Limpia Pragmática)
```
lib/
├── main.dart                     # Entry point, MultiProvider, Theme, Router
├── core/
│   ├── constants/                # Strings, API keys, enum states
│   ├── theme/                    # ThemeData (Steam Dark Palette), text styles
│   └── utils/                    # Formatters, validators, error helpers
├── data/
│   ├── models/                   # DTOs mapeados desde Firestore (Freezed/JSON)
│   ├── repositories/             # Interfaces de acceso a datos
│   └── services/                 # Implementaciones Firebase (FirestoreService, AuthService)
├── domain/                       # (Opcional) Entidades puras y casos de uso
├── presentation/
│   ├── providers/                # ChangeNotifiers (AuthProvider, CatalogProvider, CartProvider)
│   ├── screens/                  # Vistas de página (StoreScreen, LibraryScreen, CheckoutScreen)
│   ├── widgets/                  # Componentes reutilizables (GameCard, PriceTag, PlatformBadge)
│   └── router/                   # Rutas, guards de auth, transiciones
└── firebase_options.dart         # Generado automáticamente por flutterfire_cli
```
**Flujo de datos:** `UI (widgets/screens)` → `Providers` → `Repositories (Interfaces)` → `Services (Firestore/Auth)` → `Models`. Los Providers escuchan cambios y exponen estados a la UI vía `Consumer`/`context.watch`.

---

## 5. 🔐 Flujo de Autenticación (Anónimo → Registrado)
1. **Inicio Automático:** Al lanzar la app, `AuthService.signInAnonymously()` se ejecuta. El usuario puede navegar el catálogo, añadir al carrito y ver precios.
2. **Persistencia de Estado Transitorio:** El carrito y preferencias se almacenan temporalmente en memoria o en `SharedPreferences` vinculados al `anonymousUid`.
3. **Trigger de Registro:** Al intentar `Checkout` o acceder a `Wallet/Profile`, se presenta un modal de registro/login.
4. **Linking Seguro:** Se utiliza `currentUser.linkWithCredential(EmailAuthProvider.credential(email, password))`. Firebase fusiona la cuenta anónima y la nueva sin perder sesiones ni crear duplicados.
5. **Migración de Datos:** Post-linking, el `AuthProvider` escucha `authStateChanges()`. Si el `uid` cambia, se transfieren los datos temporales a la estructura persistente bajo el nuevo `uid` en Firestore (carrito → `users/{uid}/cart`, etc.).
6. **Gestión con Provider:** `AuthProvider` expone `UserState { loading, isAuthenticated, user, isAnonymous }`. La UI reacciona a `isAnonymous ? ShowStoreOnly : ShowFullFeatures`.

---

## 6. 🎨 Estrategia de UI/UX
**Paleta y Estética:** Tema oscuro inspirado en Steam (`#1b2838` fondo, `#2a475e` contenedores, `#66c0f4` acento, `#a4d007` éxito). Tipografía limpia (Inter/Roboto), bordes sutiles, efectos de hover en desktop y sombras de elevación.

| Pantalla | Propósito | Componentes Clave |
|----------|-----------|-------------------|
| **Store / Home** | Catálogo principal | Carrusel destacado, filtros por categoría, grid responsivo, skeleton loaders |
| **Game Details** | Ficha técnica y comercial | Trailer/imagen, badges de plataforma, selector de edición, tabla de requisitos, precio regional dinámico |
| **Library** | Propiedad y acceso | Lista de juegos adquiridos, filtros (All, Ready, Hidden), botón "Play/Download", estado de licencia |
| **Wallet / Balance** | Gestión financiera | Saldo actual, historial de transacciones, botón "Añadir fondos", métodos de pago guardados |
| **Checkout** | Finalización de compra | Resumen de carrito, selector de pago, confirmación de precio final, botón de compra segura |
| **Auth / Profile** | Identidad | Toggle Login/Register, formulario email/pass, gestión de país/moneda, cierre de sesión |

**Multiplataforma:** Uso de `LayoutBuilder` y `MediaQuery` para adaptar grids (1 col móvil, 4 col desktop). Navegación: `BottomNavigationBar` en móvil, `NavigationRail` en desktop/tablet.

---

## 7. 🚀 Hitos de Desarrollo (Sprints)

| Sprint | Duración Estimada | Foco Técnico | Entregables |
|--------|-------------------|--------------|-------------|
| **1. Setup & Auth** | Semanas 1-2 | Configuración Firebase, FlutterFire, estructura base, Auth flow (Anónimo ↔ Email/Pass), tema UI, router | App compilable, login funcional, navegación base, providers iniciales, reglas test mode |
| **2. Catálogo & Datos** | Semanas 3-4 | Mapeo NoSQL, servicios Firestore, modelos Freezed, carga paginada, precios regionales, UI Store & Game Details | Catálogo navegable, fichas técnicas, filtros por categoría/plataforma, formateo de moneda local |
| **3. Carrito, Wallet & Checkout** | Semanas 5-6 | Estado del carrito (Provider), transacciones simuladas, ordenes en Firestore, vinculación auth, pantalla checkout | Flujo de compra completo, generación de orders/items, actualización de wallet, receipt básico |
| **4. Biblioteca & Polish** | Semanas 7-8 | Licencias en Firestore, vista Library, optimización de queries, adaptación multiplataforma, testing manual | Biblioteca funcional, estados de licencia, UI responsive estable, documentación técnica y handoff |

---
✅ **Próximo paso recomendado:** Validar el mapeo NoSQL con datos semilla en Antigravity, generar los modelos con `freezed`, y comenzar el Sprint 1 configurando `firebase_options.dart` y `AuthProvider`. ¿Deseas que profundice en la estructura de algún modelo específico o en la configuración inicial del router antes de iniciar la codificación?
