# Resumen de Casos de Uso - Panel Administrativo

## Índice de Casos de Uso

Este documento contiene la documentación completa de los casos de uso del panel administrativo de Parrítico:

### 1. [Autenticación](./01_Autenticacion.md)
- Iniciar sesión (Login)
- Cerrar sesión (Logout)
- Control de acceso con guards

### 2. [Dashboard](./02_Dashboard.md)
- Ver estadísticas generales (Admin)
- Ver bienvenida personalizada (Organizer)
- Visualizar última actividad

### 3. [Gestión de Productos](./03_Gestion_Productos.md)
- Listar productos (activos/inactivos)
- Crear producto con variantes e imágenes
- Editar producto
- Activar/Desactivar producto
- Eliminar producto permanentemente

### 4. [Gestión de Categorías](./04_Gestion_Categorias.md)
- Listar categorías
- Crear categoría con imagen
- Editar categoría
- Activar/Desactivar categoría
- Eliminar categoría en cascada

### 5. [Gestión de Subcategorías](./05_Gestion_Subcategorias.md)
- Listar subcategorías
- Crear subcategoría asociada a categoría padre
- Editar subcategoría
- Activar/Desactivar subcategoría
- Eliminar subcategoría en cascada

### 6. [Gestión de Órdenes](./06_Gestion_Ordenes.md)
- Listar órdenes por estado
- Ver detalle completo de orden
- Filtrar por estados según rol
- Tracking de envíos

### 7. [Gestión de Contactos](./07_Gestion_Contactos.md)
- Listar mensajes de contacto
- Ver detalle de mensaje
- Marcar como revisado
- Eliminar contacto
- Notificaciones en tiempo real

### 8. [Gestión de Usuarios](./08_Gestion_Usuarios.md)
- Crear nuevo usuario (Solo Admin)
- Asignar roles (admin/organizer)

## Arquitectura del Sistema

```mermaid
graph TB
    subgraph "Frontend Admin"
        A[Login] --> B{Autenticado?}
        B -->|Sí| C[Dashboard]
        B -->|No| A
        
        C --> D{¿Rol?}
        
        D -->|Admin| E[Panel Completo]
        D -->|Organizer| F[Panel Limitado]
        
        E --> G[Productos]
        E --> H[Categorías]
        E --> I[Subcategorías]
        E --> J[Órdenes]
        E --> K[Contactos]
        E --> L[Usuarios]
        
        F --> J
        F --> K
        
        G --> M[Crear/Editar/Eliminar]
        H --> M
        I --> M
        J --> N[Ver Detalles]
        K --> N
        L --> O[Crear Usuario]
    end
    
    subgraph "Backend API"
        P[REST API]
        Q[WebSocket Server]
    end
    
    C --> P
    E --> P
    F --> P
    G --> P
    H --> P
    I --> P
    J --> P
    K --> Q
    K --> P
    L --> P
    
    P --> R[(Base de Datos)]
    Q --> R
```

## Roles y Permisos

```mermaid
graph TD
    A[Usuario] --> B{¿Rol?}
    
    B -->|Admin| C[Acceso Completo]
    B -->|Organizer| D[Acceso Limitado]
    
    C --> E[Dashboard con estadísticas]
    C --> F[Gestión de productos]
    C --> G[Gestión de categorías]
    C --> H[Gestión de subcategorías]
    C --> I[Gestión de órdenes]
    C --> J[Gestión de contactos]
    C --> K[Crear usuarios]
    
    D --> L[Dashboard de bienvenida]
    D --> I
    D --> J
    D --> M[Sin crear productos]
    D --> N[Sin crear categorías]
    D --> O[Sin crear usuarios]
```

## Tecnologías Principales

- **Frontend Framework**: React 19 con TypeScript
- **UI Library**: Ant Design 5
- **State Management**: Zustand
- **Data Fetching**: React Query (TanStack Query)
- **Routing**: React Router v7
- **Notifications**: Socket.IO Client
- **Alerts**: SweetAlert2
- **Build Tool**: Vite

## Flujo de Navegación Principal

```mermaid
flowchart TD
    Start([Inicio]) --> Login[Login]
    Login --> Auth{¿Credenciales válidas?}
    
    Auth -->|No| Login
    Auth -->|Sí| Decode{JWT Decode}
    
    Decode -->|admin| Dashboard1[Dashboard Admin]
    Decode -->|organizer| Dashboard2[Dashboard Organizer]
    
    Dashboard1 --> Products[Productos]
    Dashboard1 --> Categories[Categorías]
    Dashboard1 --> Subcategories[Subcategorías]
    Dashboard1 --> Orders[Órdenes]
    Dashboard1 --> Contacts[Contactos]
    Dashboard1 --> Users[Usuarios]
    
    Dashboard2 --> Orders
    Dashboard2 --> Contacts
    
    Products --> ProductForm[Formulario Producto]
    Categories --> CategoryForm[Formulario Categoría]
    Subcategories --> SubcategoryForm[Formulario Subcategoría]
    Orders --> OrderDetail[Detalle Orden]
    Contacts --> ContactDetail[Detalle Contacto]
    Users --> UserForm[Formulario Usuario]
    
    ProductForm --> Products
    CategoryForm --> Categories
    SubcategoryForm --> Subcategories
    OrderDetail --> Orders
    ContactDetail --> Contacts
    UserForm --> Dashboard1
    
    Products --> Logout[Cerrar Sesión]
    Categories --> Logout
    Subcategories --> Logout
    Orders --> Logout
    Contacts --> Logout
    
    Logout --> Login
```

## Integraciones

### Socket.IO - Notificaciones en Tiempo Real

```mermaid
sequenceDiagram
    participant Customer as Cliente Web
    participant Backend as Backend
    participant Socket as Socket.IO
    participant Admin as Panel Admin
    
    Customer->>Backend: Enviar mensaje de contacto
    Backend->>Socket: emit("new_contact")
    Socket-->>Admin: Notificación en tiempo real
    Admin-->>Admin: Actualizar contador pendientes
    Admin-->>Admin: Mostrar notification.info()
```

## Estados Principales

### Producto
- `isActive`: true/false
- Estados de visualización: Activo/Inactivo

### Categoría
- `isActive`: true/false
- Relación: 1:N con Subcategorías

### Subcategoría
- `isActive`: true/false
- Relación: N:1 con Categoría, 1:N con Productos

### Orden
- Estados: `pending`, `paid`, `failed`, `processing`, `shipped`, `delivered`, `cancelled`

### Contacto
- Estados: `pending`, `reviewed`

### Usuario
- Roles: `admin`, `organizer`

## Patrones de Diseño Utilizados

1. **Container/Presenter**: Separación lógica en componentes
2. **Custom Hooks**: Reutilización de lógica (useSlug, etc.)
3. **State Management**: Zustand para estado global
4. **Query Management**: React Query para cache y sincronización
5. **Route Guards**: Protección de rutas basada en autenticación
6. **Real-time Updates**: Socket.IO para notificaciones

