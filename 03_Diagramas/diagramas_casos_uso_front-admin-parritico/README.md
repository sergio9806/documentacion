# Diagramas de Casos de Uso - Panel Administrativo

## Descripción

Esta carpeta contiene la documentación completa de los casos de uso del panel administrativo de Parrítico, desarrollado con React, TypeScript y Ant Design.

## Estructura de Documentos

### 📋 [00_Resumen.md](./00_Resumen.md)
Vista general del sistema, arquitectura, roles, permisos y flujos principales. **Empieza aquí** para entender el panorama completo.

### 🔐 [01_Autenticacion.md](./01_Autenticacion.md)
Flujos de autenticación:
- Iniciar sesión (Login)
- Cerrar sesión (Logout)
- Control de acceso con guards

### 📊 [02_Dashboard.md](./02_Dashboard.md)
Funcionalidades del dashboard:
- Visualización de estadísticas (Admin)
- Bienvenida personalizada (Organizer)
- Última actividad

### 🛍️ [03_Gestion_Productos.md](./03_Gestion_Productos.md)
Gestión completa de productos:
- CRUD de productos
- Variantes con imágenes
- Control de stock
- Activación/Desactivación
- Eliminación permanente

### 📁 [04_Gestion_Categorias.md](./04_Gestion_Categorias.md)
Administración de categorías:
- CRUD de categorías
- Imágenes por categoría
- Activación/Desactivación
- Eliminación en cascada

### 📂 [05_Gestion_Subcategorias.md](./05_Gestion_Subcategorias.md)
Gestión de subcategorías:
- CRUD de subcategorías
- Relación con categorías padre
- Activación/Desactivación
- Eliminación en cascada

### 📦 [06_Gestion_Ordenes.md](./06_Gestion_Ordenes.md)
Administración de órdenes:
- Listado por estados
- Detalle completo
- Filtrado por rol
- Tracking de envíos

### 📧 [07_Gestion_Contactos.md](./07_Gestion_Contactos.md)
Manejo de contactos:
- Listado de mensajes
- Detalle de mensaje
- Marcado como revisado
- Eliminación
- Notificaciones en tiempo real (Socket.IO)

### 👥 [08_Gestion_Usuarios.md](./08_Gestion_Usuarios.md)
Gestión de usuarios:
- Crear nuevo usuario (Solo Admin)
- Asignación de roles

## Tecnologías

- **Frontend**: React 19, TypeScript
- **UI**: Ant Design 5
- **State**: Zustand
- **Data Fetching**: React Query
- **Routing**: React Router v7
- **Real-time**: Socket.IO
- **Alerts**: SweetAlert2
- **Build**: Vite

## Roles del Sistema

### 🔴 Administrador (Admin)
Acceso completo a todas las funcionalidades:
- Gestión de productos, categorías, subcategorías
- Visualización de dashboard con estadísticas
- Gestión de órdenes
- Manejo de contactos
- Creación de usuarios

### 🟢 Organizador (Organizer)
Acceso limitado:
- Dashboard de bienvenida (sin estadísticas)
- Gestión de órdenes
- Manejo de contactos
- Sin acceso a productos, categorías o usuarios

## Diagramas Incluidos

Cada documento contiene múltiples diagramas Mermaid:
- **Sequence Diagrams**: Flujos de interacción entre componentes
- **Flowcharts**: Lógica de decisión y navegación
- **State Diagrams**: Estados del sistema
- **Graph Diagrams**: Relaciones entre entidades

## Cómo Leer los Diagramas

1. **Sequence Diagrams**: Siguen la secuencia de interacción de arriba hacia abajo
2. **Flowcharts**: Muestran decisiones con diamantes (◊) y procesos con rectángulos (▭)
3. **State Diagrams**: Indican transiciones de estado del sistema
4. **Graph Diagrams**: Representan relaciones y jerarquías

## Visualización

Los diagramas están en formato Mermaid y se pueden visualizar en:
- GitHub (renderizado automático)
- Editores con soporte Mermaid (VS Code, Cursor, etc.)
- [Mermaid Live Editor](https://mermaid.live)
- Documentación con soporte Mermaid

## Flujo de Lectura Recomendado

1. Comienza con [00_Resumen.md](./00_Resumen.md)
2. Revisa [01_Autenticacion.md](./01_Autenticacion.md)
3. Continúa con [02_Dashboard.md](./02_Dashboard.md)
4. Explora las secciones de gestión según tu interés

## Actualización

Estos diagramas se actualizan cuando:
- Se agregan nuevas funcionalidades
- Se modifican flujos existentes
- Se refactoriza el código
- Se cambian las relaciones entre componentes

## Notas Importantes

- Los diagramas reflejan la implementación actual del sistema
- Todos los casos de uso incluyen manejo de errores
- Los roles determinan qué funcionalidades están disponibles
- Socket.IO proporciona actualizaciones en tiempo real para contactos

## Contacto

Para preguntas o sugerencias sobre esta documentación, contactar al equipo de desarrollo de Parrítico.

