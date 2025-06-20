# Entregable 1 de proyecto

## Descripción del proyecto Seleccionado

El objetivo es crear una tienda virtual multitenant que ofrezca todos los servicios de sus tiendas clientes. Además de que cada cliente podrá gestionar sus servicios a través de un panel de Administración además poder utilizar tanto navegación como promoción de sus productos.

De tal manera se tiene:

### Panel de Administración

- Gestión de tenants: creación, aprobación y configuración de un nuevo cliente.

- Registro y mantenimiento de servicios comunes: facturación, productos, reportes.

- Cada tenant tiene su propio esquema en la base de datos (multi-schema), para aislar datos.

### Plantilla de Tienda (Storefront Template)

- Interfaz pública donde los clientes finales navegan y compran productos.

- Consume los datos del catálogo de products de cada shcema-tenant.

- Soporta personalización mínima (logos, colores básiscos) y plantillas de promoción por tenant.

### Servicio de Facturación e Inventario

- Módulo independiente que emite facturas y controla stock.

- Expuesto vía API REST para Admin Portal y Storefront.

### Autenticación y AUtización (Autch Service)

- Emite tokens JWT y valida permisos
- Separa roles: super-admin (gestión de tenants), tenant-admin (gestión de su tienda), usuario final.

### Gestión de planes 

- Seleccionar si es requerido un esquema dentro de un servidor compartido o un servidor dedicado.

- Limitación de la personalización, cantidad de productos y tráfico.

- Niveles de servicio (SLA), backups, soporte.

### Provisionamiento automático:

- Planes Free/Standard → compartición en un mismo clúster de aplicaciones y base de datos (multi‑schema).

- Planes Premium/Enterprise → despliegue en un servidor o contenedor aislado (CPU/RAM propios) y su propia instancia de PostgreSQL


## Elección de Enfoque de Arquitectura

Grupo 4: Análisis de escalabilidad futura

Asumimos que, en el primer mes de operación, la carga podría aumentar hasta un 400 %.
La arquitectura se diseña para escalar horizontalmente cada módulo de forma independiente y permitir la adición de nuevos tenants sin impacto en el rendimiento de los existentes.

Justificación:

Multitenancy exige que cada schema pueda crecer en datos y transacciones sin degradar al resto.

Separar servicios (Auth, Billing, Admin, Storefront) como microservicios facilita replicar solo los cuellos de botella.


## Esquema general de arquitectura
![Boceto Inicial Arqutiectura por Modulos](https://github.com/user-attachments/assets/0c5b8fe8-262a-4346-894e-08bfa7169fd7)

# Descripción:

## Frontend (Storefront Template)

- Los clientes finales pueden navegar por las tiendas de cada tenant.
- Se van a consumir los datos de productos a través de **API REST**, obteniendo información de su esquema correspondiente.

## Backend (Panel de Administración)

- **Microservicios**: Se incluyen servicios como **Autenticación**, **Facturación**, **Inventario** y **Gestión de Tenants**.
- Cada uno de estos microservicios tiene su propia instancia, que puede escalarse según la demanda.

## Base de Datos Multi-Schema

- Cada tenant tiene su propio **esquema**, lo que garantiza la separación de datos.
- La base de datos se escala horizontalmente, utilizando técnicas de **sharding** (partición) si es necesario.

## API Gateway

- Actúa como punto de entrada a todos los microservicios.
- Maneja las solicitudes de los usuarios finales y las solicitudes administrativas, redirigiéndolas al servicio correspondiente.

## Balanceo de Carga

- Usado para distribuir las solicitudes entre múltiples instancias de los servidores backend y asegurar **alta disponibilidad**.

## Caché y Mensajería Asíncrona

- Se optimizará el rendimiento con **consultas eficientes** en la base de datos y técnicas de **indexación**.
- Además, para las tareas que deben procesarse de manera asíncrona, se utilizarán **colas de trabajo** simples como **RabbitMQ** para manejar procesos como la **generación de facturas** y el **envío de correos electrónicos**, evitando que el sistema se sobrecargue durante operaciones de largo plazo.
