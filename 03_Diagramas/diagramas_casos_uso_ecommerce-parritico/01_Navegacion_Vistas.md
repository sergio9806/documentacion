# Diagrama de Caso de Uso: Navegación y Vistas

## 1. Página de Inicio (Home)

```mermaid
sequenceDiagram
    actor Cliente as Cliente
    participant Home as HomePage.tsx
    participant CategoryService as category.service
    participant ProductService as product.service
    participant Backend as Backend API
    
    Cliente->>Home: Navegar a /
    Home->>Home: Cargar datos
    
    par Cargar simultaneamente
        Home->>CategoryService: getActive()
        CategoryService->>Backend: GET /api/categories/active
        Backend-->>CategoryService: Categories[]
        CategoryService-->>Home: Primeras 4 categorias
    and
        Home->>ProductService: getAll({ sort: newest })
        ProductService->>Backend: GET /api/products
        Backend-->>ProductService: Products[]
        ProductService-->>Home: Productos
    end
    
    Home->>Home: Filtrar productos activos
    Home->>Home: Mostrar primeras 8
    Home-->>Cliente: Carousel Banner + Categorias + Productos
```

## 2. Vista de Productos

```mermaid
sequenceDiagram
    actor Cliente as Cliente
    participant ProductsPage as ProductsPage.tsx
    participant CategoryService as category.service
    participant ProductService as product.service
    participant Backend as Backend API
    
    Cliente->>ProductsPage: Navegar a /products
    ProductsPage->>CategoryService: getActive()
    CategoryService->>Backend: GET /api/categories/active
    Backend-->>ProductsPage: Categories[]
    
    ProductsPage->>ProductService: getAll({ sort: newest })
    ProductService->>Backend: GET /api/products
    Backend-->>ProductsPage: Products[]
    
    ProductsPage->>ProductsPage: Filtrar activos
    ProductsPage-->>Cliente: Lista de productos con filtros
```

## 3. Aplicar Filtros de Productos

```mermaid
sequenceDiagram
    actor Cliente as Cliente
    participant ProductsPage as ProductsPage.tsx
    participant ProductService as product.service
    participant Backend as Backend API
    participant Router as React Router
    
    Cliente->>ProductsPage: Modificar filtros
    Note over Cliente,ProductsPage: Busqueda, Categoria, Orden, Precio
    
    Cliente->>ProductsPage: Click Aplicar filtros
    ProductsPage->>ProductService: getAll({ sort, category, min, max, q })
    ProductService->>Backend: GET /api/products con query params
    
    Backend-->>ProductService: Products filtrados[]
    ProductService-->>ProductsPage: Resultados
    
    ProductsPage->>Router: Actualizar URL con params
    Router-->>Cliente: URL actualizada
    
    ProductsPage-->>Cliente: Mostrar productos filtrados
```

## 4. Vista de Detalle de Producto

```mermaid
sequenceDiagram
    actor Cliente as Cliente
    participant ProductDetail as ProductDetailPage.tsx
    participant ProductService as product.service
    participant Backend as Backend API
    
    Cliente->>ProductDetail: Click en producto
    Cliente->>ProductDetail: Navegar a /products/:slug
    
    ProductDetail->>ProductService: getBySlug(slug)
    ProductService->>Backend: GET /api/products/:slug
    
    Backend-->>ProductService: Product completo
    ProductService-->>ProductDetail: Product con variantes
    
    ProductDetail->>ProductDetail: Seleccionar primera variante disponible
    ProductDetail->>ProductDetail: Cargar imagenes principales y variantes
    
    ProductDetail-->>Cliente: Mostrar detalle completo
    Note over Cliente: Imagen, descripcion, precio, variantes, stock, cantidad
```

## 5. Cambiar Variante de Producto

```mermaid
sequenceDiagram
    actor Cliente as Cliente
    participant ProductDetail as ProductDetailPage.tsx
    
    Cliente->>ProductDetail: Seleccionar variante dropdown
    ProductDetail->>ProductDetail: handleVariantChange(variantId)
    ProductDetail->>ProductDetail: Buscar variante en producto
    ProductDetail->>ProductDetail: setSelectedVariant(variant)
    ProductDetail->>ProductDetail: setSelectedImage(variant.images)
    ProductDetail->>ProductDetail: setStockDisponible(variant.stock)
    ProductDetail->>ProductDetail: resetQuantity(1)
    
    ProductDetail-->>Cliente: Mostrar nueva variante con precio y stock
```

## 6. Cambiar Imagen de Vista Previa

```mermaid
sequenceDiagram
    actor Cliente as Cliente
    participant ProductDetail as ProductDetailPage.tsx
    
    Cliente->>ProductDetail: Click en miniatura
    ProductDetail->>ProductDetail: setSelectedImage(img.url)
    
    alt Si es imagen de variante
        ProductDetail->>ProductDetail: Cambiar variante automaticamente
    end
    
    ProductDetail-->>Cliente: Actualizar imagen principal
```

## Estructura de Página Home

```mermaid
graph TD
    A[HomePage] --> B[Carousel Banner]
    B --> C[Imagen 1]
    B --> D[Imagen 2]
    
    A --> E[InfoMarquee]
    E --> F[Mensaje informativo]
    
    A --> G[Section Categorias]
    G --> H[Grid 4 Categorias]
    H --> I[Categoria Card]
    I --> J[Imagen Categoria]
    I --> K[Nombre Categoria]
    I --> L[Click -> Filtrar]
    
    A --> M[Section Productos]
    M --> N{¿Hay productos activos?}
    N -->|Si| O[Grid 8 Productos]
    N -->|No| P[Mensaje vacio]
    
    O --> Q[Product Card]
    Q --> R[Imagen Producto]
    Q --> S[Nombre Producto]
    Q --> T[Precio Base]
    Q --> U{Badge Agotado?}
    U -->|Stock 0| V[Mostrar Badge Rojo]
    U -->|Stock > 0| W[Sin Badge]
    Q --> X[Click -> Detalle]
```

## Estructura de ProductsPage

```mermaid
graph TD
    A[ProductsPage] --> B[Header: Nuestros Productos]
    
    A --> C[Seccion Filtros]
    C --> D[Input Busqueda]
    C --> E[Select Categoria]
    C --> F[Select Orden]
    C --> G[Slider Precio]
    C --> H[Btn Aplicar]
    C --> I{Btn Quitar?}
    I -->|Hay filtros activos| J[Mostrar]
    I -->|Sin filtros| K[Ocultar]
    
    A --> L{¿Resultados?}
    L -->|Si| M[Grid Productos]
    L -->|No| N[Mensaje No encontrados]
    
    M --> O[Product Card]
    O --> P[Imagen]
    O --> Q[Nombre]
    O --> R[Precio]
    O --> S{Badge Agotado?}
    S -->|Sin stock| T[Badge Rojo]
    S -->|Con stock| U[Sin Badge]
    O --> V[Click -> Detalle]
```

## Estructura de ProductDetailPage

```mermaid
graph TD
    A[ProductDetailPage] --> B[Navegacion: Volver]
    A --> C[Row Layout]
    
    C --> D[Col Imagenes]
    D --> E[Imagen Principal]
    E --> F{Badge Estado}
    F -->|Disponible| G[Tag Verde]
    F -->|Agotado| H[Tag Rojo]
    
    D --> I[Grid Miniaturas]
    I --> J{¿Es de variante?}
    J -->|Si| K[Cambiar variante auto]
    J -->|No| L[Mostrar solo]
    
    C --> M[Col Info]
    M --> N[Nombre]
    M --> O[Marca opcional]
    M --> P[Descripcion]
    M --> Q[Divider]
    M --> R[Precio Unidad]
    
    M --> S{¿Tiene variantes?}
    S -->|Si| T[Select Variantes]
    S -->|No| U[Sin select]
    
    T --> V{Stock variante?}
    V -->|> 0| W[Mostrar disponibles]
    V -->|0| X[Mostrar Agotado]
    
    M --> Y[Control Cantidad]
    Y --> Z[Boton Menos -]
    Y --> AA[Display Cantidad]
    Y --> AB[Boton Mas +]
    Y --> AC[Validar max stock]
    
    M --> AD[Total Calculado]
    M --> AE[Boton Agregar Carrito]
    AE --> AF{¿Disponible?}
    AF -->|Si| AG[Boton Activo]
    AF -->|No| AH[Boton Deshabilitado]
    
    M --> AI{Stock bajo?}
    AI -->|<= 5| AJ[Mostrar alerta fuego]
    AI -->|> 5| AK[Ocultar]
```

