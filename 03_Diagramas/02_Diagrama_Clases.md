# Diagrama de Clases - Parritico Store

## Descripción
Este diagrama muestra la estructura de clases del sistema Parritico Store basado en los modelos de datos de MongoDB/Mongoose.

## Diagrama

```mermaid
classDiagram
    class User {
        +_id
        +email
        +password
        +role
        +createdAt
        +updatedAt
    }
    
    class Product {
        +_id
        +name
        +slug
        +description
        +brand
        +category
        +subcategory
        +basePrice
        +stock
        +images
        +variants
        +attributes
        +isActive
        +createdAt
        +updatedAt
    }
    
    class Variant {
        +_id
        +sku
        +size
        +color
        +stock
        +price
        +images
    }
    
    class Category {
        +_id
        +name
        +slug
        +image
        +isActive
        +createdAt
        +updatedAt
    }
    
    class SubCategory {
        +_id
        +name
        +slug
        +category
        +isActive
        +createdAt
        +updatedAt
    }
    
    class Order {
        +_id
        +invoiceNumber
        +user
        +email
        +items
        +amount
        +currency
        +shippingAddress
        +paymentProvider
        +paymentIntentId
        +status
        +createdAt
        +updatedAt
    }
    
    class OrderItem {
        +product
        +name
        +variant
        +unitPrice
        +quantity
        +image
    }
    
    class Address {
        +fullName
        +phone
        +line1
        +line2
        +city
        +state
        +zip
        +country
    }
    
    class Contact {
        +_id
        +name
        +email
        +subject
        +message
        +status
        +createdAt
    }
    
    class Image {
        +url
        +public_id
    }
    
    Product ||--o{ Variant
    Product }o--|| Category
    Product }o--o| SubCategory
    SubCategory }o--|| Category
    Order ||--o{ OrderItem
    Order }o--|| Address
    Order }o--o| User
    OrderItem }o--|| Product
    OrderItem }o--o| Variant
    Product ||--o{ Image
    Variant ||--o{ Image
    Category ||--o{ Image
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
