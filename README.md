# 🍕 Taller: Modelado NoSQL Documental con MongoDB para una Pizzería

<p align="center"> 
  <img src="https://media.tenor.com/MwLf-almaYEAAAAi/vibe-pepe-the-frog-vibe-swag-pepe-the-frog.gif" width="350"/> 
</p>

<p align="center"> 
  <img src="https://img.shields.io/badge/MongoDB-6.0+-47A248?style=for-the-badge&logo=mongodb&logoColor=white" alt="MongoDB">
  <img src="https://img.shields.io/badge/MQL-Query%20Language-brightgreen?style=for-the-badge&logo=mongodb" alt="MQL">
  <img src="https://img.shields.io/badge/Database-Documental-darkgreen?style=for-the-badge&logo=database&logoColor=white" alt="Documental">
  <img src="https://img.shields.io/badge/License-MIT-yellow?style=for-the-badge" alt="MIT License">
  <img src="https://img.shields.io/badge/Status-In%20Progress-yellowgreen?style=for-the-badge" alt="In Progress">
</p>

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


# 🍕 Reflexión y Comparativa: Modelado Documental con MongoDB vs SQL Relacional

## 🧠 Comparativa Técnica

| Característica            | MySQL (Relacional)                                  | MongoDB (Documental)                                |
|:--------------------------|:----------------------------------------------------|:----------------------------------------------------|
| **Modelo de datos**       | Tablas con filas y columnas definidas.              | Colecciones con documentos BSON (similares a JSON). |
| **Esquema**               | Rígido y predefinido.                               | Flexible y dinámico.                                |
| **Relaciones**            | `JOIN`s a través de claves foráneas.                | Embebido de documentos o referencias (`$lookup`).   |
| **Escalabilidad**         | Vertical (aumentando la potencia del servidor).     | Horizontal (distribuyendo datos en más servidores). |
| **Lenguaje de Consulta**  | SQL (`Structured Query Language`).                  | MQL (`MongoDB Query Language`) y Aggregation Pipeline. |
| **Casos de uso**          | Sistemas transaccionales, ERPs, contabilidad.       | Big Data, catálogos de productos, redes sociales.   |

---

## 🧩 Reflexión Crítica del Proceso

> 💬 *“Pensar sin tablas nos abrió la mente.”*

Durante esta simulación grupal, el ejercicio de diseñar una base de datos sin las estructuras tradicionales relacionales fue un verdadero reto. Aquí algunas conclusiones agrupadas y contrastadas:

### ❗ Lo más difícil de imaginar sin tablas
- 🤯 Romper con la mentalidad de **normalización extrema** fue el mayor obstáculo.
- En SQL, la **redundancia** es considerada un error; en MongoDB, puede ser una **estrategia de rendimiento** válida.
- También costó asumir que los documentos **pueden crecer y divergir**, lo cual exige mayor cuidado y planeación.

### 💡 Lo que nos gustó del enfoque documental
- 📦 La **naturaleza intuitiva** de los documentos: un pedido en la app es un objeto; en MongoDB, también.
- ⚡ La **velocidad de prototipado**: agregar nuevos campos o estructuras es tan simple como modificar el documento.
- ❌ Sin necesidad de ORMs complicados o migraciones estructurales para cada cambio pequeño.

### 🤔 Dudas y reflexiones que surgieron
- 🔁 **¿Cómo garantizar consistencia con datos duplicados?**
  - Para históricos (como pedidos pasados): no se tocan.
  - Para datos activos (como dirección del cliente): se necesita lógica en la app para actualizaciones en cascada.
- 📊 **¿Cómo hacer análisis complejos sin SQL?**
  - El **Aggregation Framework** de MongoDB cubre eso. Es como `GROUP BY`... ¡con esteroides!
- 🔐 **¿Y las transacciones?**
  - MongoDB ≥ 4.0 soporta transacciones ACID multi-documento, pero SQL aún puede ser más robusto en operaciones altamente interdependientes.

---

## 🔍 Recomendación Final

Esta experiencia amplió nuestra perspectiva. Si vienes de SQL, **no asumas que NoSQL es mejor por default**. Evalúa siempre los *trade-offs*:
- ¿Necesitas flexibilidad y escalabilidad horizontal? MongoDB es fuerte.
- ¿Necesitas transacciones complejas y alta consistencia? Tal vez SQL es mejor.

👉 **Se Prueba con datos reales. Se Mide. Se Compara. Se Decide.**

---
## 👨‍💻 Autor

**Daniel Santiago Vinasco - Jhon Sebastian Ardila**

Desarrollado como parte del taller de NO-SQL Documental con MongoDB para una Pizzería

### Información de Contacto
- **GitHub**: [@DanielSantiagoV](https://github.com/DanielSantiagoV)


---

*Este proyecto cumple con todos los requerimientos especificados en el taller y proporciona una base sólida Documental con MongoDB para una Pizzería.* 

---

<p align="center">
  Developed with ❤️ by DanielSantiagoVinasco<br>
  🔥 <b><a href="https://github.com/DanielSantiagoV">Visit my GitHub</a></b> 🚀
</p>

