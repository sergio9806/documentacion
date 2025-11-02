# Diagrama de Caso de Uso: Gestión de Órdenes

## 1. Listar Órdenes (Según Rol)

```mermaid
sequenceDiagram
    actor Usuario as Admin/Organizer
    participant OrdersPage as OrdersPage.tsx
    participant AuthStore as auth.store
    participant ReactQuery as React Query
    participant OrderService as order.service.ts
    participant Backend as Backend API
    
    Usuario->>OrdersPage: Navegar a /admin/orders
    OrdersPage->>AuthStore: getRole()
    AuthStore-->>OrdersPage: role (admin | organizer)
    
    OrdersPage->>ReactQuery: useQuery(["orders"])
    ReactQuery->>OrderService: getAll()
    OrderService->>Backend: GET /api/orders
    
    Backend-->>OrderService: [Order]
    OrderService-->>ReactQuery: Orders[]
    ReactQuery-->>OrdersPage: Datos cargados
    
    OrdersPage->>OrdersPage: Agrupar por status
    Note over OrdersPage: Admin: pending, paid, failed, processing, shipped, delivered, cancelled
    Note over OrdersPage: Organizer: paid, processing, delivered, cancelled
    
    OrdersPage->>Usuario: Mostrar Tabs por estado con contadores
```

## 2. Ver Detalle de Orden

```mermaid
sequenceDiagram
    actor Usuario as Admin/Organizer
    participant OrderTable as OrderTable.tsx
    participant OrdersPage as OrdersPage.tsx
    participant Navigate as Router
    participant OrderDetailPage as OrderDetailPage.tsx
    participant ReactQuery as React Query
    participant OrderService as order.service.ts
    participant Backend as Backend API
    
    Usuario->>OrderTable: Click "Ver Detalle"
    OrderTable->>OrdersPage: navigate(`/admin/orders/${id}`)
    OrdersPage->>Navigate: Redireccionar
    Navigate->>OrderDetailPage: Cargar página
    
    OrderDetailPage->>ReactQuery: useQuery(["order", id])
    ReactQuery->>OrderService: getById(id)
    OrderService->>Backend: GET /api/orders/:id?populate=all
    
    Backend-->>OrderService: Order con datos completos
    Note over Backend,OrderService: Incluye: items[], customer, shipping, payment, tracking
    
    OrderService-->>ReactQuery: Order
    ReactQuery-->>OrderDetailPage: Datos del pedido
    
    OrderDetailPage->>Usuario: Mostrar detalles completos
    Note over OrderDetailPage,Usuario: Número factura, Cliente, Items, Totales, Estado, Tracking
```

## Vista de Órdenes por Rol

```mermaid
graph TD
    A[OrdersPage] --> B{¿Rol?}
    
    B -->|Admin| C[7 Tabs]
    C --> C1[Pendientes]
    C --> C2[Pagadas]
    C --> C3[Fallidas]
    C --> C4[Procesando]
    C --> C5[Enviadas]
    C --> C6[Entregadas]
    C --> C7[Canceladas]
    
    B -->|Organizer| D[4 Tabs]
    D --> D1[Pagadas]
    D --> D2[Procesando]
    D --> D3[Entregadas]
    D --> D4[Canceladas]
    
    C1 --> E[OrderTable]
    C2 --> E
    C3 --> E
    C4 --> E
    C5 --> E
    C6 --> E
    C7 --> E
    
    D1 --> E
    D2 --> E
    D3 --> E
    D4 --> E
```

## Estructura de Orden

```mermaid
graph TD
    A[Order] --> B[Información General]
    B --> B1[Invoice Number]
    B --> B2[Fecha]
    B --> B3[Estado]
    
    A --> C[Cliente]
    C --> C1[Nombre]
    C --> C2[Email]
    C --> C3[Teléfono]
    
    A --> D[Envio]
    D --> D1[Dirección]
    D --> D2[Ciudad]
    D --> D3[Código Postal]
    
    A --> E[Items]
    E --> E1[Producto]
    E --> E2[Cantidad]
    E --> E3[Precio Unitario]
    E --> E4[Variante]
    
    A --> F[Pago]
    F --> F1[Método]
    F --> F2[Total]
    F --> F3[Fecha Pago]
    
    A --> G[Tracking]
    G --> G1[Estado Envio]
    G --> G2[# Tracking]
```

## Tabla de Órdenes

```mermaid
graph LR
    A[OrderTable] --> B[Columna: Factura]
    A --> C[Columna: Cliente]
    A --> D[Columna: Total]
    A --> E[Columna: Estado]
    A --> F[Columna: Fecha]
    A --> G[Columna: Acciones]
    
    G --> H[Ver Detalle]
    
    E --> I{Status}
    I -->|pending| J[Tag Gold: PENDIENTE]
    I -->|paid| K[Tag Green: PAGADA]
    I -->|failed| L[Tag Red: FALLIDA]
    I -->|processing| M[Tag Blue: PROCESANDO]
    I -->|shipped| N[Tag Cyan: ENVIADA]
    I -->|delivered| O[Tag Purple: ENTREGADA]
    I -->|cancelled| P[Tag Magenta: CANCELADA]
```

## Flujo de Estados de Orden

```mermaid
stateDiagram-v2
    [*] --> pending: Crear orden
    pending --> paid: Pago exitoso
    pending --> failed: Pago fallido
    pending --> cancelled: Cancelar orden
    
    paid --> processing: Confirmar pago
    processing --> shipped: Enviar orden
    shipped --> delivered: Confirmar entrega
    
    failed --> [*]
    cancelled --> [*]
    delivered --> [*]
```

