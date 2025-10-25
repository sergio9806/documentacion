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
        +validateEmail()
        +hashPassword()
        +comparePassword()
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
        +calculateTotalStock()
        +updateStock()
    }
    
    class Variant {
        +ObjectId _id
        +String sku
        +String size
        +String color
        +Number stock
        +Number price
        +Array~Image~ images
        +calculatePrice()
        +updateStock()
    }
    
    class Category {
        +ObjectId _id
        +String name
        +String slug
        +Image image
        +Boolean isActive
        +Date createdAt
        +Date updatedAt
        +validateSlug()
    }
    
    class SubCategory {
        +ObjectId _id
        +String name
        +String slug
        +ObjectId category
        +Boolean isActive
        +Date createdAt
        +Date updatedAt
        +validateCategory()
    }
    
    class Order {
        +ObjectId _id
        +Number invoiceNumber
        +ObjectId user
        +String email
        +Array~OrderItem~ items
        +Number amount
        +String currency
        +Address shippingAddress
        +String paymentProvider
        +String paymentIntentId
        +String status
        +Date createdAt
        +Date updatedAt
        +calculateTotal()
        +updateStatus()
        +generateInvoiceNumber()
    }
    
    class OrderItem {
        +ObjectId product
        +String name
        +Variant variant
        +Number unitPrice
        +Number quantity
        +String image
        +calculateSubtotal()
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
        +validateAddress()
    }
    
    class Contact {
        +ObjectId _id
        +String name
        +String email
        +String subject
        +String message
        +String status
        +Date createdAt
        +updateStatus()
    }
    
    class Image {
        +String url
        +String public_id
        +validateUrl()
    }
    
    %% Relaciones
    Product ||--o{ Variant : contains
    Product }o--|| Category : belongs to
    Product }o--o| SubCategory : belongs to
    SubCategory }o--|| Category : belongs to
    Order ||--o{ OrderItem : contains
    Order }o--|| Address : has
    Order }o--o| User : belongs to
    OrderItem }o--|| Product : references
    OrderItem }o--o| Variant : references
    Product ||--o{ Image : has
    Variant ||--o{ Image : has
    Category ||--o{ Image : has
    
    %% Estilos
    classDef entity fill:#e3f2fd,stroke:#1976d2,stroke-width:2px
    classDef valueObject fill:#f3e5f5,stroke:#7b1fa2,stroke-width:2px
    classDef embedded fill:#fff8e1,stroke:#f57c00,stroke-width:2px
    
    class User,Product,Category,SubCategory,Order,Contact entity
    class Address,Image valueObject
    class Variant,OrderItem embedded
```

## Descripción de Clases

### Entidades Principales

#### User
- **Propósito**: Gestionar usuarios del sistema (administradores)
- **Atributos clave**: email, password, role
- **Validaciones**: Email único, contraseña mínima 8 caracteres
- **Roles**: admin, organizer

#### Product
- **Propósito**: Representar productos de la tienda
- **Atributos clave**: name, slug, basePrice, stock, variants
- **Características**: Soporte para variantes, imágenes múltiples, atributos personalizados
- **Relaciones**: Pertenece a una categoría y opcionalmente a una subcategoría

#### Category
- **Propósito**: Organizar productos en categorías
- **Atributos clave**: name, slug, image, isActive
- **Características**: Soporte para imágenes, activación/desactivación

#### SubCategory
- **Propósito**: Subclasificar productos dentro de categorías
- **Atributos clave**: name, slug, category
- **Relaciones**: Pertenece a una categoría padre

#### Order
- **Propósito**: Gestionar pedidos de compra
- **Atributos clave**: invoiceNumber, email, items, amount, status
- **Estados**: pending, paid, failed, processing, shipped, delivered, cancelled
- **Integración**: Onvo para pagos

#### Contact
- **Propósito**: Gestionar mensajes de contacto
- **Atributos clave**: name, email, subject, message, status
- **Estados**: pending, reviewed

### Objetos de Valor

#### Variant
- **Propósito**: Representar variantes de productos (talla, color, etc.)
- **Atributos clave**: sku, size, color, stock, price
- **Características**: Stock independiente, precio específico

#### OrderItem
- **Propósito**: Representar un producto en un pedido
- **Atributos clave**: product, name, variant, unitPrice, quantity
- **Características**: Snapshot del producto al momento de la compra

#### Address
- **Propósito**: Representar direcciones de envío
- **Atributos clave**: fullName, line1, city, country
- **Validaciones**: Campos obligatorios para envío

#### Image
- **Propósito**: Gestionar imágenes de productos y categorías
- **Atributos clave**: url, public_id
- **Integración**: Cloudinary para almacenamiento

## Relaciones Principales

1. **Product-Category**: Un producto pertenece a una categoría
2. **Product-SubCategory**: Un producto puede pertenecer a una subcategoría
3. **SubCategory-Category**: Una subcategoría pertenece a una categoría
4. **Product-Variant**: Un producto puede tener múltiples variantes
5. **Order-OrderItem**: Un pedido contiene múltiples items
6. **OrderItem-Product**: Un item referencia un producto
7. **Order-User**: Un pedido puede pertenecer a un usuario (opcional)

## Validaciones y Restricciones

- **Email único**: En User y Contact
- **Slug único**: En Product, Category, SubCategory
- **Stock no negativo**: En Product y Variant
- **Precios positivos**: En Product y Variant
- **Estados válidos**: En Order y Contact
- **Referencias válidas**: Foreign keys entre entidades
