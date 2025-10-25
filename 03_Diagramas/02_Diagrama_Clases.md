# Diagrama de Clases - Parritico Store

## Descripción
Este diagrama muestra la estructura de clases del sistema Parritico Store basado en los modelos de datos de MongoDB/Mongoose.

## Diagrama

```mermaid
classDiagram
    %% Entidades principales
    class User {
        +ObjectId _id
        +String email
        +String password
        +String role
        +Date createdAt
        +Date updatedAt
    }
    
    class Product {
        +ObjectId _id
        +String name
        +String slug
        +String description
        +String brand
        +ObjectId category
        +ObjectId subcategory
        +Number basePrice
        +Number stock
        +Array~Image~ images
        +Array~Variant~ variants
        +Object attributes
        +Boolean isActive
        +Date createdAt
        +Date updatedAt
    }
    
    class Variant {
        +ObjectId _id
        +String sku
        +String size
        +String color
        +Number stock
        +Number price
        +Array~Image~ images
    }
    
    class Category {
        +ObjectId _id
        +String name
        +String slug
        +Object image
        +Boolean isActive
        +Date createdAt
        +Date updatedAt
    }
    
    class SubCategory {
        +ObjectId _id
        +String name
        +String slug
        +ObjectId category
        +Boolean isActive
        +Date createdAt
        +Date updatedAt
    }
    
    class Order {
        +ObjectId _id
        +Number invoiceNumber
        +ObjectId user
        +String email
        +Array~OrderItem~ items
        +Number amount
        +String currency
        +Object shippingAddress
        +String paymentProvider
        +String paymentIntentId
        +String status
        +Date createdAt
        +Date updatedAt
    }
    
    class OrderItem {
        +ObjectId product
        +String name
        +Object variant
        +Number unitPrice
        +Number quantity
        +String image
    }
    
    class Address {
        +String fullName
        +String phone
        +String line1
        +String line2
        +String city
        +String state
        +String zip
        +String country
    }
    
    class Contact {
        +ObjectId _id
        +String name
        +String email
        +String subject
        +String message
        +String status
        +Date createdAt
    }
    
    class Image {
        +String url
        +String public_id
    }
    
    %% Relaciones principales
    Product ||--o{ Variant : "tiene variantes"
    Product }o--|| Category : "pertenece a"
    Product }o--o| SubCategory : "pertenece a"
    SubCategory }o--|| Category : "pertenece a"
    Order ||--o{ OrderItem : "contiene"
    Order }o--|| Address : "tiene dirección"
    Order }o--o| User : "pertenece a"
    OrderItem }o--|| Product : "referencia"
    OrderItem }o--o| Variant : "referencia"
    Product ||--o{ Image : "tiene imágenes"
    Variant ||--o{ Image : "tiene imágenes"
    Category ||--o{ Image : "tiene imagen"
    
    %% Estilos
    classDef entity fill:#e3f2fd,stroke:#1976d2,stroke-width:2px
    classDef valueObject fill:#f3e5f5,stroke:#7b1fa2,stroke-width:2px
    classDef embedded fill:#fff8e1,stroke:#f57c00,stroke-width:2px
    
    class User,Product,Category,SubCategory,Order,Contact entity
    class Address,Image valueObject
    class Variant,OrderItem embedded
```

## Descripción de Clases

### Entidades Principales (Colecciones MongoDB)

#### User
- **Propósito**: Gestionar usuarios del sistema (administradores)
- **Atributos clave**: email, password, role
- **Validaciones**: Email único, contraseña mínima 8 caracteres
- **Roles**: admin, organizer
- **Colección**: users

#### Product
- **Propósito**: Representar productos de la tienda con variantes
- **Atributos clave**: name, slug, basePrice, stock, variants, images
- **Características**: Soporte para variantes embebidas, imágenes múltiples
- **Relaciones**: Referencia a Category y SubCategory
- **Colección**: products

#### Category
- **Propósito**: Organizar productos en categorías principales
- **Atributos clave**: name, slug, image, isActive
- **Características**: Soporte para imagen, activación/desactivación
- **Colección**: categories

#### SubCategory
- **Propósito**: Subclasificar productos dentro de categorías
- **Atributos clave**: name, slug, category, isActive
- **Relaciones**: Referencia a Category padre
- **Colección**: subcategories

#### Order
- **Propósito**: Gestionar pedidos de compra con integración de pagos
- **Atributos clave**: invoiceNumber, email, items, amount, status
- **Estados**: pending, paid, failed, processing, shipped, delivered, cancelled
- **Integración**: Onvo para pagos, referencias a User (opcional)
- **Colección**: orders

#### Contact
- **Propósito**: Gestionar mensajes de contacto de clientes
- **Atributos clave**: name, email, subject, message, status
- **Estados**: pending, reviewed
- **Colección**: contacts

### Objetos de Valor (Embebidos)

#### Variant
- **Propósito**: Representar variantes de productos (talla, color, SKU)
- **Atributos clave**: sku, size, color, stock, price, images
- **Características**: Stock independiente, precio específico, imágenes propias
- **Ubicación**: Embebido en Product.variants[]

#### OrderItem
- **Propósito**: Representar un producto en un pedido (snapshot)
- **Atributos clave**: product, name, variant, unitPrice, quantity, image
- **Características**: Snapshot del producto al momento de compra
- **Ubicación**: Embebido en Order.items[]

#### Address
- **Propósito**: Representar direcciones de envío
- **Atributos clave**: fullName, line1, city, country, phone, state, zip
- **Validaciones**: Campos obligatorios para envío
- **Ubicación**: Embebido en Order.shippingAddress

#### Image
- **Propósito**: Gestionar imágenes de productos, variantes y categorías
- **Atributos clave**: url, public_id
- **Integración**: Cloudinary para almacenamiento y CDN
- **Ubicación**: Embebido en Product.images[], Variant.images[], Category.image

## Relaciones Principales

### Relaciones Referenciales (Foreign Keys)
1. **Product → Category**: `product.category` → `category._id` (Many-to-One)
2. **Product → SubCategory**: `product.subcategory` → `subcategory._id` (Many-to-One, opcional)
3. **SubCategory → Category**: `subcategory.category` → `category._id` (Many-to-One)
4. **Order → User**: `order.user` → `user._id` (Many-to-One, opcional)
5. **OrderItem → Product**: `orderItem.product` → `product._id` (Many-to-One)

### Relaciones Embebidas (Documentos Anidados)
1. **Product → Variants**: `product.variants[]` (One-to-Many embebido)
2. **Product → Images**: `product.images[]` (One-to-Many embebido)
3. **Variant → Images**: `variant.images[]` (One-to-Many embebido)
4. **Order → OrderItems**: `order.items[]` (One-to-Many embebido)
5. **Order → Address**: `order.shippingAddress` (One-to-One embebido)
6. **Category → Image**: `category.image` (One-to-One embebido)

### Cardinalidades
- **Product-Category**: N:1 (Muchos productos → Una categoría)
- **Product-SubCategory**: N:1 (Muchos productos → Una subcategoría)
- **SubCategory-Category**: N:1 (Muchas subcategorías → Una categoría)
- **Order-User**: N:1 (Muchos pedidos → Un usuario, opcional)
- **Product-Variant**: 1:N (Un producto → Muchas variantes embebidas)
- **Order-OrderItem**: 1:N (Un pedido → Muchos items embebidos)

## Validaciones y Restricciones

### Validaciones de Integridad
- **Email único**: En User y Contact
- **Slug único**: En Product, Category, SubCategory
- **InvoiceNumber único**: En Order (auto-incremental)
- **Referencias válidas**: Foreign keys deben apuntar a documentos existentes

### Validaciones de Negocio
- **Stock no negativo**: En Product y Variant
- **Precios positivos**: En Product y Variant
- **Estados válidos**: En Order y Contact (enums predefinidos)
- **Email válido**: Formato correcto en User, Contact y Order
- **Campos requeridos**: Validación de campos obligatorios

### Restricciones de MongoDB
- **ObjectId válidos**: Para todas las referencias
- **Índices únicos**: En campos de unicidad
- **Validación de esquema**: Mongoose schema validation
- **Timestamps automáticos**: createdAt y updatedAt
