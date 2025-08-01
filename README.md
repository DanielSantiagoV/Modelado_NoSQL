# 🍕 Taller: Modelado NoSQL Documental con MongoDB para una Pizzería

> 🧠 Exploramos cómo modelar los datos de una pizzería usando una base de datos documental en lugar de un modelo relacional tradicional.

---

## 📚 Investigación

### ❓ ¿Qué es una base de datos NoSQL?

Una base de datos NoSQL (Not Only SQL) permite almacenar información en formatos no tabulares. Es ideal para sistemas que requieren flexibilidad en la estructura de los datos, alto rendimiento y escalabilidad horizontal.

### 🍃 ¿Qué es MongoDB?

MongoDB es una base de datos NoSQL orientada a documentos. Utiliza documentos BSON (muy similares a JSON) para representar y almacenar datos complejos, anidados y semiestructurados.

### ⚖️ Diferencias clave entre MySQL (Relacional) y MongoDB (Documental)

| Característica         | MySQL (Relacional)                                  | MongoDB (Documental)                                |
|:-----------------------|:----------------------------------------------------|:----------------------------------------------------|
| **Modelo de datos**    | Tablas con filas y columnas definidas.              | Colecciones con documentos BSON (similares a JSON). |
| **Esquema**            | Rígido y predefinido.                               | Flexible y dinámico.                                |
| **Relaciones**         | `JOIN`s a través de claves foráneas.                | Embebido de documentos o referencias (`$lookup`).   |
| **Escalabilidad**      | Vertical (aumentando la potencia del servidor).     | Horizontal (distribuyendo datos en más servidores). |
| **Lenguaje de Consulta** | SQL (`Structured Query Language`).                  | MQL (`MongoDB Query Language`) y Aggregation Pipeline.|
| **Casos de uso**       | Sistemas transaccionales, ERPs, contabilidad.       | Big Data, catálogos de productos, redes sociales.   |

### 📄 ¿Qué son documentos y colecciones?

- **Documento**: Es una unidad de datos en formato JSON (ej. un pedido o un producto).
- **Colección**: Es un conjunto de documentos similares (ej. todos los pedidos).

---

## 🧩 Diseño Detallado del Modelo

En lugar de normalizar como en SQL (con tablas separadas para clientes, pedidos, etc.), usaremos documentos para agrupar datos relacionados. El objetivo es equilibrar la flexibilidad con la eficiencia en las consultas, evitando redundancias excesivas.

### 🗂️ Colecciones Principales

- **`productos`**: Almacena todos los artículos vendibles (pizzas, bebidas, postres). Incluye variaciones como tamaños y precios.
- **`combos`**: Define paquetes que agrupan varios productos. Utiliza **referencias** a los documentos en `productos` para mantener la consistencia de precios y nombres.
- **`pedidos`**: Contiene la información de cada orden. Los detalles de los productos comprados se **embebén** para crear un registro histórico inmutable.
- **`clientes`**: Guarda los datos de los usuarios. Se **referencia** desde los pedidos para no duplicar información.

### ⚖️ Justificación: Embeber vs. Referenciar

La decisión clave en MongoDB es cuándo anidar datos (embeber) y cuándo crear un enlace (referenciar).

- **Embebemos** datos cuando la relación es de "contiene" y los datos no se consultan fuera de su documento padre.
  - **Ventaja**: Lecturas atómicas y rápidas (un solo viaje a la base de datos).
  - **Ejemplo**: Los `items` dentro de un `pedido`. Un pedido siempre contiene sus productos.
  - **Ejemplo**: El `customerSnapshot` en un `pedido`. Guardamos una "foto" de los datos del cliente al momento de la compra para preservar el historial.

- **Referenciamos** datos cuando la relación es de "usa" o para evitar la duplicación de grandes volúmenes de datos que cambian con frecuencia.
  - **Ventaja**: Mantiene los datos consistentes (DRY - Don't Repeat Yourself). Actualizar un producto se refleja en todos los combos que lo usan.
  - **Ejemplo**: El `customerId` en un `pedido`. El cliente es una entidad independiente.
  - **Ejemplo**: Los `productId` en un `combo`. El combo "usa" productos que existen por sí mismos.

> **Advertencia sobre el Trade-off**: Si los precios de los ingredientes cambiaran muy frecuentemente, referenciarlos desde el producto podría ser mejor que embeberlos para evitar actualizaciones masivas. Sin embargo, para una pizzería, este cambio es poco frecuente, por lo que embeber es más eficiente para las lecturas.

### 🧬 Estructura de Campos Clave

- **Listas (Arrays)**: Ideal para relaciones "uno a muchos".
  - `items` en la colección `pedidos`.
  - `sizes` y `baseIngredients` en la colección `productos`.
- **Objetos (Sub-documentos)**: Perfectos para agrupar datos relacionados dentro de un documento.
  - `customerSnapshot` en `pedidos`.
  - Cada elemento del array `additions` en un item del pedido (ej: `{ "name": "Borde de Queso", "price": 5000.00 }`).
---

## 🧪 Ejemplos de Documentos JSON

### 📦 Producto

 ```json
 {
   "_id": "63a0c6a5b4b3f8a4c1e8a4a1",
   "name": "Pizza Hawaiana",
   "category": "pizza",
   "sizes": [
     { "name": "Mediana", "price": 32000.00 },
     { "name": "Familiar", "price": 45000.00 }
   ],
   "available": true
 }
 ```
 
 ### 🎁 Combo
 
 ```json
 {
   "_id": "63a0c6a5b4b3f8a4c1e8a4b5",
   "name": "Combo Amigos",
   "price": 55000.00,
   "isCombo": true,
   "comboItems": [
     { "productId": "63a0c6a5b4b3f8a4c1e8a4a1", "description": "1 Pizza Mediana" },
     { "productId": "63a0c7e2b4b3f8a4c1e8a4c1", "description": "2 Gaseosas" }
   ]
 }
 ```
 
 ### 👥 Pedido
 
 ```json
 {
   "_id": "63a0d3b3e8a4b3f8a4c1e8b1",
   "orderDate": "2025-08-16T20:30:00Z",
   "status": "en_preparacion",
   "customerSnapshot": {
     "name": "Ana Gómez",
     "phone": "3001234567"
   },
   "items": [
    {
       "productId": "63a0c6a5b4b3f8a4c1e8a4a1",
       "name": "Pizza Hawaiana",
       "quantity": 1,
       "additions": [
         { "name": "Borde de Queso", "price": 5000.00 }
      ],
       "subtotal": 37000.00
    }
  ],
   "totalAmount": 37000.00
 }
 ```


 