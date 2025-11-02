# Diagrama de Caso de Uso: Gestión de Usuarios

## 1. Crear Usuario (Solo Admin)

```mermaid
sequenceDiagram
    actor Admin as Administrador
    participant UserCreatePage as UserCreatePage.tsx
    participant Form as Antd Form
    participant UserService as user.service.ts
    participant Backend as Backend API
    participant Swal as SweetAlert2
    participant Navigate as Router
    
    Admin->>UserCreatePage: Navegar a /admin/users/new
    UserCreatePage-->>Admin: Mostrar formulario de registro
    
    Admin->>Form: Ingresar email
    Admin->>Form: Ingresar password (min 8 caracteres)
    Admin->>Form: Seleccionar rol (admin | organizer)
    
    Admin->>Form: Click "Crear usuario"
    Form->>Form: validateFields()
    
    alt Validación exitosa
        Form->>UserService: create({email, password, role})
        UserService->>Backend: POST /api/auth/register
        
        alt Usuario creado exitosamente
            Backend-->>UserService: User creado
            UserService-->>UserCreatePage: Success
            UserCreatePage->>Swal: Mostrar "Usuario creado correctamente"
            UserCreatePage->>Form: resetFields()
            UserCreatePage->>Navigate: Redireccionar a /admin
            Navigate-->>Admin: Volver al dashboard
        else Email ya existe u otro error
            Backend-->>UserService: Error 400/409
            UserService-->>UserCreatePage: Error
            UserCreatePage-->>Admin: Mostrar mensaje de error
        end
    else Validación fallida
        Form-->>Admin: Mostrar errores de campo
    end
```

## Restricciones de Acceso

```mermaid
graph TD
    A[Usuario intenta acceder a /admin/users/new] --> B{¿Role === admin?}
    
    B -->|Sí| C[Permitir acceso]
    B -->|No| D[Redireccionar a /admin]
    
    C --> E[Mostrar formulario de registro]
    
    D --> F{Mensaje}
    F --> G[No tiene permisos]
```

## Estructura de Usuario

```mermaid
graph TD
    A[User] --> B[Credenciales]
    B --> B1[Email - único]
    B --> B2[Password - hasheado]
    
    A --> C[Rol]
    C --> C1[admin]
    C --> C2[organizer]
    
    A --> D[Metadata]
    D --> D1[Fecha creación]
    D --> D2[Último acceso]
```

## Formulario de Registro

```mermaid
graph TD
    A[UserCreatePage] --> B[Card centrado]
    B --> C[Header con ícono]
    C --> C1[Título: Registrar Usuario]
    C --> C2[Subtítulo descriptivo]
    
    B --> D[Form]
    D --> E[Campo: Email]
    E --> E1[Validación: required + email]
    E --> E2[Ícono: MailOutlined]
    
    D --> F[Campo: Password]
    F --> F1[Validación: required + min 8]
    F --> F2[Ícono: LockOutlined]
    
    D --> G[Campo: Rol]
    G --> G1[Select]
    G1 --> G2[Option: Organizer - default]
    G1 --> G3[Option: Admin]
    
    B --> H[Footer]
    H --> I[Botón: Crear usuario]
    
    I --> J{Click}
    J --> K[Validar]
    K -->|OK| L[Enviar al backend]
    K -->|Error| M[Mostrar errores]
    
    L --> N{Respuesta}
    N -->|Success| O[Mostrar Swal + Redirect]
    N -->|Error| P[Mostrar error]
```

## Flujo de Permisos por Rol

```mermaid
stateDiagram-v2
    [*] --> LoggedIn
    
    LoggedIn --> Admin: role = admin
    LoggedIn --> Organizer: role = organizer
    
    Admin --> Dashboard
    Admin --> Products
    Admin --> Categories
    Admin --> Subcategories
    Admin --> Orders
    Admin --> Contacts
    Admin --> UsersNew: Crear usuario
    Admin --> Logout
    
    Organizer --> Dashboard
    Organizer --> Orders
    Organizer --> Contacts
    Organizer --> Logout
    
    UsersNew --> Dashboard: Después de crear
    
    note right of Admin
        Acceso completo a todas
        las funcionalidades
    end note
    
    note right of Organizer
        Acceso limitado a
        órdenes y contactos
    end note
```

## Validaciones de Formulario

```mermaid
graph TD
    A[Formulario de Usuario] --> B[Campo Email]
    B --> B1{¿Campo vacío?}
    B1 -->|Sí| B2[Error: Requerido]
    B1 -->|No| B3{¿Formato email válido?}
    B3 -->|No| B4[Error: Formato inválido]
    B3 -->|Sí| B5[✓ Válido]
    
    A --> C[Campo Password]
    C --> C1{¿Campo vacío?}
    C1 -->|Sí| C2[Error: Requerido]
    C1 -->|No| C3{¿Min 8 caracteres?}
    C3 -->|No| C4[Error: Mínimo 8 caracteres]
    C3 -->|Sí| C5[✓ Válido]
    
    A --> D[Campo Rol]
    D --> D1{¿Seleccionado?}
    D1 -->|No| D2[Error: Seleccione un rol]
    D1 -->|Sí| D3{¿Valor válido?}
    D3 -->|admin u organizer| D4[✓ Válido]
    D3 -->|No| D5[Error: Rol inválido]
    
    B5 --> E{Todos válidos?}
    C5 --> E
    D4 --> E
    
    E -->|Sí| F[Submit habilitado]
    E -->|No| G[Submit deshabilitado]
```

