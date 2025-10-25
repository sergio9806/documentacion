# Modelo de Datos - Parritico Store

## Descripción General
El modelo de datos de Parritico Store está diseñado para soportar un e-commerce completo con funcionalidades avanzadas como variantes de productos, gestión de inventario, procesamiento de pagos y administración de contenido.

## Base de Datos
- **Sistema**: MongoDB
- **ODM**: Mongoose
- **Características**: NoSQL, documentos flexibles, relaciones embebidas y referenciales

## Colecciones Principales

### 1. Users (Usuarios)

#### Propósito
Gestionar usuarios del sistema, principalmente administradores y organizadores.

#### Esquema
```typescript
interface IUser {
  _id: ObjectId;
  email: string;           // Único, formato email válido
  password: string;        // Hash bcrypt, mínimo 8 caracteres
  role: "admin" | "organizer";
  createdAt: Date;
  updatedAt: Date;
}
```

#### Validaciones
- **email**: Requerido, único, formato válido, lowercase, trim
- **password**: Requerido, mínimo 8 caracteres
- **role**: Enum ["admin", "organizer"], default "organizer"

#### Índices
- `email`: Único
- `role`: Para consultas por rol

#### Relaciones
- **One-to-Many**: Orders (opcional)

---

### 2. Products (Productos)

#### Propósito
Gestionar el catálogo de productos con soporte para variantes, imágenes múltiples y atributos personalizados.

#### Esquema
```typescript
interface IProduct {
  _id: ObjectId;
  name: string;                    // Requerido, trim
  slug: string;                   // Único, lowercase, URL-friendly
  description?: string;
  brand?: string;
  category: ObjectId;             // Referencia a Category
  subcategory?: ObjectId;         // Referencia a SubCategory
  basePrice: number;              // Precio base del producto
  stock?: number;                 // Stock general (si no hay variantes)
  images: Array<{
    url: string;
    public_id?: string;
  }>;
  variants: Array<IVariant>;      // Variantes del producto
  attributes?: Record<string, any>; // Atributos personalizados
  isActive: boolean;              // Estado del producto
  createdAt: Date;
  updatedAt: Date;
}

interface IVariant {
  _id: ObjectId;
  sku?: string;                   // SKU único de la variante
  size?: string;                  // Talla
  color?: string;                 // Color
  stock: number;                  // Stock de la variante
  price: number;                  // Precio de la variante
  images?: Array<{
    url: string;
    public_id?: string;
  }>;
}
```

#### Validaciones
- **name**: Requerido, trim
- **slug**: Requerido, único, lowercase
- **basePrice**: Requerido, número positivo
- **stock**: Número no negativo
- **category**: Referencia válida a Category
- **subcategory**: Referencia válida a SubCategory (opcional)

#### Índices
- `slug`: Único
- `category`: Para consultas por categoría
- `subcategory`: Para consultas por subcategoría
- `isActive`: Para filtrar productos activos
- `name, description, brand`: Text search

#### Relaciones
- **Many-to-One**: Category
- **Many-to-One**: SubCategory (opcional)
- **One-to-Many**: Variants (embebidas)
- **One-to-Many**: OrderItems

---

### 3. Categories (Categorías)

#### Propósito
Organizar productos en categorías principales con soporte para imágenes.

#### Esquema
```typescript
interface ICategory {
  _id: ObjectId;
  name: string;                   // Requerido, único, trim
  slug: string;                   // Único, lowercase, URL-friendly
  image?: {
    url: string;
    public_id: string;
  };
  isActive: boolean;             // Estado de la categoría
  createdAt: Date;
  updatedAt: Date;
}
```

#### Validaciones
- **name**: Requerido, único, trim
- **slug**: Requerido, único, lowercase
- **isActive**: Boolean, default true

#### Índices
- `name`: Único
- `slug`: Único
- `isActive`: Para filtrar categorías activas

#### Relaciones
- **One-to-Many**: Products
- **One-to-Many**: SubCategories

---

### 4. SubCategories (Subcategorías)

#### Propósito
Crear subclasificaciones dentro de las categorías principales.

#### Esquema
```typescript
interface ISubCategory {
  _id: ObjectId;
  name: string;                   // Requerido, trim
  slug: string;                   // Único, lowercase, URL-friendly
  category: ObjectId;             // Referencia a Category
  isActive: boolean;             // Estado de la subcategoría
  createdAt: Date;
  updatedAt: Date;
}
```

#### Validaciones
- **name**: Requerido, trim
- **slug**: Requerido, único, lowercase
- **category**: Referencia válida a Category
- **isActive**: Boolean, default true

#### Índices
- `slug`: Único
- `category`: Para consultas por categoría padre
- `isActive`: Para filtrar subcategorías activas

#### Relaciones
- **Many-to-One**: Category
- **One-to-Many**: Products

---

### 5. Orders (Pedidos)

#### Propósito
Gestionar pedidos de compra con integración de pagos y seguimiento de estados.

#### Esquema
```typescript
interface IOrder {
  _id: ObjectId;
  invoiceNumber: number;          // Número de factura único
  user?: ObjectId;                // Referencia a User (opcional)
  email: string;                  // Email del cliente
  items: Array<IOrderItem>;       // Productos del pedido
  amount: number;                 // Total del pedido
  currency: string;                 // Moneda (CRC, USD)
  shippingAddress: IAddress;      // Dirección de envío
  paymentProvider: string;        // Proveedor de pago (onvo)
  paymentIntentId?: string;       // ID de intención de pago
  status: OrderStatus;            // Estado del pedido
  createdAt: Date;
  updatedAt: Date;
}

interface IOrderItem {
  product: ObjectId;              // Referencia a Product
  name: string;                   // Snapshot del nombre
  variant?: {
    _id: ObjectId;
    size?: string;
    color?: string;
    sku?: string;
  };
  unitPrice: number;              // Precio unitario al momento de compra
  quantity: number;               // Cantidad comprada
  image?: string;                  // URL de imagen principal
}

interface IAddress {
  fullName: string;                // Nombre completo
  phone?: string;                  // Teléfono
  line1: string;                  // Dirección línea 1
  line2?: string;                 // Dirección línea 2
  city: string;                   // Ciudad
  state?: string;                  // Estado/Provincia
  zip?: string;                   // Código postal
  country: string;                // País
}

type OrderStatus = 
  | "pending"      // Pendiente de pago
  | "paid"         // Pagado
  | "failed"       // Pago fallido
  | "processing"   // En procesamiento
  | "shipped"      // Enviado
  | "delivered"    // Entregado
  | "cancelled";   // Cancelado
```

#### Validaciones
- **invoiceNumber**: Requerido, único, auto-incremental
- **email**: Requerido, formato email válido
- **amount**: Requerido, número positivo
- **currency**: String, default "CRC"
- **status**: Enum de estados válidos
- **shippingAddress**: Objeto con campos requeridos

#### Índices
- `invoiceNumber`: Único
- `email`: Para consultas por cliente
- `status`: Para filtrar por estado
- `createdAt`: Para ordenamiento temporal
- `paymentIntentId`: Para webhooks de pago

#### Relaciones
- **Many-to-One**: User (opcional)
- **One-to-Many**: OrderItems (embebidas)
- **Many-to-One**: Products (a través de OrderItems)

---

### 6. Contacts (Contactos)

#### Propósito
Gestionar mensajes de contacto de clientes con seguimiento de estado.

#### Esquema
```typescript
interface IContact {
  _id: ObjectId;
  name: string;                   // Nombre del contacto
  email: string;                  // Email del contacto
  subject: string;                // Asunto del mensaje
  message: string;                // Contenido del mensaje
  status: "pending" | "reviewed"; // Estado del mensaje
  createdAt: Date;               // Solo timestamp de creación
}
```

#### Validaciones
- **name**: Requerido, trim
- **email**: Requerido, formato email válido
- **subject**: Requerido, máximo 120 caracteres
- **message**: Requerido, máximo 1000 caracteres
- **status**: Enum ["pending", "reviewed"], default "pending"

#### Índices
- `email`: Para consultas por contacto
- `status`: Para filtrar por estado
- `createdAt`: Para ordenamiento temporal

#### Relaciones
- **Sin relaciones**: Entidad independiente

---

## Relaciones entre Colecciones

### Diagrama de Relaciones
```
Users (1) ──── (0..*) Orders
                │
                └─── (1..*) OrderItems ──── (1) Products
                                    │
                                    └─── (0..1) Variants

Products (N) ──── (1) Categories
Products (N) ──── (0..1) SubCategories

SubCategories (N) ──── (1) Categories

Contacts (independiente)
```

### Tipos de Relaciones

#### 1. Relaciones Referenciales (Foreign Keys)
- **Products → Categories**: `product.category` → `category._id`
- **Products → SubCategories**: `product.subcategory` → `subcategory._id`
- **SubCategories → Categories**: `subcategory.category` → `category._id`
- **Orders → Users**: `order.user` → `user._id` (opcional)
- **OrderItems → Products**: `orderItem.product` → `product._id`

#### 2. Relaciones Embebidas
- **Products → Variants**: `product.variants[]` (array embebido)
- **Orders → OrderItems**: `order.items[]` (array embebido)
- **Products → Images**: `product.images[]` (array embebido)
- **Variants → Images**: `variant.images[]` (array embebido)

#### 3. Referencias Virtuales
- **Orders → Products**: A través de `order.items[].product`
- **Categories → Products**: A través de `product.category`

## Validaciones y Restricciones

### Validaciones de Integridad
1. **Referencias válidas**: Todas las foreign keys deben apuntar a documentos existentes
2. **Unicidad**: Campos únicos no pueden duplicarse
3. **Formato de datos**: Emails, URLs, fechas en formato correcto
4. **Rangos**: Números positivos, longitudes de texto apropiadas

### Validaciones de Negocio
1. **Stock no negativo**: Productos y variantes no pueden tener stock negativo
2. **Precios positivos**: Todos los precios deben ser números positivos
3. **Estados válidos**: Solo estados predefinidos en enums
4. **Categorías activas**: Solo productos de categorías activas pueden estar activos

### Restricciones de Integridad Referencial
1. **Cascada de eliminación**: Al eliminar una categoría, verificar productos asociados
2. **Orfanos**: Evitar referencias a documentos eliminados
3. **Consistencia**: Mantener consistencia entre documentos relacionados

## Índices y Optimización

### Índices Principales
```javascript
// Users
db.users.createIndex({ "email": 1 }, { unique: true })
db.users.createIndex({ "role": 1 })

// Products
db.products.createIndex({ "slug": 1 }, { unique: true })
db.products.createIndex({ "category": 1 })
db.products.createIndex({ "subcategory": 1 })
db.products.createIndex({ "isActive": 1 })
db.products.createIndex({ "name": "text", "description": "text", "brand": "text" })

// Categories
db.categories.createIndex({ "name": 1 }, { unique: true })
db.categories.createIndex({ "slug": 1 }, { unique: true })
db.categories.createIndex({ "isActive": 1 })

// SubCategories
db.subcategories.createIndex({ "slug": 1 }, { unique: true })
db.subcategories.createIndex({ "category": 1 })
db.subcategories.createIndex({ "isActive": 1 })

// Orders
db.orders.createIndex({ "invoiceNumber": 1 }, { unique: true })
db.orders.createIndex({ "email": 1 })
db.orders.createIndex({ "status": 1 })
db.orders.createIndex({ "createdAt": -1 })
db.orders.createIndex({ "paymentIntentId": 1 })

// Contacts
db.contacts.createIndex({ "email": 1 })
db.contacts.createIndex({ "status": 1 })
db.contacts.createIndex({ "createdAt": -1 })
```

### Consultas Optimizadas
1. **Búsqueda de productos**: Índice de texto completo
2. **Filtrado por categoría**: Índice en campo category
3. **Pedidos por fecha**: Índice en createdAt descendente
4. **Productos activos**: Índice en isActive
5. **Webhooks de pago**: Índice en paymentIntentId

## Migraciones y Versionado

### Estrategia de Migración
1. **Backward compatibility**: Mantener compatibilidad con versiones anteriores
2. **Migración gradual**: Actualizar documentos de forma incremental
3. **Validación**: Verificar integridad después de migraciones
4. **Rollback**: Plan de reversión en caso de problemas

### Versionado de Esquemas
1. **Versionado semántico**: Mayor.Minor.Patch
2. **Documentación de cambios**: Registro de modificaciones
3. **Scripts de migración**: Automatización de cambios
4. **Testing**: Pruebas de migración en entornos de desarrollo

## Consideraciones de Rendimiento

### Optimizaciones
1. **Índices compuestos**: Para consultas complejas
2. **Proyección**: Seleccionar solo campos necesarios
3. **Paginación**: Limitar resultados de consultas
4. **Agregación**: Usar pipeline de agregación para consultas complejas

### Monitoreo
1. **Métricas de consulta**: Tiempo de ejecución, uso de índices
2. **Tamaño de colecciones**: Monitoreo de crecimiento
3. **Rendimiento**: Análisis de consultas lentas
4. **Recursos**: Uso de CPU, memoria, disco

## Seguridad de Datos

### Protección de Información Sensible
1. **Encriptación**: Contraseñas hasheadas con bcrypt
2. **Validación**: Sanitización de inputs
3. **Autorización**: Control de acceso por roles
4. **Auditoría**: Log de cambios importantes

### Cumplimiento
1. **GDPR**: Manejo de datos personales
2. **Retención**: Políticas de eliminación de datos
3. **Backup**: Respaldo regular de datos
4. **Recuperación**: Plan de recuperación ante desastres
