# Diagrama de Caso de Uso: Gestión de Subcategorías

## 1. Listar Subcategorías

```mermaid
sequenceDiagram
    actor Admin as Administrador
    participant SubCategoriesPage as SubCategoriesPage.tsx
    participant ReactQuery as React Query
    participant SubCategoryService as subcategory.service.ts
    participant Backend as Backend API
    
    Admin->>SubCategoriesPage: Navegar a /admin/subcategories
    SubCategoriesPage->>ReactQuery: useQuery(["subcategories"])
    ReactQuery->>SubCategoryService: getAll()
    SubCategoryService->>Backend: GET /api/subcategories?populate=category
    
    Backend-->>SubCategoryService: [SubCategory con category]
    SubCategoryService-->>ReactQuery: SubCategories[]
    ReactQuery-->>SubCategoriesPage: Datos cargados
    
    SubCategoriesPage->>SubCategoriesPage: Separar activas e inactivas
    SubCategoriesPage->>Admin: Mostrar Tabs (Activas/Inactivas) con contadores
```

## 2. Crear Subcategoría

```mermaid
sequenceDiagram
    actor Admin as Administrador
    participant SubCategoriesPage as SubCategoriesPage.tsx
    participant Modal as Antd Modal
    participant Form as Antd Form
    participant ReactQuery as React Query
    participant CategoryService as category.service
    participant UseSlug as useSlug Hook
    participant SubCategoryService as subcategory.service.ts
    participant Backend as Backend API
    
    Admin->>SubCategoriesPage: Click "Agregar Subcategoría"
    SubCategoriesPage->>ReactQuery: Cargar categorías
    ReactQuery->>CategoryService: getAll()
    CategoryService->>Backend: GET /api/categories
    Backend-->>SubCategoriesPage: Categories[]
    
    SubCategoriesPage->>Modal: Abrir modal
    Modal-->>Admin: Mostrar formulario
    
    Admin->>Form: Ingresar nombre
    Form->>UseSlug: generateSlug(nombre)
    UseSlug->>UseSlug: Convertir a slug
    UseSlug-->>Form: slug generado (disabled)
    
    Admin->>Form: Seleccionar categoría padre
    Form->>Form: Guardar category._id
    
    Admin->>Modal: Click "Guardar"
    Modal->>Form: validateFields()
    Form->>SubCategoryService: create({name, slug, category})
    
    SubCategoryService->>Backend: POST /api/subcategories
    Backend-->>SubCategoryService: SubCategory creada
    SubCategoryService-->>SubCategoriesPage: Success
    SubCategoriesPage->>Swal: Mostrar "Subcategoría creada"
    SubCategoriesPage->>ReactQuery: invalidateQueries(["subcategories"])
    SubCategoriesPage->>Form: resetFields() y cerrar modal
```

## 3. Editar Subcategoría

```mermaid
sequenceDiagram
    actor Admin as Administrador
    participant SubCategoryTable as SubCategoryTable.tsx
    participant SubCategoriesPage as SubCategoriesPage.tsx
    participant Modal as Antd Modal
    participant Form as Antd Form
    participant ReactQuery as React Query
    participant SubCategoryService as subcategory.service.ts
    participant Backend as Backend API
    
    Admin->>SubCategoryTable: Click "Editar" en subcategoría
    SubCategoryTable->>SubCategoriesPage: handleEdit(subcategory)
    SubCategoriesPage->>ReactQuery: Cargar categorías disponibles
    ReactQuery->>CategoryService: getAll()
    
    SubCategoriesPage->>Modal: Abrir modal
    SubCategoriesPage->>Form: setFieldsValue({name, category: category._id})
    SubCategoriesPage->>UseSlug: setSlug(subcategory.slug)
    
    Modal-->>Admin: Mostrar formulario con datos
    
    Admin->>Form: Modificar campos
    Admin->>Modal: Click "Guardar"
    Modal->>Form: validateFields()
    Form->>SubCategoryService: update(id, {name, slug, category})
    
    SubCategoryService->>Backend: PUT /api/subcategories/:id
    Backend-->>SubCategoryService: SubCategory actualizada
    SubCategoryService-->>SubCategoriesPage: Success
    SubCategoriesPage->>Swal: Mostrar "Subcategoría actualizada"
    SubCategoriesPage->>ReactQuery: invalidateQueries(["subcategories"])
    SubCategoriesPage->>Form: resetFields() y cerrar modal
```

## 4. Activar/Desactivar Subcategoría

```mermaid
sequenceDiagram
    actor Admin as Administrador
    participant SubCategoryTable as SubCategoryTable.tsx
    participant SubCategoriesPage as SubCategoriesPage.tsx
    participant SubCategoryService as subcategory.service.ts
    participant Backend as Backend API
    participant ReactQuery as React Query
    
    Admin->>SubCategoryTable: Click toggle (Activar/Desactivar)
    SubCategoryTable->>SubCategoriesPage: handleToggle(id)
    SubCategoriesPage->>SubCategoryService: toggleActive(id)
    SubCategoryService->>Backend: PATCH /api/subcategories/:id/toggle
    
    Backend-->>SubCategoryService: isActive actualizado
    SubCategoryService-->>SubCategoriesPage: Success
    SubCategoriesPage->>ReactQuery: invalidateQueries(["subcategories"])
    ReactQuery-->>SubCategoriesPage: Refrescar lista de subcategorías
```

## 5. Eliminar Subcategoría en Cascada

```mermaid
sequenceDiagram
    actor Admin as Administrador
    participant SubCategoriesPage as SubCategoriesPage.tsx
    participant Swal as SweetAlert2
    participant SubCategoryService as subcategory.service.ts
    participant Backend as Backend API
    participant ReactQuery as React Query
    
    Admin->>SubCategoriesPage: Click "Eliminar" en subcategoría inactiva
    SubCategoriesPage->>Swal: Confirmación de eliminación
    Swal-->>Admin: ¿Eliminar subcategoría? (Eliminará productos asociados)
    
    alt Usuario confirma
        Admin->>Swal: Sí, eliminar
        SubCategoriesPage->>SubCategoryService: removeCascade(id)
        SubCategoryService->>Backend: DELETE /api/subcategories/:id/cascade
        
        Backend->>Backend: Eliminar subcategoría
        Backend->>Backend: Eliminar productos asociados
        
        Backend-->>SubCategoryService: Cascada completada
        SubCategoryService-->>SubCategoriesPage: Success
        SubCategoriesPage->>Swal: Mostrar "Subcategoría eliminada junto a productos"
        SubCategoriesPage->>ReactQuery: invalidateQueries(["subcategories"])
    else Usuario cancela
        Admin->>Swal: Cancelar
        Swal-->>SubCategoriesPage: Operación cancelada
    end
```

## Relación Categoría-Subcategoría

```mermaid
graph TD
    A[Categoría] -->|Padre| B[Subcategoría 1]
    A -->|Padre| C[Subcategoría 2]
    A -->|Padre| D[Subcategoría 3]
    
    B -->|Pertenece| E[Productos de Sub 1]
    C -->|Pertenece| F[Productos de Sub 2]
    D -->|Pertenece| G[Productos de Sub 3]
    
    style A fill:#e1f5ff
    style B fill:#fff3e0
    style C fill:#fff3e0
    style D fill:#fff3e0
```

## Vista de Subcategorías

```mermaid
graph TD
    A[Subcategorías Page] --> B[Header: Subcategorías + Botón Agregar]
    B --> C[Tabs]
    
    C --> D[Tab Activas]
    D --> E[Tag Verde con contador]
    E --> F[SubCategoryTable - Solo activas]
    
    C --> G[Tab Inactivas]
    G --> H[Tag Rojo con contador]
    H --> I[SubCategoryTable - Solo inactivas]
    
    F --> J[Editar | Desactivar]
    I --> K[Editar | Activar | Eliminar Cascada]
```

## Tabla de Subcategorías

```mermaid
graph LR
    A[SubCategoryTable] --> B[Columna: Nombre]
    A --> C[Columna: Slug]
    A --> D[Columna: Categoría Padre]
    A --> E[Columna: Estado]
    A --> F[Columna: Fecha]
    A --> G[Columna: Acciones]
    
    G --> H{Modo}
    H -->|Active| I[Editar + Toggle Off]
    H -->|Inactive| J[Editar + Toggle On + Eliminar]
```

