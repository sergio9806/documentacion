# Diagrama de Caso de Uso: Contacto

## 1. Enviar Mensaje de Contacto

```mermaid
sequenceDiagram
    actor Cliente as Cliente
    participant ContactPage as ContactPage.tsx
    participant ContactService as contact.service
    participant Backend as Backend API
    participant Swal as SweetAlert2
    participant Router as React Router
    
    Cliente->>ContactPage: Navegar a /contact
    ContactPage-->>Cliente: Mostrar formulario
    
    Cliente->>ContactPage: Llenar campos
    Note over Cliente,ContactPage: Nombre, Email, Asunto, Mensaje
    
    Cliente->>ContactPage: Click Enviar mensaje
    ContactPage->>ContactPage: Validar formulario
    
    alt Validacion exitosa
        ContactPage->>ContactService: sendMessage(values)
        ContactService->>Backend: POST /api/contacts
        
        Backend->>Backend: Guardar mensaje
        Backend->>Backend: Socket.IO emit new_contact
        Backend-->>ContactService: Success response
        ContactService-->>ContactPage: Success
        
        ContactPage->>Swal: Mostrar success
        Swal-->>Cliente: Mensaje enviado correctamente
        
        ContactPage->>ContactPage: form.resetFields()
        ContactPage->>Router: navigate(/)
        Router-->>Cliente: Redirigir a Home
    else Validacion fallida
        ContactPage-->>Cliente: Mostrar errores campos
    else Error servidor
        Backend-->>ContactService: Error
        ContactService-->>ContactPage: Error
        ContactPage->>Swal: Mostrar error
        Swal-->>Cliente: Error al enviar mensaje
    end
```

## Validaciones del Formulario

```mermaid
graph TD
    A[Formulario Contacto] --> B[Campo Nombre]
    B --> B1{¿Requerido?}
    B1 -->|No| B2[Error: Ingresa nombre]
    B1 -->|Si| B3[OK]
    
    A --> C[Campo Email]
    C --> C1{¿Requerido?}
    C1 -->|No| C2[Error: Ingresa correo]
    C1 -->|Si| C3{¿Formato email?}
    C3 -->|No| C4[Error: Correo invalido]
    C3 -->|Si| C5[OK]
    
    A --> D[Campo Asunto]
    D --> D1{¿Requerido?}
    D1 -->|No| D2[Error: Ingresa asunto]
    D1 -->|Si| D3[OK]
    
    A --> E[Campo Mensaje]
    E --> E1{¿Requerido?}
    E1 -->|No| E2[Error: Ingresa mensaje]
    E1 -->|Si| E3{¿Min 10 caracteres?}
    E3 -->|No| E4[Error: Minimo 10 caracteres]
    E3 -->|Si| E5[OK]
    
    B3 --> F{Todos validos?}
    C5 --> F
    D3 --> F
    E5 --> F
    
    F -->|Si| G[Submit habilitado]
    F -->|No| H[Submit deshabilitado]
```

## Flujo Completo de Contacto

```mermaid
flowchart TD
    A[Home Page] --> B[Click Contacto]
    B --> C[Contact Page]
    C --> D[Mostrar Formulario]
    
    D --> E[Cliente llena datos]
    E --> F[Click Enviar]
    F --> G{¿Valido?}
    
    G -->|No| H[Mostrar errores]
    H --> E
    
    G -->|Si| I[Enviar al backend]
    I --> J[Guardar mensaje]
    J --> K[Emitir socket event]
    
    K --> L{Respuesta?}
    L -->|Success| M[Swal Success]
    L -->|Error| N[Swal Error]
    
    M --> O[Reset formulario]
    O --> P[Navigate Home]
    
    N --> Q[Reintentar]
    Q --> E
```

## Estructura de ContactPage

```mermaid
graph TD
    A[ContactPage] --> B[Header]
    B --> C[Titulo: Contactanos]
    B --> D[Subtitulo: Dudas y consultas]
    
    A --> E[Form Vertical]
    E --> F[Nombre completo - Input]
    E --> G[Email - Input]
    E --> H[Asunto - Input]
    E --> I[Mensaje - TextArea 5 rows]
    
    A --> J[Footer]
    J --> K[Boton Enviar mensaje]
    K --> L[Loading state]
    K --> M[Submit action]
```

## Notificación en Tiempo Real

```mermaid
sequenceDiagram
    actor Cliente as Cliente Web
    participant Backend as Backend API
    participant Socket as Socket.IO
    participant AdminPanel as Admin Panel
    
    Cliente->>Backend: POST /api/contacts
    Backend->>Backend: Guardar mensaje pendiente
    Backend->>Socket: emit("new_contact", contact)
    
    Socket-->>AdminPanel: Evento recibido
    AdminPanel->>AdminPanel: Mostrar notification
    Note over AdminPanel: Nombre del remitente
    AdminPanel->>AdminPanel: Incrementar contador
```

## Confirmación de Envío

```mermaid
sequenceDiagram
    actor Cliente as Cliente
    participant ContactPage as ContactPage.tsx
    participant Swal as SweetAlert2
    
    ContactPage->>Swal: Mostrar confirmacion
    Swal-->>Cliente: Dialog Exitoso
    Note over Cliente: Titulo: Mensaje enviado
    Note over Cliente: Texto: Gracias por contactarnos
    Note over Cliente: Boton: Volver al inicio
    
    Cliente->>Swal: Click boton
    Swal-->>ContactPage: Cerrar dialog
    ContactPage->>ContactPage: Reset form
    ContactPage->>ContactPage: Navigate home
```

## Estructura de Datos del Mensaje

```mermaid
graph TD
    A[Contact Message] --> B[Informacion Personal]
    B --> B1[name: string]
    B --> B2[email: string]
    
    A --> C[Mensaje]
    C --> C1[subject: string]
    C --> C2[message: string - min 10 chars]
    
    A --> D[Metadata]
    D --> D1[status: pending - default]
    D --> D2[createdAt: Date]
    D --> D3[reviewedAt?: Date]
```

## Integración con Backend

```mermaid
graph LR
    A[ContactService] --> B[POST /api/contacts]
    B --> C[Backend Controller]
    C --> D[Validar datos]
    D --> E[Guardar en DB]
    E --> F[Emitir Socket.IO]
    F --> G[Admin Notificado]
    G --> H[Return success]
    H --> I[Frontend actualizado]
```

## Flujo de Errores

```mermaid
stateDiagram-v2
    [*] --> Enviando: Submit form
    Enviando --> Exito: Respuesta 200
    Enviando --> ErrorValidacion: Error 400
    Enviando --> ErrorServidor: Error 500
    
    Exito --> ResetForm: Reset campos
    ResetForm --> RedirigirHome: Navegar
    RedirigirHome --> [*]
    
    ErrorValidacion --> MostrarErrores: Errores form
    MostrarErrores --> UsuarioCorrige: Cliente corrige
    UsuarioCorrige --> Enviando
    
    ErrorServidor --> MostrarErrorServidor: Swal error
    MostrarErrorServidor --> Reintentar: Cliente reintenta
    Reintentar --> Enviando
```

