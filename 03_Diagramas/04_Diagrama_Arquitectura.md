# Diagrama de Arquitectura - Parritico Store

## Descripción
Este diagrama muestra la arquitectura completa del sistema Parritico Store, incluyendo frontend, backend, base de datos e integraciones externas.

## Diagrama

```mermaid
graph TB
    %% Clientes
    subgraph "Clientes"
        C1[👤 Cliente Web]
        C2[👨‍💼 Administrador]
    end
    
    %% Frontend Applications
    subgraph "Frontend Applications"
        FE1[🛒 E-commerce Frontend<br/>React + Vite<br/>Tailwind CSS + Ant Design]
        FE2[⚙️ Admin Panel<br/>React + Vite<br/>Ant Design + React Query]
    end
    
    %% Backend Services
    subgraph "Backend Services"
        API[🚀 Express.js API<br/>TypeScript<br/>Node.js]
        AUTH[🔐 JWT Authentication<br/>bcrypt]
        UPLOAD[📁 File Upload<br/>Multer + Formidable]
        RATE[⏱️ Rate Limiting<br/>express-rate-limit]
    end
    
    %% Database Layer
    subgraph "Database Layer"
        DB[(🗄️ MongoDB<br/>Mongoose ODM)]
        SESSION[💾 Session Storage<br/>MongoDB Sessions]
    end
    
    %% External Services
    subgraph "External Services"
        ONVO[💳 Onvo Payment<br/>API Integration]
        CLOUD[☁️ Cloudinary<br/>Image Management]
        SOCKET[🔌 Socket.IO<br/>Real-time Communication]
    end
    
    %% Infrastructure
    subgraph "Infrastructure"
        SERVER[🖥️ Node.js Server<br/>Port 4000]
        CORS[🌐 CORS Configuration]
        ENV[⚙️ Environment Variables<br/>.env]
    end
    
    %% Connections
    C1 --> FE1
    C2 --> FE2
    
    FE1 --> API
    FE2 --> API
    
    API --> AUTH
    API --> UPLOAD
    API --> RATE
    
    AUTH --> DB
    UPLOAD --> CLOUD
    API --> DB
    API --> SESSION
    
    API --> ONVO
    API --> SOCKET
    
    SERVER --> API
    SERVER --> CORS
    SERVER --> ENV
    
    %% Styling
    classDef client fill:#e8f5e8,stroke:#2e7d32,stroke-width:2px
    classDef frontend fill:#e3f2fd,stroke:#1976d2,stroke-width:2px
    classDef backend fill:#fff3e0,stroke:#f57c00,stroke-width:2px
    classDef database fill:#f3e5f5,stroke:#7b1fa2,stroke-width:2px
    classDef external fill:#fce4ec,stroke:#c2185b,stroke-width:2px
    classDef infrastructure fill:#f1f8e9,stroke:#558b2f,stroke-width:2px
    
    class C1,C2 client
    class FE1,FE2 frontend
    class API,AUTH,UPLOAD,RATE backend
    class DB,SESSION database
    class ONVO,CLOUD,SOCKET external
    class SERVER,CORS,ENV infrastructure
```

## Arquitectura Detallada

### 🎯 Capa de Presentación (Frontend)

#### E-commerce Frontend
- **Tecnología**: React 19 + Vite + TypeScript
- **UI Framework**: Tailwind CSS + Ant Design
- **Estado**: Zustand para gestión de estado
- **Routing**: React Router DOM
- **Características**:
  - Catálogo de productos
  - Carrito de compras persistente
  - Proceso de checkout
  - Integración con Onvo para pagos

#### Admin Panel
- **Tecnología**: React 19 + Vite + TypeScript
- **UI Framework**: Ant Design + Tailwind CSS
- **Estado**: Zustand + React Query
- **Autenticación**: JWT con jwt-decode
- **Características**:
  - Dashboard administrativo
  - Gestión de productos y categorías
  - Gestión de pedidos
  - Panel de contactos

### 🚀 Capa de Aplicación (Backend)

#### API REST (Express.js)
- **Framework**: Express.js con TypeScript
- **Puerto**: 4000
- **Características**:
  - Arquitectura MVC
  - Middleware de autenticación JWT
  - Validación con express-validator
  - Rate limiting
  - CORS configurado

#### Módulos Principales
- **AuthController**: Autenticación y autorización
- **ProductController**: CRUD de productos
- **OrderController**: Gestión de pedidos
- **PaymentController**: Integración con Onvo
- **CategoryController**: Gestión de categorías
- **ContactController**: Mensajes de contacto

### 🗄️ Capa de Datos

#### MongoDB
- **ODM**: Mongoose
- **Colecciones principales**:
  - Users (administradores)
  - Products (productos con variantes)
  - Categories/SubCategories
  - Orders (pedidos)
  - Contacts (mensajes)

#### Modelos de Datos
- **Product**: Soporte para variantes, imágenes múltiples
- **Order**: Estados de pedido, integración con Onvo
- **User**: Roles de administrador
- **Category**: Jerarquía de categorías

### 🔌 Servicios Externos

#### Onvo (Pagos)
- **Propósito**: Procesamiento de pagos
- **Integración**: API REST + Webhooks
- **Flujo**: Payment Intent → Checkout Session → Webhook
- **Monedas**: CRC (Colones costarricenses)

#### Cloudinary (Imágenes)
- **Propósito**: Gestión de imágenes
- **Características**: Upload, transformación, CDN
- **Integración**: SDK de Node.js
- **Organización**: Carpetas por tipo (products, variants, categories)

#### Socket.IO (Tiempo Real)
- **Propósito**: Notificaciones en tiempo real
- **Eventos**: Cambios de estado de pedidos
- **Configuración**: CORS para múltiples frontends

### 🛡️ Seguridad y Middleware

#### Autenticación
- **JWT**: Tokens con expiración
- **bcrypt**: Hash de contraseñas
- **Roles**: admin, organizer

#### Validación
- **express-validator**: Validación de datos
- **Mongoose**: Validaciones de esquema
- **Sanitización**: Limpieza de inputs

#### Rate Limiting
- **express-rate-limit**: Protección contra abuso
- **Configuración**: Límites por endpoint

### 📁 Estructura del Proyecto

```
parritico/
├── backend-parritico/          # API Backend
│   ├── src/
│   │   ├── controllers/       # Controladores
│   │   ├── models/           # Modelos de datos
│   │   ├── routes/           # Rutas de API
│   │   ├── middleware/       # Middleware personalizado
│   │   ├── config/           # Configuraciones
│   │   └── utils/            # Utilidades
├── ecommerce-parritico/       # Frontend E-commerce
│   ├── src/
│   │   ├── modules/          # Módulos de funcionalidad
│   │   ├── core/             # Servicios base
│   │   └── layouts/          # Componentes de layout
├── front-admin-parritico/     # Panel de Administración
│   ├── src/
│   │   ├── pages/            # Páginas del admin
│   │   └── core/              # Servicios base
└── docs/                      # Documentación
```

### 🔄 Flujo de Datos

1. **Cliente** interactúa con **Frontend**
2. **Frontend** hace requests a **API**
3. **API** valida y procesa datos
4. **API** interactúa con **MongoDB**
5. **API** integra con **servicios externos**
6. **Webhooks** notifican cambios de estado
7. **Socket.IO** actualiza frontends en tiempo real

### 🚀 Despliegue y Configuración

#### Variables de Entorno
- **Database**: MongoDB connection string
- **JWT**: Secret key para tokens
- **Onvo**: API keys y webhook secrets
- **Cloudinary**: Cloud name, API key, secret
- **CORS**: URLs permitidas para frontends

#### Puertos y URLs
- **Backend**: http://localhost:4000
- **E-commerce**: http://localhost:5173
- **Admin Panel**: http://localhost:5174
- **MongoDB**: mongodb://localhost:27017/parritico
