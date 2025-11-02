# Diagrama de Caso de Uso: Proceso de Checkout

## 1. Iniciar Checkout

```mermaid
sequenceDiagram
    actor Cliente as Cliente
    participant CartPage as CartPage.tsx
    participant CheckoutPage as CheckoutPage.tsx
    participant CartStore as cart.store
    
    Cliente->>CartPage: Click Finalizar compra
    CartPage->>CheckoutPage: Navegar a /checkout
    CheckoutPage->>CartStore: get items
    
    alt Carrito vacio
        CheckoutPage->>CheckoutPage: Mostrar warning
        CheckoutPage-->>Cliente: Mensaje carrito vacio
    else Con items
        CheckoutPage->>CheckoutPage: Calcular total
        CheckoutPage-->>Cliente: Mostrar formulario checkout
    end
```

## 2. Llenar Formulario de Checkout

```mermaid
sequenceDiagram
    actor Cliente as Cliente
    participant CheckoutPage as CheckoutPage.tsx
    participant Form as Antd Form
    
    Cliente->>CheckoutPage: Llenar datos
    Note over Cliente,CheckoutPage: Nombre completo, Email, Telefono, Direccion, Ciudad
    
    Cliente->>CheckoutPage: Seleccionar metodo pago
    CheckoutPage->>CheckoutPage: setPaymentMethod(metodo)
    Note over CheckoutPage: onvo o stripe
    
    Cliente->>CheckoutPage: Click Continuar al pago
    CheckoutPage->>Form: validateFields()
    
    alt Validacion exitosa
        CheckoutPage->>CheckoutPage: handleFinish(values)
    else Validacion fallida
        Form-->>Cliente: Mostrar errores campos
    end
```

## 3. Procesar Checkout - Onvo

```mermaid
sequenceDiagram
    actor Cliente as Cliente
    participant CheckoutPage as CheckoutPage.tsx
    participant OrderService as order.service
    participant Backend as Backend API
    participant CartStore as cart.store
    
    CheckoutPage->>CheckoutPage: Crear orderData
    Note over CheckoutPage: email, shippingAddress, items
    
    CheckoutPage->>OrderService: create(orderData)
    OrderService->>Backend: POST /api/orders
    
    Backend->>Backend: Crear orden
    Backend->>Backend: Generar paymentIntent Onvo
    Backend-->>OrderService: { paymentIntent: { url } }
    OrderService-->>CheckoutPage: Response con URL
    
    CheckoutPage->>CartStore: clearCart()
    CheckoutPage->>CheckoutPage: message.success
    CheckoutPage->>CheckoutPage: window.location.href = url
    CheckoutPage-->>Cliente: Redirigir a Onvo checkout
```

## 4. Procesar Checkout - Stripe

```mermaid
sequenceDiagram
    actor Cliente as Cliente
    participant CheckoutPage as CheckoutPage.tsx
    participant OrderService as order.service
    participant Backend as Backend API
    participant Router as React Router
    
    CheckoutPage->>CheckoutPage: Crear orderData
    CheckoutPage->>OrderService: createStripeOrder(orderData)
    OrderService->>Backend: POST /api/orders/stripe
    
    Backend->>Backend: Crear orden
    Backend->>Backend: Crear paymentIntent Stripe
    Backend-->>OrderService: { orderId }
    OrderService-->>CheckoutPage: Response con orderId
    
    CheckoutPage->>CheckoutPage: message.success
    CheckoutPage->>Router: navigate(/checkout/payment?orderId=...)
    Router-->>Cliente: Redirigir a Stripe payment
```

## Estructura de CheckoutPage

```mermaid
graph TD
    A[CheckoutPage] --> B[Header: Finalizar compra]
    
    A --> C[Row Layout]
    C --> D[Col Form 14/24]
    C --> E[Col Summary 10/24]
    
    D --> F[Form Vertical]
    F --> G[Nombre completo]
    F --> H[Email]
    F --> I[Telefono opcional]
    F --> J[Direccion]
    F --> K[Ciudad]
    
    F --> L[Divider]
    
    F --> M[Seccion Metodo Pago]
    M --> N[Radio Onvo]
    N --> O[Badge Transferencia]
    O --> P[Subtexto Onvo]
    
    M --> Q[Radio Stripe]
    Q --> R[Badge Tarjeta]
    R --> S[Subtexto Stripe]
    
    F --> T[Boton Continuar]
    T --> U{Metodo seleccionado?}
    U -->|Onvo| V[Flujo Onvo]
    U -->|Stripe| W[Flujo Stripe]
    
    E --> X[Summary Card]
    X --> Y[Total Items]
    X --> Z[Total Precio]
```

## Selección de Método de Pago

```mermaid
stateDiagram-v2
    [*] --> Default: Inicio
    Default --> Onvo: onvo seleccionado por defecto
    
    Onvo --> Stripe: Cliente click Stripe
    Stripe --> Onvo: Cliente click Onvo
    
    Onvo --> SubmitOnvo: Submit form
    Stripe --> SubmitStripe: Submit form
    
    SubmitOnvo --> ProcessOnvo: Crear orden Onvo
    ProcessOnvo --> RedirectOnvo: Redirigir a Onvo
    
    SubmitStripe --> ProcessStripe: Crear orden Stripe
    ProcessStripe --> RedirectStripe: Redirigir a payment
    
    RedirectOnvo --> [*]
    RedirectStripe --> [*]
```

## Construcción de OrderData

```mermaid
graph TD
    A[Form Values] --> B[orderData]
    B --> C[email]
    B --> D[shippingAddress]
    B --> E[items]
    
    D --> F[fullName]
    D --> G[phone]
    D --> H[line1: address]
    D --> I[city]
    D --> J[country: Costa Rica]
    
    E --> K[Por cada item del carrito]
    K --> L[product: _id]
    K --> M{variant?}
    
    M -->|Si| N[variant object]
    N --> O[_id]
    N --> P[sku]
    N --> Q[color]
    N --> R[size]
    
    M -->|No| S[undefined]
    
    K --> T[quantity]
```

## Validaciones del Formulario

```mermaid
graph TD
    A[Formulario Checkout] --> B[Campo Nombre]
    B --> B1{¿Requerido?}
    B1 -->|No| B2[Error: Requerido]
    B1 -->|Si| B3[OK]
    
    A --> C[Campo Email]
    C --> C1{¿Requerido?}
    C1 -->|No| C2[Error: Requerido]
    C1 -->|Si| C3{¿Formato email?}
    C3 -->|No| C4[Error: Correo inválido]
    C3 -->|Si| C5[OK]
    
    A --> D[Campo Telefono]
    D --> D1[Opcional]
    D1 --> D2[OK]
    
    A --> E[Campo Direccion]
    E --> E1{¿Requerido?}
    E1 -->|No| E2[Error: Requerido]
    E1 -->|Si| E3[OK]
    
    A --> F[Campo Ciudad]
    F --> F1{¿Requerido?}
    F1 -->|No| F2[Error: Requerido]
    F1 -->|Si| F3[OK]
    
    B3 --> G{Todos validos?}
    C5 --> G
    D2 --> G
    E3 --> G
    F3 --> G
    
    G -->|Si| H[Submit habilitado]
    G -->|No| I[Submit deshabilitado]
```

## Flujo de Checkout Completo

```mermaid
flowchart TD
    A[Cart Page] --> B[Click Finalizar]
    B --> C{¿Carrito vacio?}
    C -->|Si| D[Mostrar warning]
    C -->|No| E[Checkout Page]
    
    E --> F[Mostrar Formulario]
    F --> G[Cliente llena datos]
    G --> H[Selecciona metodo pago]
    
    H --> I{Metodo?}
    I -->|Onvo| J[Click Continuar Onvo]
    I -->|Stripe| K[Click Continuar Stripe]
    
    J --> L[Validar form]
    K --> L
    L --> M{¿Valido?}
    M -->|No| N[Mostrar errores]
    N --> G
    
    M -->|Si| O{Metodo?}
    O -->|Onvo| P[create Onvo order]
    O -->|Stripe| Q[createStripeOrder]
    
    P --> R[Redirect Onvo]
    Q --> S[Redirect Stripe Payment]
    
    R --> T[Onvo Checkout]
    S --> U[Stripe Payment Page]
    
    T --> V[Customer paga]
    U --> W[Customer paga]
    
    V --> X[Success Page]
    W --> X
```

