# Diagrama de Caso de Uso: Proceso de Pago

## 1. Pago con Onvo

```mermaid
sequenceDiagram
    actor Cliente as Cliente
    participant CheckoutPage as CheckoutPage.tsx
    participant Backend as Backend API
    participant Onvo as Onvo Platform
    
    CheckoutPage->>Backend: POST /api/orders (Onvo)
    Backend->>Backend: Crear orden pending
    Backend->>Onvo: Generar paymentIntent
    Onvo-->>Backend: URL checkout
    Backend-->>CheckoutPage: paymentIntent.url
    
    CheckoutPage->>CheckoutPage: clearCart()
    CheckoutPage->>Onvo: window.location.href = url
    Onvo-->>Cliente: Formulario de pago bancario
    
    Cliente->>Onvo: Completar pago
    Onvo->>Onvo: Procesar transferencia
    Onvo->>Backend: Webhook confirmacion
    Backend->>Backend: Actualizar orden a paid
    Onvo-->>Cliente: Redirect a success page
```

## 2. Pago con Stripe - Cargar Página

```mermaid
sequenceDiagram
    actor Cliente as Cliente
    participant CheckoutPage as CheckoutPage.tsx
    participant CheckoutPayment as CheckoutPaymentPage.tsx
    participant OrderService as order.service
    participant Backend as Backend API
    
    CheckoutPage->>CheckoutPayment: Navigate /checkout/payment?orderId=X
    CheckoutPayment->>CheckoutPayment: Cargar orden
    
    CheckoutPayment->>OrderService: getById(orderId)
    OrderService->>Backend: GET /api/orders/:id
    
    Backend-->>OrderService: Order
    OrderService-->>CheckoutPayment: Order data
    
    CheckoutPayment->>CheckoutPayment: Validar monto minimo
    alt Monto menor a 250
        CheckoutPayment-->>Cliente: Mostrar alerta minimo
    end
    
    CheckoutPayment-->>Cliente: Mostrar formulario Stripe
```

## 3. Procesar Pago con Stripe

```mermaid
sequenceDiagram
    actor Cliente as Cliente
    participant StripeForm as StripePaymentForm.tsx
    participant StripeSDK as Stripe JS
    participant Backend as Backend API
    participant CartStore as cart.store
    
    Cliente->>StripeForm: Llenar tarjeta
    StripeForm->>StripeForm: setCardComplete(true)
    
    Cliente->>StripeForm: Click pagar
    StripeForm->>Backend: POST /stripe/create-payment-intent {orderId}
    Backend->>StripeSDK: Crear paymentIntent
    StripeSDK-->>Backend: clientSecret
    Backend-->>StripeForm: clientSecret
    
    StripeForm->>StripeSDK: confirmCardPayment(clientSecret, card)
    StripeSDK->>StripeSDK: Procesar tarjeta
    
    alt Pago exitoso
        StripeSDK-->>StripeForm: paymentIntent {status: succeeded}
        StripeForm->>StripeForm: message.success
        
        alt Test mode
            StripeForm->>Backend: PUT /orders/:id/status
            Backend->>Backend: Actualizar a paid y stock
        end
        
        StripeForm->>CartStore: clearCart()
        StripeForm->>StripeForm: navigate(/success?orderId=...)
    else Pago fallido
        StripeSDK-->>StripeForm: error
        StripeForm->>StripeForm: message.error
    end
```

## 4. Página de Éxito

```mermaid
sequenceDiagram
    actor Cliente as Cliente
    participant SuccessPage as SuccessPage.tsx
    participant OrderService as order.service
    participant Backend as Backend API
    
    Cliente->>SuccessPage: Llegar a /success?orderId=X
    SuccessPage->>SuccessPage: Cargar orden
    
    SuccessPage->>OrderService: getById(orderId)
    OrderService->>Backend: GET /api/orders/:id
    
    Backend-->>OrderService: Order completa
    OrderService-->>SuccessPage: Order data
    
    SuccessPage-->>Cliente: Mostrar detalles orden
    Note over Cliente: Numero, Estado, Items, Total
```

## Flujo de Pago Onvo

```mermaid
flowchart TD
    A[Checkout Form] --> B[Metodo: Onvo]
    B --> C[Submit]
    C --> D[Crear orden]
    D --> E[Onvo genera URL]
    E --> F[Redirect a Onvo]
    F --> G[Cliente paga en Onvo]
    G --> H[Webhook recibido]
    H --> I{Confirmacion?}
    I -->|Si| J[Orden actualizada paid]
    I -->|No| K[Orden pending]
    J --> L[Redirect success]
    K --> M[Esperar confirmacion]
    L --> N[Success Page]
```

## Flujo de Pago Stripe

```mermaid
flowchart TD
    A[Checkout Form] --> B[Metodo: Stripe]
    B --> C[Submit]
    C --> D[Crear orden Stripe]
    D --> E[Navigate payment page]
    E --> F[CheckoutPayment Page]
    F --> G{Validar minimo}
    G -->|Menor 250| H[Mostrar alerta]
    H --> I[Cliente agrega productos]
    G -->|Mayor 250| J[Cargar orden]
    J --> K[Mostrar Stripe form]
    K --> L[Cliente llena tarjeta]
    L --> M[Click Pagar]
    M --> N[Request clientSecret]
    N --> O[confirmCardPayment]
    O --> P{Pago exitoso?}
    P -->|Si| Q[Test mode?]
    P -->|No| R[Mostrar error]
    Q -->|Si| S[Auto update status]
    Q -->|No| T[Webhook update]
    S --> U[Clear cart]
    T --> U
    U --> V[Success Page]
```

## Validación de Monto Mínimo Stripe

```mermaid
graph TD
    A[CheckoutPayment Page] --> B[Cargar orden]
    B --> C[Monto orden]
    C --> D{Monto menor 250?}
    
    D -->|Si| E[Mostrar Alert]
    E --> F[Mensaje minimo]
    F --> G[Boton ver productos]
    G --> H[Cliente agrega mas]
    H --> I[Volver checkout]
    I --> B
    
    D -->|No| J[Formulario Stripe visible]
    J --> K[Cliente puede pagar]
```

## Estado de Orden

```mermaid
stateDiagram-v2
    [*] --> Pending: Crear orden
    Pending --> Paid: Pago confirmado
    Pending --> Failed: Pago fallido
    Pending --> Cancelled: Cliente cancela
    
    Paid --> Processing: En preparacion
    Processing --> Shipped: Enviado
    Shipped --> Delivered: Entregado
    
    Failed --> [*]
    Cancelled --> [*]
    Delivered --> [*]
```

## Webhook Onvo

```mermaid
sequenceDiagram
    participant Onvo as Onvo
    participant Backend as Backend API
    participant DB as Database
    
    Onvo->>Backend: POST /webhook/onvo
    Backend->>Backend: Validar firma webhook
    Backend->>Backend: Extraer orderId y status
    
    alt Pago exitoso
        Backend->>DB: Actualizar orden a paid
        Backend->>DB: Actualizar stock productos
        Backend-->>Onvo: 200 OK
    else Pago fallido
        Backend->>DB: Actualizar orden a failed
        Backend-->>Onvo: 200 OK
    end
```

## Test Mode Stripe

```mermaid
sequenceDiagram
    participant StripeForm as StripePaymentForm
    participant StripeSDK as Stripe
    participant Backend as Backend API
    
    StripeSDK->>StripeForm: paymentIntent succeeded
    StripeForm->>StripeForm: Check isTestMode
    
    alt Test mode (livemode: false)
        StripeForm->>Backend: PUT /orders/:id/status
        Note over StripeForm,Backend: { status: paid, stockUpdate: true, isStripeTestMode: true }
        Backend->>Backend: Actualizar orden
        Backend->>Backend: Reducir stock
        Backend-->>StripeForm: OK
    else Live mode
        Note over StripeForm: Webhook maneja actualizacion
    end
    
    StripeForm->>StripeForm: Clear cart y redirect
```

## Success Page - Información Mostrada

```mermaid
graph TD
    A[SuccessPage] --> B[Header Exito]
    B --> C[Icono CheckCircle Verde]
    B --> D[Titulo: Pago completado]
    B --> E[Subtitulo: Orden procesada]
    
    A --> F[Card Detalles Orden]
    F --> G[Numero de orden]
    F --> H[Estado]
    F --> I[Email]
    F --> J[Direccion]
    
    A --> K[Card Productos]
    K --> L[Lista Items]
    L --> M[Imagen producto]
    L --> N[Nombre + Variante]
    L --> O[Cantidad]
    L --> P[Precio]
    
    K --> Q[Total Orden]
    
    A --> R[Boton Continuar comprando]
    R --> S[Navigate /products]
```

