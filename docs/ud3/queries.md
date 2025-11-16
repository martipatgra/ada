# 🧩 Formas de realizar consultas en Hibernate

## 🎯 Objetivo
Aprender las principales formas de crear **consultas (querys)** en Hibernate 6 / Jakarta JPA:  
- JPQL / HQL  
- Criteria API  
- SQL Nativo  
- @NamedQuery  

Además, se explica **qué tipo de datos devuelve cada una** (entidad, DTO, `Object[]`, `Tuple`, etc.).

---

## 1️⃣ JPQL / HQL (Hibernate Query Language)

### 📘 Qué es
JPQL (Java Persistence Query Language) se inspiró en las primeras versiones de HQL y es un subconjunto del HQL moderno.
Lenguaje orientado a **entidades** y **atributos**, no a tablas SQL.  

Hibernate usa un poderoso lenguaje de consulta (HQL) que es similar en apariencia a SQL. Sin embargo, en comparación con SQL, **HQL está completamente orientado a objetos y comprende nociones como herencia, polimorfismo y asociación**.

### 🧠 Ejemplo
```java
String jpql = "SELECT s FROM Space s WHERE s.active = true AND s.capacity >= :min";
List<Space> result = session.createQuery(jpql, Space.class)
    .setParameter("min", 5)
    .getResultList();
```

```java title="named parameters"
//JPQL
TypedQuery<Usuario> q = session.createQuery(
  "SELECT u FROM Usuario u WHERE u.email LIKE :mail ORDER BY u.nombre",
  Usuario.class);
q.setParameter("mail", "%@example.com%");
q.setMaxResults(50).setFirstResult(0); // paginación
List<Usuario> usuarios = q.getResultList();
```

```java title="ordinal parameters"
    TypedQuery<Tramite> query = session.createQuery(
                            "from Tramite where tipo = ?1",
                            Tramite.class)
                    .setParameter(1, "Crédito");
```

!!! warning Warning
    No es una buena idea mezclar _named parameters_ y _ordinales_ en una sola consulta.

**JOIN y proyecciones (DTO):**
```java
List<UsuarioPedidoDTO> dtos = session.createQuery(
  "SELECT new com.acme.dto.UsuarioPedidoDTO(u.id, u.nombre, COUNT(p)) " +
  "FROM Usuario u LEFT JOIN u.pedidos p " +
  "GROUP BY u.id, u.nombre " +
  "ORDER BY COUNT(p) DESC",
  UsuarioPedidoDTO.class
).getResultList();

//también se puede hacer el join implícito  (INNER JOIN) navegando con ".": r.event.name, navegamos 
//desde Registration, por Event hasta el atributo de name en la entidad Event.
String jpql = "SELECT r.event.name, r.event.organizer.name, r.registrationDate, r.attendee.name " +
                    "FROM Registration r " +
                    "WHERE r.event.organizer.email = ?1 " +
                    "AND r.status = 'confirmed' " +
                    "ORDER BY r.event.name";
```

### 🧩 Tipo de dato devuelto

#### Entidad completa

```java
List<Customer> list = session.createQuery(
    "select c from Customer c where c.active = true", Customer.class
).getResultList();
```

### Varios campos --> Object[] por defecto

```java
List<Object[]> rows = session.createQuery(
    "select c.name, c.email from Customer c where c.active = true", Object[].class
).getResultList();

for (Object[] row : rows) {
  String name   = (String) row[0];
  String email  = (String) row[1];
}
```

### DTO directo --> preferido ✅

```java
String hql =
    "select new com.acme.CustomerDto(c.id, c.name, c.email) " +
    "from Customer c where c.active = true";
TypedQuery<CustomerDto> q = session.createQuery(hql, CustomerDto.class);
List<CustomerDto> rows = q.getResultList();
```

### Tuple

```java
TypedQuery<Tuple> q = session.createQuery(
    "select c.name as name, c.email as email from Customer c", Tuple.class
);
List<Tuple> rows = q.getResultList(); 
for (Tuple t : rows) {
    String name = t.get("name", String.class);
}
```

| Tipo de consulta | Devuelve |
|------------------|-----------|
| `SELECT s FROM Space s` | `List<Space>` (entidades gestionadas) |
| `SELECT s.name FROM Space s` | `List<String>` |
| `SELECT s.id, s.name FROM Space s` | `List<Object[]>` o `Tuple` |
| `SELECT new es.coworking.SpaceDTO(s.id, s.name)` | `List<SpaceDTO>` |

### ✅ Ventajas HQL/JPQL
- Legible y portable entre SGBD.  
- Permite joins, subconsultas, funciones agregadas.  
- Compatible con proyecciones (DTOs).

---

## 2️⃣ Criteria API (JPA)

### 📘 Qué es
API **programática y tipada** para construir queries tipadas y dinámicas con objetos Java.  
Ideal cuando los filtros dependen de condiciones del usuario o se generan en tiempo de ejecución.

### 🚶 Pasos

1. Consigue el CriteriaBuilder de la session.

`CriteriaBuilder` es la fábrica de construcciones (builder) stateless de la API de Criteria de JPA/Hibernate.
Sirve para **crear de forma tipada** todos los “ladrillos” de una consulta. Cada llamada **devuelve objetos nuevos**. No guarda configuración ni estado global.

| Categoría                 | Método                                                               | Uso / Ejemplo                   | SQL equivalente                      |
| ------------------------- | -------------------------------------------------------------------- | ------------------------------- | ------------------------------------ |
| **Select / Agregación**   | `cb.count(expr)`                                                     | Cuenta filas                    | `COUNT(*)`                           |
|                           | `cb.countDistinct(expr)`                                             | Cuenta sin duplicados           | `COUNT(DISTINCT x)`                  |
|                           | `cb.sum(expr)` / `cb.avg(expr)`                                      | Suma / media                    | `SUM(x)` / `AVG(x)`                  |
|                           | `cb.max(expr)` / `cb.min(expr)`                                      | Máximo / mínimo                 | `MAX(x)` / `MIN(x)`                  |
| **Comparación**           | `cb.equal(x, y)` / `cb.notEqual(x, y)`                               | Igual / distinto                | `=`, `<>`                            |
|                           | `cb.greaterThan(x,y)`, `cb.gt` / `cb.ge`, `cb.greaterThanOrEqualTo`  | Mayor / mayor o igual           | `>`, `>=`                            |
|                           | `cb.lessThan(x, y)`, `cb/lt` / `cb.le`, `cb.lessThanOrEqualTo`       | Menor / menor o igual           | `<`, `<=`                            |
|                           | `cb.between(x, a, b)`                                                | Dentro de un rango              | `BETWEEN a AND b`                    |
| **Booleanos**             | `cb.isTrue(expr)`                                                    | Comprueba `true`                | `WHERE col = true`                   |
|                           | `cb.isFalse(expr)`                                                   | Comprueba `false`               | `WHERE col = false`                  |
| **Nulos y colecciones**   | `cb.isNull(expr)` / `cb.isNotNull(expr)`                             | Valor nulo / no nulo            | `IS NULL` / `IS NOT NULL`            |
|                           | `cb.isEmpty(collection)` / `cb.isNotEmpty(collection)`               | Colección vacía / no vacía      | `NOT EXISTS(...)` / `EXISTS(...)`    |
|                           | `cb.isMember(elem, collection)` / `cb.isNotMember(elem, collection)` | Pertenencia a colección         | `elem IN (subquery)`                 |
| **Texto / cadenas**       | `cb.like(expr, pattern)` / `cb.notLike(expr, pattern)`               | Coincidencia parcial            | `LIKE` / `NOT LIKE`                  |
|                           | `cb.lower(expr)` / `cb.upper(expr)`                                  | Minúsculas / mayúsculas         | `LOWER()` / `UPPER()`                |
|                           | `cb.concat(a, b)`                                                    | Concatenación de cadenas        | `CONCAT(a, b)`                       |
|                           | `cb.length(expr)`                                                    | Longitud de cadena              | `LENGTH(expr)`                       |
|                           | `cb.locate(str, sub)`                                                | Posición de substring           | `LOCATE(sub, str)`                   |
| **Fechas / tiempos**      | `cb.currentDate()` / `cb.currentTimestamp()`                         | Fecha/hora actual               | `CURRENT_DATE` / `CURRENT_TIMESTAMP` |
|                           | `cb.greaterThan(date1, date2)`                                       | Comparar fechas                 | `>`                                  |
| **Lógicos**               | `cb.and(p1, p2, …)`                                                  | Conjunción                      | `AND`                                |
|                           | `cb.or(p1, p2, …)`                                                   | Disyunción                      | `OR`                                 |
|                           | `cb.not(p)`                                                          | Negación                        | `NOT`                                |
| **Subconsultas**          | `cb.exists(subquery)` / `cb.not(cb.exists(...))`                     | Comprobar existencia            | `EXISTS(subquery)`                   |
| **Ordenación**            | `cb.asc(expr)` / `cb.desc(expr)`                                     | Orden asc / desc                | `ORDER BY col ASC/DESC`              |
 

```java
CriteriaBuilder cb = session.getCriteriaBuilder();
```

2. Crea la consulta (elige el tipo de resultado):

CriteriaQuery es la representación en Java de una consulta JPQL.

```java
CriteriaQuery<Usuario> cq = cb.createQuery(Usuario.class); // tipada
// o: CriteriaQuery<Tuple> cq = cb.createTupleQuery();      // tuplas
// o: CriteriaQuery<Object[]> cq = cb.createQuery(Object[].class); // arrays
// o: CriteriaQuery<TopSpaceRevenueDto> d = cb.createQuery(TopSpaceRevenueDto.class); //Dto
```

3. Define la raíz (Root)

La raíz siempre es una entidad. Es el FROM de la query SQL.

```java
Root<Usuario> u = cq.from(Usuario.class);
```

4. Define las uniones JOIN, si las necesitas

```java
Root<Booking> root = criteria.from(Booking.class);
Join<Booking, Space> join = root.join("space");//nombre del objeto en la clase Booking
```

5. Selecciona lo que quieres devolver

```java
cq.select(u); // opcional si solo hay un root, devuelve todo el objeto, como poner * en una select
cq.select(u.get("email")); //devuelve un campo
cq.multiselect(u.get("id"), u.get("nombre")); // -> devuelve varios campos en forma de Tuple
cq.select(cb.construct(UserDTO.class, u.get("id"), u.get("nombre"))); //Como DTO/wrapper (lo más limpio)
```

6. Añade filtros (predicados)

```java
 cq.multiselect(
            root.get("nombre"),
            cb.count(root.get("id"))
    )
    .where(cb.and(cb.isNull(root.get("fechaFin")),
            cb.ge(root.get("capacidad"),10)))     //decimos que la fechaFin no sea nula y que capacidad sea mayor que 10
```

7. Orden

```java
cq.orderBy(cb.asc(u.get("nombre")));
TypedQuery<Usuario> q = session.createQuery(cq);
```

8. Ejecuta

```java
List<Usuario> resultados = session.createQuery(cq).getResultList();
```

### 🧠 Ejemplos
```java
CriteriaBuilder cb = session.getCriteriaBuilder();
CriteriaQuery<Space> cq = cb.createQuery(Space.class);
Root<Space> root = cq.from(Space.class);

cq.select(root)
  .where(cb.and(
      cb.isTrue(root.get("active")),
      cb.ge(root.get("capacity"), 5)
  ));

List<Space> result = session.createQuery(cq).getResultList();
```

#### JOIN 
```java
CriteriaBuilder cb = session.getCriteriaBuilder();
CriteriaQuery<Space> cq = cb.createQuery(Space.class);
Root<Space> root = cq.from(Space.class);
Join<Space, Venue> jVenue = root.join("venue");//INNER JOIN
    cq.select(root)
            .where(cb.equal(jVenue.get("city"), "Zaragoza"));
```

#### Proyección a DTO

```java
CriteriaQuery<SpaceCardDto> cq3 = cb.createQuery(SpaceCardDto.class);
            Root<Space> r3 = cq3.from(Space.class);
            cq3.select(cb.construct(
                    SpaceCardDto.class,
                    r3.get("code"), r3.get("name"), r3.get("hourlyPrice")
            ));

            List<SpaceCardDto> rows2 = session.createQuery(cq3).getResultList();
            System.out.println(rows2);
```

#### BETWEEN y LIKE
```java
//BETWEEN, espacios en los que el precio por hora está entre 10 y 30
cq.select(root)
        .where(cb.between(root.get("hourlyPrice"),
                new BigDecimal("10"), new BigDecimal("30")));

//LIKE
String keyword = "meeting";
  cq.select(root)
          .where(cb.like(cb.lower(root.get("name")), "%" + keyword.toLowerCase() + "%"));


```

#### COUNT

```java
CriteriaQuery<Long> cqCount = cb.createQuery(Long.class);
Root<Space> rCount = cqCount.from(Space.class);
cqCount.select(cb.count(rCount))
        .where(cb.isTrue(rCount.get("active")));

Long total = session.createQuery(cqCount).getSingleResult();
```

#### Resultado como TUPLE

```java
//resultado como tuple
  CriteriaQuery<Tuple> cqTuple = cb.createTupleQuery();
  Root<Space> rT = cqTuple.from(Space.class);
  Join<Space, Venue> vT = rT.join("venue");

  cqTuple.multiselect(
          rT.get("id").alias("id"),
          rT.get("name").alias("name"),
          vT.get("name").alias("venueName")
  );

  List<Tuple> rows3 = session.createQuery(cqTuple).getResultList();

  for (Tuple t : rows3) {
      Long id = t.get("id", Long.class);
      String name = t.get("name", String.class);
      String venueName = t.get("venueName", String.class);
      System.out.println(id + "  "  + name + "   " + venueName);
  }
```

#### Group By, Having
```java
CriteriaQuery<Object[]> cq2 = cb.createQuery(Object[].class);
Root<Space> s2 = cq2.from(Space.class);
Join<Space, Venue> v2 = s2.join("venue");

Expression<BigDecimal> avgPrice = cb.avg(s2.get("price")).as(BigDecimal.class);

cq2.multiselect(v2.get("city"), avgPrice)
   .groupBy(v2.get("city"))
   .having(cb.greaterThan(avgPrice, new BigDecimal("20")));
```

### 🧩 Tipo de dato devuelto
- Si defines `CriteriaQuery<Space>` → `List<Space>`  
- Si seleccionas varios atributos (`cb.tuple(...)`) → `List<Tuple>`  
- Si seleccionas columnas específicas → `List<Object[]>`  
- También puede proyectarse a un DTO usando `cb.construct(...)`.

### ✅ Ventajas
- Type-safe (detecta errores en compilación).  
- Ideal para consultas dinámicas.  
- 100% estándar JPA.

## ➡️ Tabla Resumen

- CriteriaBuilder (`cb`) → fábrica de piezas: `Predicate`, `Expression`, `Order`, etc.   
- CriteriaQuery (`cq`) → plano de la consulta: `select`, `from`, `where`, `groupBy`, `having`, `orderBy`, `distinct`.   
- Desde el `from` (un Root/Join) sacas rutas (Path) y haces joins.      

| Clase                          | Qué es                            | Quién la crea                        | Dónde se usa                             | Ejemplo corto                                             |
| ------------------------------ | --------------------------------- | ------------------------------------ | ---------------------------------------- | --------------------------------------------------------- |
| `CriteriaBuilder`              | Fábrica y helpers                 | `em.getCriteriaBuilder()`            | En todas partes para construir cosas     | `cb.equal(…), cb.and(…), cb.sum(…)`                       |
| `CriteriaQuery<T>`             | Plano tipado de la query `SELECT` | `cb.createQuery(T.class)`            | `select`, `where`, `groupBy`, …          | `cq.select(u).where(pred)`                                |
| `Root<T>`                      | Raíz de `FROM` (tabla principal)  | `cq.from(T.class)`                   | Para acceder a campos y hacer joins      | `Root<User> u = cq.from(User.class)`                      |
| `Join<X,Y>`                    | Join tipado a una relación        | `root.join("roles")`                 | Igual que tabla unida                    | `Join<User,Role> r = u.join("roles")`                     |
| `Path<X>`                      | Ruta a un atributo (columna)      | `root.get("email")`                  | Alimenta `Expression`, `Predicate`       | `Path<String> p = u.get("name")`                          |
| `Expression<X>`                | Valor calculado (sum, lower, etc) | `cb.lower(path)`, `cb.sum(…)`        | `select`, `groupBy`, `orderBy`, `having` | `Expression<String> e = cb.lower(p)`                      |
| `Predicate`                    | Condición booleana                | `cb.equal(…)`, `cb.and(…)`           | `where`, `having`                        | `Predicate p = cb.equal(u.get("active"), true)`           |
| `Order`                        | Ordenación                        | `cb.asc(expr)`, `cb.desc(expr)`      | `orderBy`                                | `cq.orderBy(cb.desc(u.get("createdAt")))`                 |
| `Subquery<X>`                  | Subconsulta                       | `cq.subquery(X.class)`               | Como parte de `where`, `select`          | `Subquery<Long> sq = cq.subquery(Long.class)`             |
| `CompoundSelection<T>`         | Selección compuesta (DTO/tuple)   | `cb.construct(Dto.class, …)`         | `select` / `multiselect`                 | `cq.select(cb.construct(Dto.class, u.get("id"), …))`      |
| `Tuple` y `TupleQuery`         | Fila heterogénea y su query       | `cb.createTupleQuery()`              | Cuando quieres columnas sueltas          | `TupleQuery tq = cb.createTupleQuery()`                   |


---

## 3️⃣ SQL Nativo (`createNativeQuery`)

### 📘 Qué es
Permite ejecutar SQL **directamente sobre la base de datos**.  
Se usa cuando necesitas funciones específicas del motor o rendimiento extremo.

### 🧠 Ejemplo
```java
String sql = "SELECT * FROM spaces WHERE active = 1 AND capacity >= ?";
List<Space> result = session.createNativeQuery(sql, Space.class)
    .setParameter(1, 5)
    .getResultList();

List<Object[]> filas = session.createNativeQuery(
  "SELECT id, nombre FROM usuarios WHERE email LIKE ?")
  .setParameter(1, "%@example.com%")
  .getResultList();
```

### 🧩 Tipo de dato devuelto
| Parámetro usado | Devuelve |
|-----------------|-----------|
| `createNativeQuery(sql)` | `List<Object[]>` |
| `createNativeQuery(sql, Space.class)` | `List<Space>` (entidades gestionadas) |
| `createNativeQuery(sql, Tuple.class)` | `List<Tuple>`  |

### ✅ Ventajas
- Acceso total al SQL y funciones nativas.  
- Permite mapear a entidades o DTOs.  
- Útil para consultas complejas o migraciones.

### ⚠️ Desventajas
- No es portable entre bases de datos.  
- No aprovecha completamente el contexto de persistencia.

---

## 4️⃣ @NamedQuery

### 📘 Qué es
Consultas **JPQL predefinidas**, declaradas en la propia entidad.  
Hibernate las valida al iniciar la aplicación.

### 🧠 Ejemplo
```java
@Entity
@NamedQuery(
  name = "Space.findActive",
  query = "FROM Space s WHERE s.active = true"
)
public class Space { ... }

// Uso
List<Space> spaces = em.createNamedQuery("Space.findActive", Space.class)
                       .getResultList();
```

### 🧩 Tipo de dato devuelto
- Igual que JPQL: entidad (`List<Space>`), atributo (`List<String>`), DTO (`List<Object[]>` o `List<DTO>`).

### ✅ Ventajas
- Centraliza consultas reutilizables.  
- Validación temprana (errores detectados al arrancar).  
- Buenas prácticas para queries compartidas.

---

#### 🎐 _EJECUTANDO la consulta_

Dentro de la query se puede restringir el número de resultados que devuelve con la opción `setMaxResults(int maxResults)`.
Ejemplo:

```java
String sql = "SELECT * FROM spaces WHERE active = 1 AND capacity >= ?";
Space result = session.createNativeQuery(sql, Space.class)
    .setParameter(1, 5)
    .setMaxResults(1);
    .getSingleResultOrNull();
```

La interfaz `TypedQuery` se utiliza para controlar la ejecución de la consulta. Ofrece tres tipos de resultados:

- `getResultList()`: devuelve una lista con los resultados (lista vacía si no hay filas). No lanza excepción por “cero resultados”..
- `getSingleResult()`: es solo para casos en los que la consulta siempre **devuelve exactamente un resultado**. Lanza excepción si no hay ninguno o si hay más de uno.
- `getSingleResultOrNull()` → devuelve el único resultado o null si no hay ninguno; sigue lanzando excepción si hay más de uno.
- `getResultStream()`: permite que los resultados se recuperen **de forma incremental**, utilizando un cursor de base de datos.

!!! warning Warning
    El método `getResultStream()` **no suele ser útil**. Casi siempre es una mala idea mantener abierto el cursor de una base de datos.

---

## 🧾 Comparativa general

| Tipo de Query | Sintaxis | Tipo de retorno | Uso típico | Ventajas | Desventajas |
|----------------|-----------|------------------|-------------|-----------|--------------|
| **JPQL/HQL** | Texto (entidades) | `List<Entidad>`, `List<Object[]>`, `List<Tuple>`, `List<DTO>` | Consultas estándar | Legible y portable | No tipo-segura |
| **Criteria API** | Programática (Java) | `List<Entidad>`, `List<Tuple>`, `List<DTO>` | Filtros dinámicos | Tipo-segura y estándar | Verbosa |
| **SQL Nativo** | SQL directo | `List<Object[]>`, `List<Tuple>`, `List<Entidad>` | Casos complejos o rendimiento | Potente y flexible | No portable |
| **@NamedQuery** | Declarativa | Igual que JPQL | Consultas reutilizables | Validada al inicio | Menos flexible |

---

## 📚 Conclusión
- Usa **JPQL/HQL** para la mayoría de los casos.  
- Usa **Criteria API** cuando necesites construir filtros dinámicos.  
- Usa **SQL nativo** cuando necesites funciones específicas o rendimiento extremo.  
- Usa **@NamedQuery** para centralizar y reutilizar consultas comunes.
