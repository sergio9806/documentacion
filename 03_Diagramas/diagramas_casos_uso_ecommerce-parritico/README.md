# Diagramas de Casos de Uso - E-commerce Parrítico

## Descripción

Esta carpeta contiene la documentación completa de los casos de uso del e-commerce público de Parrítico, desarrollado con React, TypeScript, Ant Design y TailwindCSS.

## Estructura de Documentos

### 📋 [00_Resumen.md](./00_Resumen.md)
Vista general del sistema, arquitectura, tecnologías y flujos principales. **Empieza aquí** para entender el panorama completo.

### 🏠 [01_Navegacion_Vistas.md](./01_Navegacion_Vistas.md)
Navegación y vistas del e-commerce:
- Página de inicio con carousel y productos destacados
- Vista de productos con filtros avanzados
- Vista detallada de producto individual
- Cambio de variantes e imágenes

### 🛒 [02_Gestion_Carrito.md](./02_Gestion_Carrito.md)
Gestión completa del carrito de compras:
- Agregar productos al carrito
- Ver carrito
- Actualizar cantidades
- Eliminar productos
- Persistencia en localStorage

### 💳 [03_Proceso_Checkout.md](./03_Proceso_Checkout.md)
Proceso de checkout:
- Formulario de checkout completo
- Resumen de pedido
- Selección de método de pago (Onvo/Stripe)

### 💰 [04_Proceso_Pago.md](./04_Proceso_Pago.md)
Procesamiento de pagos:
- Integración con Onvo (transferencia bancaria)
- Integración con Stripe (tarjeta crédito/débito)
- Validación de montos mínimos
- Página de éxito

### 📧 [05_Contacto.md](./05_Contacto.md)
Sistema de contacto:
- Envío de mensajes de contacto
- Validaciones de formulario
- Notificaciones en tiempo real al admin

## Tecnologías

- **Frontend**: React 19, TypeScript
- **UI**: Ant Design 5
- **State**: Zustand con persistencia
- **Routing**: React Router v7
- **Pagos**: Onvo & Stripe
- **Build**: Vite
- **Styling**: TailwindCSS

## Flujos Principales

### 🔄 Flujo de Compra Completo

```
Home → Productos → Detalle → Agregar a Carrito → Carrito → Checkout → Pago → Éxito
```

### 📦 Gestión de Carrito

```
Agregar → Actualizar → Eliminar → Vaciar → Checkout
```

### 💳 Proceso de Pago

```
Onvo: Checkout → Onvo → Pago → Webhook → Éxito
Stripe: Checkout → Payment Page → Tarjeta → Webhook → Éxito
```

## Características Principales

### ✨ Productos
- Catálogo completo con categorías
- Búsqueda y filtrado avanzado
- Productos con variantes (color, talla)
- Control de stock
- Imágenes múltiples

### 🛒 Carrito
- Persistencia local
- Agregar múltiples productos
- Cantidades dinámicas
- Validación de stock

### 💳 Pagos
- **Onvo**: Transferencia bancaria directa
- **Stripe**: Tarjetas de crédito/débito
- Validación de montos mínimos
- Webhooks de confirmación

### 📧 Contacto
- Formulario de contacto completo
- Validaciones robustas
- Notificaciones al admin
- Mensajes de confirmación

## Diagramas Incluidos

Cada documento contiene múltiples diagramas Mermaid:
- **Sequence Diagrams**: Flujos de interacción detallados
- **Flowcharts**: Lógica de decisión y navegación
- **State Diagrams**: Estados del sistema
- **Graph Diagrams**: Estructuras de datos y vistas

## Cómo Leer los Diagramas

1. **Sequence Diagrams**: Siguen la secuencia de interacción de arriba hacia abajo
2. **Flowcharts**: Muestran decisiones con diamantes (◊) y procesos con rectángulos
3. **State Diagrams**: Indican transiciones de estado
4. **Graph Diagrams**: Representan estructuras de datos y vistas

## Visualización

Los diagramas están en formato Mermaid y se pueden visualizar en:
- GitHub (renderizado automático)
- Editores con soporte Mermaid (VS Code, Cursor, etc.)
- [Mermaid Live Editor](https://mermaid.live)
- Documentación con soporte Mermaid

## Flujo de Lectura Recomendado

1. Comienza con [00_Resumen.md](./00_Resumen.md)
2. Revisa [01_Navegacion_Vistas.md](./01_Navegacion_Vistas.md)
3. Continúa con [02_Gestion_Carrito.md](./02_Gestion_Carrito.md)
4. Explora [03_Proceso_Checkout.md](./03_Proceso_Checkout.md)
5. Finaliza con [04_Proceso_Pago.md](./04_Proceso_Pago.md) y [05_Contacto.md](./05_Contacto.md)

## Integraciones

### Onvo
- Plataforma de pagos bancarios
- Transferencia en línea
- Webhooks automáticos

### Stripe
- Procesamiento de tarjetas
- Validación de montos
- Formularios seguros

### Socket.IO
- Notificaciones en tiempo real
- Eventos de contacto

## Notas Importantes

- El carrito persiste entre sesiones vía localStorage
- Los pagos pueden procesarse con Onvo o Stripe
- Las notificaciones al admin son en tiempo real
- Todos los formularios tienen validaciones robustas
- Los productos pueden tener múltiples variantes e imágenes

## Contacto

Para preguntas o sugerencias sobre esta documentación, contactar al equipo de desarrollo de Parrítico.

