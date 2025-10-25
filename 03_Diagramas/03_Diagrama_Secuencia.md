# Diagrama de Secuencia - Parritico Store

## Descripción
Este diagrama muestra los flujos de secuencia principales del sistema: agregar al carrito, checkout y proceso de pago.

## 1. Flujo: Agregar Producto al Carrito

```mermaid
sequenceDiagram
    participant C as Cliente
    participant F as Frontend
    participant B as Backend
    participant DB as Base de Datos
    
    C->>F: Selecciona producto
    F->>B: GET /api/products/:id
    B->>DB: Product.findById(id)
    DB-->>B: Product data
    B-->>F: Product details
    F-->>C: Muestra detalles del producto
    
    C->>F: Selecciona variante (opcional)
    C->>F: Especifica cantidad
    C->>F: Click "Agregar al Carrito"
    
    F->>F: Validar stock disponible
    F->>F: useCartStore.addToCart()
    F->>F: Persistir en localStorage
    F-->>C: Confirmación "Producto agregado"
    
    Note over C,F: El carrito se mantiene en el estado local del cliente
```

## 2. Flujo: Proceso de Checkout

```mermaid
sequenceDiagram
    participant C as Cliente
    participant F as Frontend
    participant B as Backend
    participant DB as Base de Datos
    participant O as Onvo API
    
    C->>F: Click "Proceder al Checkout"
    F->>F: Validar carrito no vacío
    F->>F: Mostrar formulario de datos
    
    C->>F: Completa datos de envío
    C->>F: Confirma información
    F->>B: POST /api/orders
    
    B->>DB: Validar productos y stock
    DB-->>B: Productos válidos
    
    B->>B: Calcular total
    B->>DB: Generar número de factura
    B->>DB: Guardar orden (status: pending)
    
    B->>O: POST /payment-intents
    Note over B,O: amount * 100, currency: CRC
    O-->>B: Payment Intent ID
    
    B->>O: POST /checkout-sessions
    Note over B,O: success_url, cancel_url
    O-->>B: Checkout URL
    
    B->>DB: Actualizar orden con paymentIntentId
    B-->>F: { order, paymentIntent: { url } }
    F->>F: Redirigir a Onvo
    F-->>C: Redirección a página de pago
```

## 3. Flujo: Proceso de Pago

```mermaid
sequenceDiagram
    participant C as Cliente
    participant O as Onvo
    participant B as Backend
    participant DB as Base de Datos
    participant F as Frontend
    
    C->>O: Accede a checkout URL
    O->>C: Muestra formulario de pago
    
    C->>O: Completa datos de pago
    C->>O: Confirma pago
    
    alt Pago Exitoso
        O->>B: Webhook: payment.succeeded
        B->>DB: Buscar orden por paymentIntentId
        B->>DB: Actualizar status: "paid"
        B->>DB: Reducir stock de productos
        B-->>O: 200 OK
        
        O->>F: Redirección a success_url
        F->>B: GET /api/orders/:id
        B->>DB: Buscar orden
        DB-->>B: Order data
        B-->>F: Order details
        F-->>C: Página de confirmación
    else Pago Fallido
        O->>B: Webhook: payment.failed
        B->>DB: Actualizar status: "failed"
        B-->>O: 200 OK
        
        O->>F: Redirección a cancel_url
        F-->>C: Página de error de pago
    end
```

## 4. Flujo: Gestión de Pedidos (Admin)

```mermaid
sequenceDiagram
    participant A as Administrador
    participant FA as Frontend Admin
    participant B as Backend
    participant DB as Base de Datos
    
    A->>FA: Accede al panel de administración
    FA->>B: GET /api/orders (con autenticación)
    B->>B: Verificar token JWT
    B->>DB: Order.find().populate()
    DB-->>B: Lista de pedidos
    B-->>FA: Orders data
    FA-->>A: Dashboard con pedidos
    
    A->>FA: Selecciona pedido
    FA->>B: GET /api/orders/:id
    B->>DB: Order.findById()
    DB-->>B: Order details
    B-->>FA: Order data
    FA-->>A: Detalles del pedido
    
    A->>FA: Cambia estado del pedido
    FA->>B: PUT /api/orders/:id/status
    B->>DB: Order.findByIdAndUpdate()
    DB-->>B: Updated order
    B-->>FA: Success response
    FA-->>A: Estado actualizado
```

## 5. Flujo: Gestión de Productos (Admin)

```mermaid
sequenceDiagram
    participant A as Administrador
    participant FA as Frontend Admin
    participant B as Backend
    participant DB as Base de Datos
    participant C as Cloudinary
    
    A->>FA: Crear nuevo producto
    FA->>A: Muestra formulario
    
    A->>FA: Completa datos del producto
    A->>FA: Sube imágenes
    FA->>B: POST /api/products (multipart/form-data)
    
    B->>B: Validar datos
    B->>DB: Verificar categoría existe
    DB-->>B: Category valid
    
    B->>C: Upload images to Cloudinary
    C-->>B: Image URLs y public_ids
    
    B->>DB: Product.create()
    DB-->>B: New product
    B-->>FA: Product created
    FA-->>A: Confirmación de creación
    
    Note over A,C: El proceso incluye validación de categorías, subida de imágenes y creación de variantes
```

## 6. Flujo: Webhook de Pago

```mermaid
sequenceDiagram
    participant O as Onvo
    participant B as Backend
    participant DB as Base de Datos
    participant S as Socket.IO
    
    O->>B: POST /api/webhooks/onvo
    Note over O,B: Evento: payment.succeeded o payment.failed
    
    B->>B: Verificar firma del webhook
    B->>B: Extraer payment_intent_id
    
    B->>DB: Order.findOne({ paymentIntentId })
    DB-->>B: Order found
    
    alt Pago Exitoso
        B->>DB: Order.updateOne({ status: "paid" })
        B->>DB: Reducir stock de productos
        B->>S: Emit "order:paid" event
    else Pago Fallido
        B->>DB: Order.updateOne({ status: "failed" })
        B->>S: Emit "order:failed" event
    end
    
    B-->>O: 200 OK
    S-->>B: Notificar cambios en tiempo real
```

## Descripción de los Flujos

### Flujo de Compra Completo
1. **Navegación**: Cliente explora productos
2. **Selección**: Agrega productos al carrito
3. **Checkout**: Completa datos de envío
4. **Pago**: Procesa pago con Onvo
5. **Confirmación**: Recibe confirmación de compra
6. **Gestión**: Admin gestiona el pedido

### Puntos de Integración
- **Onvo**: Para procesamiento de pagos
- **Cloudinary**: Para gestión de imágenes
- **Socket.IO**: Para notificaciones en tiempo real
- **JWT**: Para autenticación de administradores

### Manejo de Errores
- Validación de stock antes del pago
- Rollback en caso de fallos
- Webhooks para sincronización de estados
- Notificaciones en tiempo real
