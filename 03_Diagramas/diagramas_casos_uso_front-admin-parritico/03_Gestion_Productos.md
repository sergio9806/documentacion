# Diagrama de Caso de Uso: Gestión de Productos

## 1. Listar Productos

```mermaid
sequenceDiagram
    actor Admin as Administrador
    participant ProductsPage as ProductsPage.tsx
    participant ReactQuery as React Query
    participant ProductService as product.service.ts
    participant Backend as Backend API
    
    Admin->>ProductsPage: Navegar a /admin/products
    ProductsPage->>ReactQuery: useQuery(["products"])
    ReactQuery->>ProductService: getAll()
    ProductService->>Backend: GET /api/products
    
    Backend-->>ProductService: [Product]
    ProductService-->>ReactQuery: Products[]
    ReactQuery-->>ProductsPage: Datos cargados
    
    ProductsPage->>ProductsPage: Separar activos e inactivos
    ProductsPage->>Admin: Mostrar Tabs (Activos/Inactivos)
    ProductsPage->>Admin: Mostrar tabla con contadores
```

## 2. Crear Producto

```mermaid
sequenceDiagram
    actor Admin as Administrador
    participant ProductForm as ProductFormPage.tsx
    participant CategoryService as category.service
    participant SubCategoryService as subcategory.service
    participant ProductService as product.service.ts
    participant Backend as Backend API
    participant ReactQuery as React Query
    
    Admin->>ProductForm: Click "Agregar Producto"
    ProductForm->>ReactQuery: Cargar categorías
    ReactQuery->>CategoryService: getAll()
    CategoryService->>Backend: GET /api/categories
    Backend-->>ProductForm: Categories[]
    
    ProductForm->>ReactQuery: Cargar subcategorías
    ReactQuery->>SubCategoryService: getAll()
    SubCategoryService->>Backend: GET /api/subcategories
    Backend-->>ProductForm: SubCategories[]
    
    Admin->>ProductForm: Llenar formulario
    Note over Admin,ProductForm: name, slug, brand, description, basePrice, stock
    Note over Admin,ProductForm: category, subcategory, variants[], images[]
    
    Admin->>ProductForm: Guardar Producto
    ProductForm->>ProductForm: Validar formulario
    ProductForm->>ProductForm: Crear FormData
    ProductForm->>ProductService: create(formData)
    ProductService->>Backend: POST /api/products (multipart/form-data)
    
    Backend-->>ProductService: Product creado
    ProductService-->>ProductForm: Success
    ProductForm->>ReactQuery: invalidateQueries(["products"])
    ProductForm->>Admin: Redireccionar a /admin/products
```

## 3. Editar Producto

```mermaid
sequenceDiagram
    actor Admin as Administrador
    participant ProductForm as ProductFormPage.tsx
    participant ProductService as product.service.ts
    participant Backend as Backend API
    participant ReactQuery as React Query
    
    Admin->>ProductForm: Click "Editar" en producto
    ProductForm->>ReactQuery: useQuery(["product", id])
    ReactQuery->>ProductService: getById(id)
    ProductService->>Backend: GET /api/products/:id
    
    Backend-->>ProductService: Product con datos completos
    ProductService-->>ReactQuery: Product
    ReactQuery-->>ProductForm: Datos del producto
    
    ProductForm->>ProductForm: setFieldsValue() con datos
    ProductForm->>ProductForm: setFileList() con imágenes
    Note over ProductForm: Incluye imágenes principales y por variante
    
    Admin->>ProductForm: Modificar campos
    Admin->>ProductForm: Guardar cambios
    ProductForm->>ProductForm: Validar y crear FormData
    ProductForm->>ProductService: update(id, formData)
    ProductService->>Backend: PUT /api/products/:id (multipart/form-data)
    
    Backend-->>ProductService: Product actualizado
    ProductService-->>ProductForm: Success
    ProductForm->>ReactQuery: invalidateQueries(["products"])
    ProductForm->>Admin: Redireccionar a /admin/products
```

## 4. Activar/Desactivar Producto

```mermaid
sequenceDiagram
    actor Admin as Administrador
    participant ProductTable as ProductTable.tsx
    participant ProductsPage as ProductsPage.tsx
    participant ProductService as product.service.ts
    participant Backend as Backend API
    participant ReactQuery as React Query
    
    Admin->>ProductTable: Click toggle (Activar/Desactivar)
    ProductTable->>ProductsPage: handleToggle(id)
    ProductsPage->>ProductService: toggleActive(id)
    ProductService->>Backend: PATCH /api/products/:id/toggle
    
    Backend-->>ProductService: isActive actualizado
    ProductService-->>ProductsPage: Success
    ProductsPage->>ReactQuery: invalidateQueries(["products"])
    ReactQuery-->>ProductsPage: Refrescar lista de productos
```

## 5. Eliminar Producto (Permanente)

```mermaid
sequenceDiagram
    actor Admin as Administrador
    participant ProductsPage as ProductsPage.tsx
    participant Swal as SweetAlert2
    participant ProductService as product.service.ts
    participant Backend as Backend API
    participant ReactQuery as React Query
    
    Admin->>ProductsPage: Click "Eliminar" en producto inactivo
    ProductsPage->>Swal: Confirmación de eliminación
    Swal-->>Admin: ¿Eliminar producto?
    
    alt Usuario confirma
        Admin->>Swal: Sí, eliminar
        ProductsPage->>ProductService: remove(id)
        ProductService->>Backend: DELETE /api/products/:id
        
        Backend-->>ProductService: Product eliminado
        ProductService-->>ProductsPage: Success
        ProductsPage->>Swal: Mostrar éxito
        ProductsPage->>ReactQuery: invalidateQueries(["products"])
    else Usuario cancela
        Admin->>Swal: Cancelar
        Swal-->>ProductsPage: Operación cancelada
    end
```

## Flujo de Variantes

```mermaid
graph TD
    A[Agregar Variante] --> B[Sku, Size, Color, Stock, Price]
    B --> C[Subir hasta 3 imágenes por variante]
    C --> D[Guardar Variante]
    
    D --> E{¿Producto con variantes?}
    E -->|Sí| F[Mostrar precios por variante]
    E -->|No| G[Mostrar precio base]
```

