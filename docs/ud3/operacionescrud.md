# 🧪 Operaciones CRUD con objetos en Hibernate

En este bloque verás cómo **cargar, guardar, actualizar y borrar** entidades con JPA/Hibernate.

---

## 🧩 1️⃣ CREATE (Insertar registros)

### 📘 `persist`
Inserta nuevas entidades en la base de datos.  
Hibernate se encarga de generar el `INSERT` SQL automáticamente.

```java
Usuario u = new Usuario("Ana", "ana@example.com");
session.beginTransaction();
session.persist(u);          // INSERT en flush/commit

session.getTransaction().commit();
```

### 🧩 Notas
- `persist()` no devuelve el ID (asigna el ID a la entidad en memoria).   
- **Marca la entidad como “managed”**, lista para sincronizar con la BD al hacer `commit()`.    
- Id **autogenerado**: no confíes en él hasta el `flush` (o tras commit).   

---

## 🧾 2️⃣ READ (Leer registros)

### 📘 `find/getReference`
Recupera entidades desde la base de datos.

Se utilizan los métodos find/getReference:      

`find(Entidad.class, id)` → **ACCESO REAL** (SELECT inmediato):       
  - Ejecuta enseguida un SELECT ... FROM customer WHERE id = 1.   
  - Devuelve el objeto completo, ya inicializado.   
  - Si no existe, devuelve null.    
  - El objeto está gestionado por el EntityManager/Session.

`getReference(...)` → **PROXY** (carga diferida / lazy, no SELECT hasta acceso):    
  - **NO ejecuta** ningún **SELECT** todavía.   
  - Devuelve un **proxy** (un objeto “fantasma” que simula ser el `Usuario`).    
  - El proxy **solo contiene el ID** y un mecanismo interno para hacer SELECT más tarde, si tocas algún campo que necesita los datos.     

```java
Usuario u = session.find(Usuario.class, 1L);          // SELECT inmediato
Usuario p = session.getReference(Usuario.class, 1L);  // Proxy LAZY
```

---

## 🔁 3️⃣ UPDATE (Actualizar registros)

### 📘 `merge` o entidad en estado managed
Modifica una entidad existente. Hibernate detecta los cambios automáticamente si la entidad está **en estado “managed”**.

- **Entidad detached**: usa `merge(detached)`. **Devuelve otra instancia** *managed*.

```java
// managed
Transaction tx = session.beginTransaction();

User user = session.find(User.class, 1L); //entidad user en estado managed
user.setEmail("nuevoemail@example.com"); // Hibernate detecta el cambio

tx.commit(); // Hibernate ejecuta el UPDATE automáticamente
```

### ⚙️ Si la entidad no está en sesión
Usa `merge()` para sincronizar los datos con el contexto:
```java
User detachedUser = new User();
detachedUser.setId(1L);
detachedUser.setEmail("nuevoemail@example.com");

session.merge(detachedUser); // sincroniza y actualiza
```

---

## ❌ 4️⃣ DELETE (Eliminar registros)

### `remove`
Borra una entidad de la base de datos.

- `remove(managed)` → marca **removed** → `DELETE` en flush/commit.

```java
Usuario u = session.find(Usuario.class, 1L);
if (u != null) {
  session.remove(u); // DELETE
}
```

### ⚠️ Precaución
- Si la entidad tiene relaciones (`@OneToMany`, etc.), el borrado puede cascadar si usas `cascade = CascadeType.REMOVE`.
- Si no existe en BD, `find()` devuelve `null` (no lanza error).

---

## `flush()`, `clear()`
- `flush()` → sincroniza ahora con la BD.
- `clear()` → vacía el contexto (managed → detached).

---

## 🔄 Ciclo de vida de una entidad

| Estado | Descripción | Ejemplo |
|---------|--------------|---------|
| **Transient** | No está asociada al contexto ni en la BD. | `new User()` |
| **Persistent (Managed)** | Asociada a una sesión activa (`persist`, `find`). | `session.persist(user)` |
| **Detached** | Ya no gestionada (sesión cerrada). | después de `session.close()` |
| **Removed** | Marcada para eliminación. | `session.remove(user)` |

---

## 🧠 Buenas prácticas
- Usa **transacciones** (`beginTransaction` / `commit`) para todas las operaciones de escritura.  
- Cierra siempre la **sesión** (`session.close()`) para liberar recursos.  
- Evita `session.flush()` manual salvo que sea necesario.  
- Usa **DAO y servicios** para encapsular la lógica CRUD.
- Usa `getReference` para asociar por id sin SELECT extra.

---

## 🧾 Conclusión
- Hibernate gestiona automáticamente las operaciones CRUD basándose en el **estado de la entidad**.  
- La mayoría de los `INSERT`, `UPDATE` y `DELETE` se ejecutan automáticamente al hacer `commit()`.  
- Usar patrones DAO/Service mantiene un código limpio, mantenible y desacoplado.

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