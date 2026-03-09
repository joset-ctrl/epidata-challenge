# Challenge Sr. Fullstack – E-commerce con Microservicios y Eventos

Este repositorio es mi solución al desafío técnico Fullstack Senior: tomar un backend monolítico en NestJS con varios problemas estructurales, diagnosticarlo, refactorizarlo hacia una arquitectura **event-driven**, y conectar un frontend en React que consuma esos eventos en tiempo real.

## ¿Qué hice en resumen?

- Diagnosticé y arreglé fallas críticas en el backend original (validaciones rotas, acoplamiento fuerte, cero eventos, config no cloud-ready)
- Implementé **eventos de dominio** con EventEmitter2 (product.created y product.activated)
- Creé un flujo asincrónico real: el backend emite eventos → frontend los escucha en vivo con **Server-Sent Events (SSE)**
- Desarrollé un frontend limpio en React + Vite que reacciona automáticamente a los eventos
- Todo deployado y funcional en Render

## Problemas encontrados en el código original

- El flujo de creación de productos + detalles estaba **roto**:  
  El endpoint para agregar detalles no validaba campos obligatorios → el producto nunca llegaba a `isActive: true`
- Lógica de negocio muy acoplada → no había forma de reaccionar a cambios importantes desde otros servicios
- TypeORM sin SSL → imposible deploy en la mayoría de proveedores cloud modernos

## Lo que cambié (las partes más interesantes)

### Eventos de dominio emitidos

| Evento               | Cuándo se emite                              | Para qué sirve                                                                 |
|----------------------|----------------------------------------------|---------------------------------------------------------------------------------|
| `product.created`    | Después de crear el producto base            | Indexación de búsqueda, notificaciones, integraciones externas…                |
| `product.activated`  | Cuando se completa con detalles y se activa  | Inicializar stock, mostrar en catálogo público, disparar procesos downstream   |

### Frontend en tiempo real

El backend expone `/product/stream-events` (SSE).  
El frontend usa `EventSource` nativo y:

- Muestra logs visuales cuando llegan eventos
- Actualiza automáticamente la grilla de productos sin recargar la página

### Decisiones técnicas clave

- **SSE** en vez de WebSockets → unidireccional, liviano, sin dependencias extras
- TypeORM cloud-ready: credenciales por env + `ssl: { rejectUnauthorized: false }`
- Frontend preparado para dev y prod con `VITE_API_URL`

## Dónde verlo funcionando

Todo deployado en **Render** (100% operativo):

- **Backend API**: https://epidata-challenge-api.onrender.com
- **Frontend**: https://epidata-challenge-frontend.onrender.com
- Base de datos: PostgreSQL 14 

## Cómo correrlo localmente

### Requisitos

- Node.js ≥ 18
- PostgreSQL (local o vía Docker)

### Backend

bash
# En la raíz del proyecto
npm install
npm run migration:run
npm run seed                # opcional pero recomendado
npm run start:dev 

### Frontend
cd frontend
npm install
npm run dev

Por defecto apunta a http://localhost:3000.
Si querés cambiar la URL del backend, crea un archivo .env en /frontend con:
textVITE_API_URL=http://localhost:3000
