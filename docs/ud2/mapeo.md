# 🔌 Introducción a JDBC y conectores

Un **conector o driver** es un mecanismo que permite a un lenguaje de programación conectarse, y trabajar, contra una base de datos. Se encarga de mantener el diálogo con la base de datos, para poder llevar a cabo el acceso y manipulación de los datos.

En este curso, nos vamos a centrar en **JDBC (Java Data Base Connectivity)**, puesto que, desde el punto de vista de Java, es una de las tecnologías más importantes de conectividad a la base de datos.

## 📖 ¿Qué es JDBC?

**JDBC (Java DataBase Connectivity)** es una API de Java diseñada por Sun/Oracle que permite a las aplicaciones acceder a **bases de datos relacionales** de manera uniforme.  
Con JDBC podemos:

- 🔗 Conectarnos a diferentes sistemas gestores de bases de datos (SGBD).  
- 📝 Ejecutar sentencias SQL.  
- 📊 Recuperar y manipular resultados.  

👉 En resumen, JDBC actúa como un **puente entre Java y la base de datos**.

---

## ⚖️ El desfase objeto–relacional

Las bases de datos relacionales no están diseñadas para almacenar objetos, ya que existe un desfase entre las construcciones típicas que proporciona el modelo relacional y las proporcionadas por la programación basada en objetos.

Java trabaja con **objetos**, mientras que las bases de datos relacionales almacenan información en **tablas** con filas y columnas.  
Este desajuste se conoce como **desfase objeto–relacional (Object–Relational Impedance Mismatch)**.

Ejemplos de problemas comunes:

- 🔄 Conversión entre **tipos de datos** (`int` ↔ `INTEGER`, `String` ↔ `VARCHAR`).  
- 🔗 Representación de **relaciones** (1:N, N:M) en estructuras de objetos.  
- 🧬 Manejo de **herencia** en BBDD que no la soportan de forma nativa.  

👉 En esta UD lo resolveremos manualmente con JDBC. Más adelante veremos soluciones ORM como **Hibernate**.

---

## 🌐 Protocolos de acceso a bases de datos

Los conectores JDBC utilizan **protocolos de acceso estándar** para comunicarse con el SGBD.  
Una URL de conexión JDBC suele tener la siguiente forma:

!!! example "Example 🤓"
    * **jdbc**:*bbdd*://_server_:_port_/_schema_
    * **jdbc**:mysql://localhost:3306/severo

---

## 🔧 Tipos de conectores JDBC

Un **conector JDBC (driver)** es una librería que implementa las interfaces de JDBC para un SGBD específico.  
Cada gestor (MySQL, PostgreSQL, Oracle…) tiene su propio driver.

Existen dos tipos principales:

### 🗂️ Drivers embebidos (embedded)
- La base de datos se ejecuta **dentro de la aplicación**.  
- Ejemplos: **H2**, **Apache Derby**.  
- ✅ Ventajas: sencillos, portables, sin instalación aparte.  
- ❌ Inconvenientes: poca escalabilidad, no aptos para producción masiva.  

### 🖥️ Drivers independientes (cliente-servidor)
- Se conectan a un **SGBD externo**.  
- Ejemplos: **MySQL**, **PostgreSQL**, **Oracle**.  
- ✅ Ventajas: potentes, multiusuario, adecuados para aplicaciones reales.  
- ❌ Inconvenientes: requieren instalación y configuración del servidor.  

---

![jdbc](../img/ud2/4jdbc.png)
