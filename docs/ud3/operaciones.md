# 🧪 Operaciones con objetos

En este bloque verás cómo **cargar, guardar, actualizar y borrar** entidades; cómo **consultar** usando **HQL/JPQL**, **SQL nativo** y **Criteria API**; y cómo **gestionar transacciones** con JPA/Hibernate.

---

## 💾 Operaciones de carga, almacenamiento y modificación (ORM)

### 1 Persistir / guardar
- **JPA/Hibernate**: `persist(ent)` → *transient → managed* → genera **INSERT** en `flush/commit`.
- Id **autogenerado**: no confíes en él hasta el `flush` (o tras commit).

```java
Usuario u = new Usuario("Ana", "ana@example.com");
session.beginTransaction();
session.persist(u);          // INSERT en flush/commit

session.getTransaction().commit();
```

### 2 Carga de entidades
- `find(Entidad.class, id)` / `getReference(...)` (proxy, no SELECT hasta acceso).
- Consultas (`createQuery`) → `getResultList()` / `getSingleResult()`.

```java
Usuario u = em.find(Usuario.class, 1L);          // SELECT inmediato
Usuario p = em.getReference(Usuario.class, 1L);  // Proxy LAZY
```

### 3 Actualización - Merge
- **Entidad detached**: usa `merge(detached)`. **Devuelve otra instancia** *managed*.

```java
// managed
Usuario u = em.find(Usuario.class, 1L);
u.setNombre("Ana María"); // UPDATE al sincronizar

// detached -> managed
Usuario dto = ...; // construido fuera
Usuario m = em.merge(dto); // m es managed, hay que hacer commit o flush para la BD
```

### 4 Borrado
- `remove(managed)` → marca **removed** → `DELETE` en flush/commit.
- Con relaciones: cuida `cascade = REMOVE` y `orphanRemoval = true` donde proceda.

```java
Usuario u = em.find(Usuario.class, 1L);
em.remove(u); // DELETE
```

### 5 `flush()`, `clear()`, estados
- `flush()` → sincroniza ahora con la BD.
- `clear()` → vacía el contexto (managed → detached).
- Estados: **transient**, **managed**, **detached**, **removed**.

### 6 Buenas prácticas
- Usa `getReference` para asociar por id sin SELECT extra.
- Helpers `add/remove` en colecciones bidireccionales (y `orphanRemoval` si aplica).
- Transacciones en servicios, no en DAOs (usa `@Transactional`).

### Mini-guía de uso (persist vs merge)

- `persist`: para nuevos (id null). Pasa la instancia a managed.
- `merge`: para detached (crea/actualiza), entidades con id. Devuelve la instancia managed (no la que le pasas).

---

## 🔎 Consultas: SQL, HQL/JPQL y Criteria API

Hibernate Query Language - [HQL Official Website](https://docs.jboss.org/hibernate/orm/6.1/userguide/html_single/Hibernate_User_Guide.html#hql)

JPQL (Java Persistence Query Language) se inspiró en las primeras versiones de HQL y es un subconjunto del HQL moderno.

HQL (y JPQL) se basan libremente en SQL y son fáciles de aprender para cualquiera que esté familiarizado con SQL.

Hibernate usa un poderoso lenguaje de consulta (HQL) que es similar en apariencia a SQL. Sin embargo, en comparación con SQL, **HQL está completamente orientado a objetos y comprende nociones como herencia, polimorfismo y asociación**.

### 🧨 1 JPQL/HQL (orientado a entidades)
- Consulta **entidades y sus atributos**, NO tablas.
- 🎐 _BINDING arguments en los parámetros de la query_. Parámetros **nombrados** `:param` o **posicionales** `?1`.
- **JOIN** sobre asociaciones del modelo.

```java title="named parameters"
//JPQL
TypedQuery<Usuario> q = em.createQuery(
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
List<UsuarioPedidoDTO> dtos = em.createQuery(
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

### 🧨 2 SQL nativo
- Útil para funciones específicas del motor o *performance*.
- Puedes mapear a **entidades**, **DTOs** (proyecciones) o **escalares**.

```java
List<Object[]> filas = em.createNativeQuery(
  "SELECT id, nombre FROM usuarios WHERE email LIKE ?")
  .setParameter(1, "%@example.com%")
  .getResultList();
```

#### 🎐 _EJECUTANDO la consulta_
La interfaz Query se utiliza para controlar la ejecución de la consulta. Ofrece tres tipos de resultados:

- `getResultList()`: devuelve una lista con los resultados (lista vacía si no hay filas). No lanza excepción por “cero resultados”..
- `getSingleResult()`: es solo para casos en los que la consulta siempre **devuelve exactamente un resultado**. Lanza excepción si no hay ninguno o si hay más de uno.
- `getSingleResultOrNull()` → devuelve el único resultado o null si no hay ninguno; sigue lanzando excepción si hay más de uno.
- `getResultStream()`: permite que los resultados se recuperen **de forma incremental**, utilizando un cursor de base de datos.

!!! warning Warning
    El método `getResultStream()` **no suele ser útil**. Casi siempre es una mala idea mantener abierto el cursor de una base de datos.

### 3 Criteria API (tipado, dinámico) [Sitio Web Oficial](https://docs.jboss.org/hibernate/orm/6.1/userguide/html_single/Hibernate_User_Guide.html#criteria)

#### 🧮 ¿Qué es y cuándo usarla?

La Criteria API permite construir consultas tipadas y dinámicas (con filtros opcionales) en tiempo de compilación. Es ideal cuando:

- La consulta se arma condicionalmente (si viene un filtro, si no, etc.).
- Quieres seguridad de tipos (sin strings JPQL).

#### 🚶 Pasos

1. Consigue el CriteriaBuilder de la session

```java
CriteriaBuilder cb = session.getCriteriaBuilder();
```

2. Crea la consulta (elige el tipo de resultado):

```java
CriteriaQuery<Usuario> cq = cb.createQuery(Usuario.class); // tipada
// o: CriteriaQuery<Tuple> cq = cb.createTupleQuery();      // tuplas
// o: CriteriaQuery<Object[]> cq = cb.createQuery(Object[].class); // arrays
```

3. Define la raíz (Root)

La raíz siempre es una entidad.

```java
Root<Usuario> u = cq.from(Usuario.class);
```

4. Define las uniones JOIN, si las necesitas

```java
Root<Reservation> root = criteria.from(Reservation.class);
Join<Reservation, Space> join = root.join("space");//nombre del objeto en la clase Reservation
```

5. Selecciona lo que quieres devolver

```java
cq.select(u); // opcional si solo hay un root, devuelve todo el objeto
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

7. Orden/paginación (opcional)

```java
cq.orderBy(cb.asc(u.get("nombre")));
TypedQuery<Usuario> q = em.createQuery(cq);
q.setFirstResult((pagina-1)*tam).setMaxResults(tam);
```

8. Ejecuta

```java
List<Usuario> resultados = em.createQuery(cq).getResultList();
```

--- 

###  📌 Consejos de consultas
- Proyecciones a **DTO** para lecturas complejas; no devuelvas entidades a la capa web.
- Indexa columnas de filtrado/ordenación.

---

## 🔐 Gestión de transacciones

Hibernate trabaja con sesiones. Abres una transacción con session.beginTransaction(), trabajas con la sesión (las operaciones quedan en el contexto de persistencia), y al commit() Hibernate hace un flush automático que envía los INSERT/UPDATE/DELETE a la BD; si hay error, haces rollback() y se descartan los cambios

- beginTransaction() inicia una transacción asociada a la Session.
- commit() hace un flush automático (emite INSERT/UPDATE/DELETE pendientes) y confirma.
- rollback() revierte todo lo hecho en la transacción si hay error.
- Usa siempre try-with-resources para cerrar la Session; no reutilices sesiones entre hilos.
- Comprueba tx.isActive() antes de hacer rollback().

```java
Transaction tx = null;
try (Session session = sessionFactory.openSession()) {
    tx = session.beginTransaction();

    // --- trabajo con la BD ---
    // session.persist(entidad);
    // session.merge(detached);
    // session.remove(entidad);
    // consultas HQL/Criteria/Native
    // --------------------------

    tx.commit();                 // hace flush implícito y confirma cambios
} catch (RuntimeException e) {
    if (tx != null && tx.isActive()) tx.rollback();  // revierte la tx
    throw e;                      // vuelve a lanzar para que la capa superior decida
}
```