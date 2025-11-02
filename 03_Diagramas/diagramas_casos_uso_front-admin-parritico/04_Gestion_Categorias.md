# Diagrama de Caso de Uso: Gestión de Categorías

## 1. Listar Categorías

```mermaid
sequenceDiagram
    actor Admin as Administrador
    participant CategoriesPage as CategoriesPage.tsx
    participant ReactQuery as React Query
    participant CategoryService as category.service.ts
    participant Backend as Backend API
    
    Admin->>CategoriesPage: Navegar a /admin/categories
    CategoriesPage->>ReactQuery: useQuery(["categories"])
    ReactQuery->>CategoryService: getAll()
    CategoryService->>Backend: GET /api/categories
    
    Backend-->>CategoryService: [Category]
    CategoryService-->>ReactQuery: Categories[]
    ReactQuery-->>CategoriesPage: Datos cargados
    
    CategoriesPage->>CategoriesPage: Separar activas e inactivas
    CategoriesPage->>Admin: Mostrar Tabs (Activas/Inactivas) con contadores
```

## 2. Crear Categoría

```mermaid
sequenceDiagram
    actor Admin as Administrador
    participant CategoriesPage as CategoriesPage.tsx
    participant Modal as Antd Modal
    participant Form as Antd Form
    participant UseSlug as useSlug Hook
    participant CategoryService as category.service.ts
    participant Backend as Backend API
    participant ReactQuery as React Query
    
    Admin->>CategoriesPage: Click "Agregar Categoría"
    CategoriesPage->>Modal: Abrir modal
    Modal-->>Admin: Mostrar formulario
    
    Admin->>Form: Ingresar nombre
    Form->>UseSlug: generateSlug(nombre)
    UseSlug->>UseSlug: Convertir a slug (lowercase, sin espacios)
    UseSlug-->>Form: slug generado (disabled)
    
    Admin->>Form: Subir imagen
    Form->>Form: Guardar en fileList
    
    Admin->>Modal: Click "Guardar"
    Modal->>Form: validateFields()
    Form->>CategoryService: create(formData)
    
    Note over CategoryService: FormData: name, slug, image
    CategoryService->>Backend: POST /api/categories (multipart/form-data)
    
    Backend-->>CategoryService: Category creada
    CategoryService-->>CategoriesPage: Success
    CategoriesPage->>Swal: Mostrar "Categoría creada"
    CategoriesPage->>ReactQuery: invalidateQueries(["categories"])
    CategoriesPage->>Form: resetFields() y cerrar modal
```

## 3. Editar Categoría

```mermaid
sequenceDiagram
    actor Admin as Administrador
    participant CategoryTable as CategoryTable.tsx
    participant CategoriesPage as CategoriesPage.tsx
    participant Modal as Antd Modal
    participant Form as Antd Form
    participant CategoryService as category.service.ts
    participant Backend as Backend API
    participant ReactQuery as React Query
    
    Admin->>CategoryTable: Click "Editar" en categoría
    CategoryTable->>CategoriesPage: handleEdit(category)
    CategoriesPage->>Modal: Abrir modal
    CategoriesPage->>Form: setFieldsValue({name: category.name})
    CategoriesPage->>UseSlug: setSlug(category.slug)
    CategoriesPage->>Form: setFileList([category.image])
    
    Modal-->>Admin: Mostrar formulario con datos
    
    Admin->>Form: Modificar campos
    Admin->>Modal: Click "Guardar"
    Modal->>Form: validateFields()
    Form->>CategoryService: update(id, formData)
    
    Note over CategoryService: FormData: name, slug, image (si cambió)
    CategoryService->>Backend: PUT /api/categories/:id (multipart/form-data)
    
    Backend-->>CategoryService: Category actualizada
    CategoryService-->>CategoriesPage: Success
    CategoriesPage->>Swal: Mostrar "Categoría actualizada"
    CategoriesPage->>ReactQuery: invalidateQueries(["categories"])
    CategoriesPage->>Form: resetFields() y cerrar modal
```

## 4. Activar/Desactivar Categoría

```mermaid
sequenceDiagram
    actor Admin as Administrador
    participant CategoryTable as CategoryTable.tsx
    participant CategoriesPage as CategoriesPage.tsx
    participant CategoryService as category.service.ts
    participant Backend as Backend API
    participant ReactQuery as React Query
    
    Admin->>CategoryTable: Click toggle (Activar/Desactivar)
    CategoryTable->>CategoriesPage: handleToggle(id)
    CategoriesPage->>CategoryService: toggleActive(id)
    CategoryService->>Backend: PATCH /api/categories/:id/toggle
    
    Backend-->>CategoryService: isActive actualizado
    CategoryService-->>CategoriesPage: Success
    CategoriesPage->>ReactQuery: invalidateQueries(["categories"])
    ReactQuery-->>CategoriesPage: Refrescar lista de categorías
```

## 5. Eliminar Categoría en Cascada

```mermaid
sequenceDiagram
    actor Admin as Administrador
    participant CategoriesPage as CategoriesPage.tsx
    participant Swal as SweetAlert2
    participant CategoryService as category.service.ts
    participant Backend as Backend API
    participant ReactQuery as React Query
    
    Admin->>CategoriesPage: Click "Eliminar" en categoría inactiva
    CategoriesPage->>Swal: Confirmación de eliminación
    Swal-->>Admin: ¿Eliminar categoría? (Eliminará productos asociados)
    
    alt Usuario confirma
        Admin->>Swal: Sí, eliminar
        CategoriesPage->>CategoryService: removeCascade(id)
        CategoryService->>Backend: DELETE /api/categories/:id/cascade
        
        Backend->>Backend: Eliminar categoría
        Backend->>Backend: Eliminar subcategorías asociadas
        Backend->>Backend: Eliminar productos asociados
        
        Backend-->>CategoryService: Cascada completada
        CategoryService-->>CategoriesPage: Success
        CategoriesPage->>Swal: Mostrar "Categoría eliminada en cascada"
        CategoriesPage->>ReactQuery: invalidateQueries(["categories"])
    else Usuario cancela
        Admin->>Swal: Cancelar
        Swal-->>CategoriesPage: Operación cancelada
    end
```

## Vista de Categorías

```mermaid
graph TD
    A[Categorías Page] --> B[Header: Categorías + Botón Agregar]
    B --> C[Tabs]
    
    C --> D[Tab Activas]
    D --> E[Tag Verde con contador]
    E --> F[CategoryTable - Solo activas]
    
    C --> G[Tab Inactivas]
    G --> H[Tag Rojo con contador]
    H --> I[CategoryTable - Solo inactivas]
    
    F --> J[Editar | Desactivar]
    I --> K[Editar | Activar | Eliminar Cascada]
```

## Tabla de Categorías

```mermaid
graph LR
    A[CategoryTable] --> B[Columna: Imagen]
    A --> C[Columna: Nombre]
    A --> D[Columna: Slug]
    A --> E[Columna: Estado]
    A --> F[Columna: Fecha]
    A --> G[Columna: Acciones]
    
    G --> H{Modo}
    H -->|Active| I[Editar + Toggle Off]
    H -->|Inactive| J[Editar + Toggle On + Eliminar]
```

