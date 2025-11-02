# Resumen de Casos de Uso - E-commerce Parrítico

## Índice de Casos de Uso

Este documento contiene la documentación completa de los casos de uso del e-commerce público de Parrítico:

### 1. [Navegación y Vistas](./01_Navegacion_Vistas.md)
- Página de inicio (Home)
- Vista de productos
- Vista de producto individual
- Página sobre nosotros
- Página de contacto

### 2. [Gestión de Carrito](./02_Gestion_Carrito.md)
- Agregar productos al carrito
- Ver carrito
- Actualizar cantidad
- Eliminar productos
- Persistencia del carrito

### 3. [Proceso de Checkout](./03_Proceso_Checkout.md)
- Formulario de checkout
- Resumen de pedido
- Selección de método de pago

### 4. [Proceso de Pago](./04_Proceso_Pago.md)
- Pago con Onvo (Transferencia bancaria)
- Pago con Stripe (Tarjeta)
- Validación de montos mínimos
- Confirmación de pago exitoso

### 5. [Contacto](./05_Contacto.md)
- Envío de mensaje de contacto
- Validación de formulario
- Confirmación de envío

## Arquitectura del Sistema

```mermaid
graph TB
    subgraph "Frontend E-commerce"
        A[Home Page] --> B[Products Page]
        B --> C[Product Detail]
        
        C --> D[Cart Store]
        D --> E[Cart Page]
        E --> F[Checkout Page]
        
        F --> G{Payment Method}
        G -->|Onvo| H[Onvo Checkout]
        G -->|Stripe| I[Stripe Payment]
        
        H --> J[Success Page]
        I --> J
        
        A --> K[About Page]
        A --> L[Contact Page]
        L --> M[Send Message]
    end
    
    subgraph "Backend API"
        N[REST API]
        O[Payment Services]
    end
    
    B --> N
    C --> N
    D --> N
    F --> N
    H --> O
    I --> O
    M --> N
    
    N --> P[(Database)]
    O --> P
```

## Roles del Sistema

### 🌐 Cliente Público
Acceso a todas las funcionalidades del e-commerce:
- Navegar productos
- Ver detalles
- Agregar al carrito
- Realizar compra
- Contactar soporte

## Tecnologías Principales

- **Frontend Framework**: React 19 con TypeScript
- **UI Library**: Ant Design 5
- **State Management**: Zustand con persistencia
- **Routing**: React Router v7
- **Pagos**: Onvo & Stripe
- **Build Tool**: Vite
- **Styling**: TailwindCSS

## Flujo de Compra Principal

```mermaid
flowchart TD
    Start([Inicio]) --> Home[Home Page]
    
    Home --> Browse[Buscar Productos]
    Browse --> Filters[Aplicar Filtros]
    Filters --> Products[Vista Productos]
    
    Products --> Detail[Detalle Producto]
    Detail --> AddCart[Agregar al Carrito]
    AddCart --> Confirm{Confirmar}
    
    Confirm -->|Seguir comprando| Browse
    Confirm -->|Ver carrito| Cart[Carrito]
    
    Cart --> Update[Actualizar Cantidad]
    Update --> Cart
    
    Cart --> Checkout[Checkout]
    
    Checkout --> Form[Llenar Formulario]
    Form --> PayMethod{Metodo Pago}
    
    PayMethod -->|Onvo| OnvoPay[Onvo Payment]
    PayMethod -->|Stripe| StripePay[Stripe Payment]
    
    OnvoPay --> Success[Success Page]
    StripePay --> Success
    
    Success --> Home
```

## Estado del Carrito

```mermaid
stateDiagram-v2
    [*] --> Empty: Inicio
    Empty --> WithItems: Agregar producto
    WithItems --> WithItems: Actualizar cantidad
    WithItems --> WithItems: Eliminar producto
    WithItems --> Empty: Vaciar carrito
    WithItems --> Checkout: Ir a checkout
    Checkout --> Empty: Pago exitoso
```

## Métodos de Pago

```mermaid
graph TD
    A[Checkout] --> B{Elegir Metodo}
    
    B -->|Onvo| C[Transferencia Bancaria]
    C --> D[Crear Orden Onvo]
    D --> E[Obtener URL Checkout]
    E --> F[Redirigir a Onvo]
    F --> G[Cliente paga en Onvo]
    G --> H[Webhook confirma pago]
    H --> I[Success Page]
    
    B -->|Stripe| J[Tarjeta Credito/Debito]
    J --> K[Validar minimo 250]
    K --> L{Monto suficiente?}
    L -->|No| M[Mostrar alerta]
    M --> N[Volver a productos]
    L -->|Si| O[Crear Orden Stripe]
    O --> P[Formulario Stripe]
    P --> Q[Procesar tarjeta]
    Q --> R{Pago exitoso?}
    R -->|Si| I
    R -->|No| S[Mensaje de error]
```

## Estructura de Datos

### Producto
- Información básica (nombre, descripción, precio base)
- Categoría y subcategoría
- Imágenes
- Variantes (color, talla, stock por variante)
- Stock general

### Carrito
- Lista de items
- Producto + Variante (opcional) + Cantidad
- Persistencia en localStorage

### Orden
- Información del cliente
- Dirección de envío
- Items (productos y variantes)
- Método de pago seleccionado
- Total a pagar

## Integraciones Externas

### Onvo
- Plataforma de pagos bancarios
- Transferencia en línea
- Webhooks para confirmación

### Stripe
- Procesador de tarjetas
- Validación de montos
- Formulario seguro de pago

## Patrones de Diseño Utilizados

1. **State Management**: Zustand con persistencia
2. **Cart Persistence**: localStorage
3. **Payment Gateways**: Onvo & Stripe integrados
4. **Responsive Design**: TailwindCSS
5. **Component Reusability**: Layout compartido

