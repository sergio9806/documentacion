# Documentación del Proyecto Parritico Store

## 📋 Índice General

Esta documentación proporciona una visión completa del proyecto Parritico Store, incluyendo metodología de desarrollo, arquitectura, modelos de datos y diagramas UML.

---

## 🏗️ Estructura del Proyecto

```
parritico/
├── backend-parritico/          # API Backend (Node.js + Express)
├── ecommerce-parritico/        # Frontend E-commerce (React)
├── front-admin-parritico/      # Panel de Administración (React)
└── docs/                       # Documentación del Proyecto
    ├── README.md              # Este archivo
    ├── 02_Metodologia.md      # Metodología Scrum
    ├── 03_Diagramas/           # Diagramas UML
    └── 04_Modelo_de_Datos.md   # Modelo de datos
```

---

## 📚 Documentación Disponible

### 1. 📊 Diagramas UML
**Ubicación**: `/docs/03_Diagramas/`

#### 1.1 Diagrama de Casos de Uso
- **Archivo**: `01_Diagrama_Casos_Uso.md`
- **Descripción**: Casos de uso principales del sistema
- **Actores**: Cliente, Administrador, Sistemas Externos
- **Funcionalidades**: Navegación, compra, administración

#### 1.2 Diagrama de Clases
- **Archivo**: `02_Diagrama_Clases.md`
- **Descripción**: Estructura de clases basada en modelos de datos
- **Entidades**: User, Product, Order, Category, Contact
- **Relaciones**: One-to-Many, Many-to-One, Embebidas

#### 1.3 Diagrama de Secuencia
- **Archivo**: `03_Diagrama_Secuencia.md`
- **Descripción**: Flujos de interacción del sistema
- **Flujos principales**:
  - Agregar producto al carrito
  - Proceso de checkout
  - Proceso de pago
  - Gestión de pedidos (Admin)
  - Gestión de productos (Admin)
  - Webhook de pago

#### 1.4 Diagrama de Arquitectura
- **Archivo**: `04_Diagrama_Arquitectura.md`
- **Descripción**: Arquitectura completa del sistema
- **Componentes**: Frontend, Backend, Base de datos, Servicios externos
- **Tecnologías**: React, Node.js, MongoDB, Onvo, Cloudinary

#### 1.5 Diagramas del Panel Administrativo
- **Carpeta**: `diagramas_casos_uso_front-admin-parritico/`
- **Descripción**: Casos de uso específicos del panel administrativo
- **Contenido**: 8 documentos con diagramas Mermaid detallados
- **Funcionalidades**: Autenticación, Dashboard, Productos, Categorías, Subcategorías, Órdenes, Contactos, Usuarios
- **Ver más**: [README de Diagramas Admin](./03_Diagramas/diagramas_casos_uso_front-admin-parritico/README.md)

#### 1.6 Diagramas del E-commerce Público
- **Carpeta**: `diagramas_casos_uso_ecommerce-parritico/`
- **Descripción**: Casos de uso del e-commerce público
- **Contenido**: 6 documentos con diagramas Mermaid detallados
- **Funcionalidades**: Navegación, Carrito, Checkout, Pagos (Onvo/Stripe), Contacto
- **Ver más**: [README de Diagramas E-commerce](./03_Diagramas/diagramas_casos_uso_ecommerce-parritico/README.md)

### 2. 🎯 Metodología de Desarrollo
**Archivo**: `02_Metodologia.md`

#### Contenido:
- **Metodología**: Scrum
- **Roles**: Product Owner, Scrum Master, Development Team
- **Ceremonias**: Sprint Planning, Daily Standup, Sprint Review, Retrospective
- **Artefactos**: Product Backlog, Sprint Backlog, Incremento
- **Herramientas**: Git, GitHub, Node.js, React, MongoDB
- **Métricas**: Velocidad, calidad, satisfacción

### 3. 🗄️ Modelo de Datos
**Archivo**: `04_Modelo_de_Datos.md`

#### Contenido:
- **Base de datos**: MongoDB con Mongoose
- **Colecciones principales**:
  - Users (Usuarios)
  - Products (Productos con variantes)
  - Categories/SubCategories (Categorías)
  - Orders (Pedidos)
  - Contacts (Mensajes de contacto)
- **Relaciones**: Referenciales y embebidas
- **Validaciones**: Integridad y negocio
- **Índices**: Optimización de consultas

---

## 🚀 Tecnologías Utilizadas

### Backend
- **Node.js**: Runtime de JavaScript
- **Express.js**: Framework web
- **TypeScript**: Tipado estático
- **MongoDB**: Base de datos NoSQL
- **Mongoose**: ODM para MongoDB
- **JWT**: Autenticación
- **bcrypt**: Hash de contraseñas

### Frontend
- **React 19**: Biblioteca de UI
- **TypeScript**: Tipado estático
- **Vite**: Build tool
- **Tailwind CSS**: Framework CSS
- **Ant Design**: Componentes UI
- **Zustand**: Gestión de estado
- **React Router**: Enrutamiento

### Servicios Externos
- **Onvo**: Procesamiento de pagos
- **Cloudinary**: Gestión de imágenes
- **Socket.IO**: Comunicación en tiempo real

### Herramientas de Desarrollo
- **ESLint**: Linting de código
- **Git**: Control de versiones
- **npm**: Gestión de dependencias

---

## 🏛️ Arquitectura del Sistema

### Componentes Principales

#### 1. Frontend E-commerce
- **Propósito**: Interfaz para clientes
- **Funcionalidades**: Catálogo, carrito, checkout, pagos
- **Tecnologías**: React + Tailwind CSS + Ant Design

#### 2. Panel de Administración
- **Propósito**: Gestión administrativa
- **Funcionalidades**: Productos, pedidos, categorías, usuarios
- **Tecnologías**: React + Ant Design + React Query

#### 3. API Backend
- **Propósito**: Lógica de negocio y datos
- **Funcionalidades**: CRUD, autenticación, integraciones
- **Tecnologías**: Express.js + MongoDB + JWT

#### 4. Base de Datos
- **Propósito**: Almacenamiento de datos
- **Características**: NoSQL, documentos flexibles
- **Tecnologías**: MongoDB + Mongoose

#### 5. Servicios Externos
- **Onvo**: Procesamiento de pagos
- **Cloudinary**: Gestión de imágenes
- **Socket.IO**: Notificaciones en tiempo real

---

## 📊 Flujos Principales

### 1. Flujo de Compra
```
Cliente → Catálogo → Producto → Carrito → Checkout → Pago → Confirmación
```

### 2. Flujo de Administración
```
Admin → Login → Dashboard → Gestión → Productos/Pedidos → Actualización
```

### 3. Flujo de Pago
```
Checkout → Onvo → Procesamiento → Webhook → Actualización de Estado
```

---

## 🔧 Configuración y Despliegue

### Requisitos del Sistema
- **Node.js**: v18 o superior
- **MongoDB**: v5 o superior
- **npm**: v8 o superior

### Variables de Entorno
```env
# Base de datos
MONGODB_URI=mongodb://localhost:27017/parritico

# JWT
JWT_SECRET=your-secret-key

# Onvo
ONVO_API_KEY=your-onvo-key
ONVO_WEBHOOK_SECRET=your-webhook-secret

# Cloudinary
CLOUDINARY_CLOUD_NAME=your-cloud-name
CLOUDINARY_API_KEY=your-api-key
CLOUDINARY_API_SECRET=your-api-secret

# Frontend URLs
FRONTEND_STORE_URL=http://localhost:5173
FRONTEND_ADMIN_URL=http://localhost:5174
```

### Instalación
```bash
# Backend
cd backend-parritico
npm install
npm run dev

# Frontend E-commerce
cd ecommerce-parritico
npm install
npm run dev

# Frontend Admin
cd front-admin-parritico
npm install
npm run dev
```

---

## 📈 Métricas y Monitoreo

### Métricas de Desarrollo
- **Velocidad del equipo**: Story points por sprint
- **Calidad del código**: Cobertura de pruebas, complejidad
- **Satisfacción**: Feedback en sprint reviews

### Métricas de Negocio
- **Conversión**: Tasa de conversión de visitantes a compradores
- **Pedidos**: Número de pedidos por período
- **Productos**: Productos más vendidos
- **Clientes**: Análisis de comportamiento

---

## 🔒 Seguridad

### Medidas Implementadas
- **Autenticación JWT**: Tokens seguros con expiración
- **Hash de contraseñas**: bcrypt para encriptación
- **Validación de datos**: Sanitización de inputs
- **CORS**: Configuración de orígenes permitidos
- **Rate Limiting**: Protección contra abuso

### Mejoras Futuras
- **HTTPS**: Certificados SSL
- **Auditoría**: Log de acciones sensibles
- **Backup**: Respaldo regular de datos
- **Monitoreo**: Alertas de seguridad

---

## 🚀 Roadmap y Mejoras Futuras

### Funcionalidades Pendientes
- [ ] Sistema de notificaciones por email
- [ ] Dashboard de analytics avanzado
- [ ] Sistema de cupones y descuentos
- [ ] Integración con redes sociales
- [ ] App móvil nativa

### Mejoras Técnicas
- [ ] Implementar CI/CD
- [ ] Aumentar cobertura de pruebas
- [ ] Optimización de rendimiento
- [ ] Microservicios
- [ ] Cache distribuido

---

## 📞 Contacto y Soporte

### Equipo de Desarrollo
- **Desarrollador Principal**: Sergio
- **Metodología**: Scrum
- **Repositorio**: GitHub
- **Documentación**: Esta carpeta `/docs`

### Recursos Adicionales
- **API Documentation**: Endpoints y ejemplos
- **Code Examples**: Ejemplos de uso
- **Troubleshooting**: Solución de problemas comunes
- **Best Practices**: Mejores prácticas de desarrollo

---

## 📝 Changelog

### Versión 1.0.0 (Actual)
- ✅ Implementación completa del e-commerce
- ✅ Panel de administración funcional
- ✅ Integración con Onvo para pagos
- ✅ Gestión de imágenes con Cloudinary
- ✅ Sistema de autenticación JWT
- ✅ Documentación completa

### Próximas Versiones
- 🔄 v1.1.0: Mejoras de rendimiento
- 🔄 v1.2.0: Nuevas funcionalidades
- 🔄 v2.0.0: Arquitectura de microservicios

---

## 📄 Licencia

Este proyecto es propiedad de Parritico Store. Todos los derechos reservados.

---

*Última actualización: Octubre 2025*
*Versión de documentación: 1.0.0*
