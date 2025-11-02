# Diagrama de Caso de Uso: Gestión de Carrito

## 1. Agregar Producto al Carrito

```mermaid
sequenceDiagram
    actor Cliente as Cliente
    participant ProductDetail as ProductDetailPage.tsx
    participant CartStore as cart.store.ts
    participant Swal as SweetAlert2
    participant Router as React Router
    
    Cliente->>ProductDetail: Click Agregar al Carrito
    
    ProductDetail->>ProductDetail: Validar disponibilidad
    alt Producto agotado
        ProductDetail->>Swal: Mostrar warning
        Swal-->>Cliente: Producto agotado
    else Seleccionar variante requerida
        ProductDetail->>Swal: Mostrar info
        Swal-->>Cliente: Selecciona variante
    else Stock insuficiente
        ProductDetail->>Swal: Mostrar error
        Swal-->>Cliente: Solo X unidades
    else Todo OK
        ProductDetail->>CartStore: addToCart(product, variant, quantity)
        
        CartStore->>CartStore: Buscar si existe en items
        alt Ya existe
            CartStore->>CartStore: Incrementar cantidad
        else Nuevo producto
            CartStore->>CartStore: Agregar nuevo item
        end
        
        CartStore->>CartStore: Guardar en localStorage
        
        ProductDetail->>ProductDetail: Actualizar stock local
        ProductDetail->>Swal: Mostrar success
        Swal-->>Cliente: Confirmacion con opciones
    end
```

## 2. Ver Carrito

```mermaid
sequenceDiagram
    actor Cliente as Cliente
    participant CartPage as CartPage.tsx
    participant CartStore as cart.store.ts
    
    Cliente->>CartPage: Navegar a /cart
    CartPage->>CartStore: get items
    
    alt Carrito vacio
        CartPage-->>Cliente: Mostrar mensaje vacio
        CartPage-->>Cliente: Boton ver productos
    else Con items
        CartPage->>CartPage: Calcular totales
        CartPage-->>Cliente: Mostrar items y resumen
    end
```

## 3. Actualizar Cantidad en Carrito

```mermaid
sequenceDiagram
    actor Cliente as Cliente
    participant CartPage as CartPage.tsx
    participant CartStore as cart.store.ts
    participant Message as Antd Message
    
    Cliente->>CartPage: Click incrementar cantidad
    
    alt Cantidad menor a stock
        CartPage->>CartStore: updateQuantity(productId, variantId, quantity + 1)
        CartStore->>CartStore: Actualizar item en lista
        CartStore->>CartStore: Guardar en localStorage
        CartPage->>CartPage: Recalcular totales
        CartPage-->>Cliente: Actualizar display
    else Cantidad igual o mayor a stock
        CartPage->>Message: Warning
        Message-->>Cliente: Solo X unidades disponibles
    end
```

## 4. Eliminar Producto del Carrito

```mermaid
sequenceDiagram
    actor Cliente as Cliente
    participant CartPage as CartPage.tsx
    participant CartStore as cart.store.ts
    
    Cliente->>CartPage: Click eliminar producto
    CartPage->>CartStore: removeFromCart(productId, variantId)
    CartStore->>CartStore: Filtrar items
    CartStore->>CartStore: Guardar en localStorage
    CartPage->>CartPage: Recalcular totales
    CartPage-->>Cliente: Actualizar lista
```

## 5. Limpiar Carrito Completo

```mermaid
sequenceDiagram
    actor Cliente as Cliente
    participant CartPage as CartPage.tsx
    participant CartStore as cart.store.ts
    
    Cliente->>CartPage: Pago exitoso
    CartPage->>CartStore: clearCart()
    CartStore->>CartStore: set items = []
    CartStore->>CartStore: Limpiar localStorage
    CartPage-->>Cliente: Carrito vacio
```

## Persistencia del Carrito

```mermaid
sequenceDiagram
    actor Cliente as Cliente
    participant CartStore as cart.store.ts
    participant Storage as localStorage
    
    Note over CartStore,Storage: Zustand Persist Middleware
    
    Cliente->>CartStore: Agregar producto
    CartStore->>CartStore: Actualizar estado
    CartStore->>Storage: Guardar items serializados
    
    Cliente->>CartStore: Recargar pagina
    CartStore->>Storage: Leer items almacenados
    Storage-->>CartStore: Items deserializados
    CartStore->>CartStore: Restaurar estado
    CartStore-->>Cliente: Carrito restaurado
```

## Estado del Carrito

```mermaid
graph TD
    A[CartStore] --> B[State]
    B --> C[items: CartItem[]]
    
    C --> D[CartItem]
    D --> E[product: Product]
    D --> F[variant?: Variant]
    D --> G[quantity: number]
    
    A --> H[Actions]
    H --> I[addToCart]
    H --> J[removeFromCart]
    H --> K[updateQuantity]
    H --> L[clearCart]
    
    I --> M{¿Existe?}
    M -->|Si| N[Incrementar cantidad]
    M -->|No| O[Agregar nuevo]
    
    J --> P[Filtrar items]
    K --> Q[Actualizar item]
    L --> R[Vaciar items]
    
    N --> S[localStorage]
    O --> S
    P --> S
    Q --> S
    R --> S
```

## Vista de Carrito

```mermaid
graph TD
    A[CartPage] --> B{¿Carrito vacio?}
    
    B -->|Si| C[Mensaje vacio]
    C --> D[Boton ver productos]
    
    B -->|No| E[Lista Items]
    E --> F[Item Card]
    F --> G[Imagen Producto]
    F --> H[Nombre + Variante]
    F --> I[Control Cantidad]
    I --> J[Boton Menos]
    I --> K[Display Cantidad]
    I --> L[Boton Mas]
    F --> M[Subtotal]
    F --> N[Boton Eliminar]
    
    E --> O[Resumen Pedido]
    O --> P[Total Items]
    O --> Q[Total Precio]
    O --> R[Boton Finalizar Compra]
    R --> S[Ir a Checkout]
```

## Flujo de Agregado al Carrito

```mermaid
stateDiagram-v2
    [*] --> ClickAgregar: Cliente click
    ClickAgregar --> ValidarStock: Iniciar proceso
    
    ValidarStock --> SinStock: Stock = 0
    ValidarStock --> VarianteRequerida: Tiene variantes y no seleccionada
    ValidarStock --> Insuficiente: Cantidad > stock
    ValidarStock --> Validado: Todo OK
    
    SinStock --> MostrarWarning: Warning
    VarianteRequerida --> MostrarInfo: Info
    Insuficiente --> MostrarError: Error
    
    Validado --> BuscarExiste: En cart store
    BuscarExiste --> Existe: Item encontrado
    BuscarExiste --> NoExiste: Item nuevo
    
    Existe --> Incrementar: quantity += cantidad
    NoExiste --> Agregar: Nuevo item
    
    Incrementar --> Guardar: Actualizar localStorage
    Agregar --> Guardar
    
    Guardar --> ActualizarStock: Stock local -= cantidad
    ActualizarStock --> MostrarExito: Swal success
    MostrarExito --> Opciones: Ver carrito o Seguir
    
    MostrarWarning --> [*]
    MostrarInfo --> [*]
    MostrarError --> [*]
    Opciones --> [*]
```

## Cálculo de Totales

```mermaid
graph LR
    A[CartItems] --> B[Por cada item]
    B --> C{Precio}
    C -->|Con variante| D[variant.price]
    C -->|Sin variante| E[product.basePrice]
    
    D --> F[Precio unitario]
    E --> F
    
    F --> G[Cantidad]
    G --> H[Subtotal = precio * cantidad]
    
    H --> I[Sumar todos subtotales]
    I --> J[Total del Carrito]
    
    B --> K[Total Items]
    K --> L[Sumar cantidades]
    L --> M[Items en carrito]
```

