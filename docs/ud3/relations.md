# 🧩 Relaciones avanzadas en Hibernate

Este documento explica de forma clara y visual los conceptos más importantes relacionados con **las relaciones entre entidades en Hibernate**, incluyendo:

- 🎯 Estrategias de carga (`fetch`)
- 🔗 Cascadas (`cascade`)
- 🧹 Eliminación de huérfanos (`orphanRemoval`)
- 🧱 Atributos disponibles en cada tipo de relación
- 🔒 Uso de `optional` y `nullable` correctamente

---

## 🎯 1. Estrategias de carga (`FetchType.LAZY` vs `FetchType.EAGER`)

Determinan **cuándo** se cargan las entidades relacionadas.

- **LAZY (perezoso):** solo se carga la entidad principal; la relación se carga **cuando se accede por primera vez** al atributo.
- **EAGER (ansioso):** la relación se carga **junto con la entidad principal**, en la misma consulta (JOIN o varias consultas internas).

| Tipo | Descripción | Ejemplo típico |
|------|--------------|----------------|
| `LAZY` | Se carga solo cuando se accede al atributo. ✅ Recomendado para la mayoría de casos. | `@OneToMany(fetch = FetchType.LAZY)` |
| `EAGER` | Se carga automáticamente junto con la entidad principal. ⚠️ Puede generar sobrecarga o el problema N+1. | `@ManyToOne(fetch = FetchType.EAGER)` |

### 🧠 Ejemplo

```java
@Entity
public class Departamento {
    @Id @GeneratedValue
    private Long id;
    private String nombre;

    @OneToMany(mappedBy = "departamento", fetch = FetchType.LAZY)
    private List<Empleado> empleados = new ArrayList<>();
}

@Entity
public class Empleado {
    @Id @GeneratedValue
    private Long id;
    private String nombre;

    @ManyToOne(fetch = FetchType.EAGER)
    @JoinColumn(name = "departamento_id")
    private Departamento departamento;
}
```

👉 Al cargar un `Departamento`, los `Empleados` **no se traen** hasta que accedemos a ellos con el método  `getEmpleados()`.
👉 Al obtener un `Empleado`, su `departamento` se carga inmediatamente.

```java
Departamento d = session.get(Departamento.class, 1L); // empleados no se cargan
System.out.println(d.getNombre());
System.out.println(d.getEmpleados().size()); // aquí Hibernate lanza otra query y carga los empleados
```

> 💡 **Consejo:** Usa `LAZY` por defecto. Solo cambia a `EAGER` si realmente necesitas los datos siempre. Un mal uso puede causar el problema **N+1 queries**.

---

## 🔗 2. Cascadas (`cascade`)

Permite que una operación sobre una entidad principal (persist, remove, merge...) se aplique también a sus entidades relacionadas.

### Tipos de `CascadeType`

| Tipo | Descripción |
|------|--------------|
| `PERSIST` | Inserta entidades relacionadas automáticamente. |
| `MERGE` | Actualiza entidades relacionadas. |
| `REMOVE` | Elimina las relacionadas. |
| `REFRESH` | Refresca las entidades relacionadas desde la BD. |
| `DETACH` | Las elimina o saca del contexto de persistencia. |
| `ALL` | Aplica todos los anteriores. |

---

### 🧠 Ejemplo

```java
@Entity
public class Pedido {
    @Id @GeneratedValue
    private Long id;
    private String cliente;

    @OneToMany(mappedBy = "pedido", cascade = CascadeType.ALL)
    private List<LineaPedido> lineas = new ArrayList<>();
}

@Entity
public class LineaPedido {
    @Id @GeneratedValue
    private Long id;
    private String producto;

    @ManyToOne
    @JoinColumn(name = "pedido_id")
    private Pedido pedido;
}
```

👉 Al guardar un `Pedido`, se guardan también sus `LineaPedido` gracias al `cascade`.

### ⚖️ Uso

```java
Pedido p = new Pedido();
p.setCliente("Juan");

LineaPedido l1 = new LineaPedido();
l1.setProducto("Teclado");
l1.setCantidad(2);
l1.setPedido(p);

p.getLineas().add(l1);

session.persist(p); // ✅ Gracias al cascade, también se guarda LineaPedido
```

---

## 🧹 3. Eliminación de huérfanos (`orphanRemoval`)

Cuando se elimina un objeto de una colección en el lado del padre, o cuando se desconecta o desvincula una relación, por ejemplo, al establecer un campo a null o al asignar una nueva entidad. Si esa colección tiene `orphanRemoval=true`, Hibernate lo borra también de la BD.

### 🧠 Ejemplo

```java
@OneToMany(mappedBy = "pedido", orphanRemoval = true)
private List<LineaPedido> lineas = new ArrayList<>();

// Uso
pedido.getLineas().remove(0); //
session.flush(); // 🧹 La línea eliminada desaparece de la BD
```

> ⚠️ **Importante:** `orphanRemoval` solo funciona con relaciones tipo `@OneToOne` o `@OneToMany`.

### ⚖️ Uso

```java
Pedido p = session.find(Pedido.class, 1L);
LineaPedido l = p.getLineas().get(0);

p.getLineas().remove(l); // ❌ se elimina de la lista de líneas del pedido
session.flush(); // 🧹 Hibernate borra la LineaPedido de la BD
```

---

## 🧱 4. Qué atributos se pueden usar en cada relación

| Relación      | mappedBy | cascade | fetch | orphanRemoval | JoinColumn              | JoinTable              | optional | nullable                  |
| ------------- | -------- | ------- | ----- | ------------- | ----------------------- | ---------------------- | -------- | ------------------------- |
| `@ManyToOne`  | ❌        | ✅       | ✅     | ❌             | ✅                       | ❌                      | ✅        | ✅                         |
| `@OneToMany`  | ✅/❌      | ✅       | ✅     | ✅             | ✅ (solo unidireccional) | ✅ (si no hay mappedBy) | ❌        | ⚠️ solo si unidireccional |
| `@OneToOne`   | ✅/❌      | ✅       | ✅     | ✅             | ✅                       | ❌                      | ✅        | ✅                         |
| `@ManyToMany` | ✅/❌      | ✅       | ✅     | ❌             | ❌                       | ✅                      | ❌        | ❌                         |


📘 **Notas rápidas**
- `mappedBy` → se usa en el lado **no propietario** (donde no está la FK).  
- `cascade` y `fetch` → se pueden usar en todas.  
- `orphanRemoval` → solo en **@OneToOne / @OneToMany**.  
- `JoinColumn` → define la **columna FK** (solo en el lado propietario).  
- `JoinTable` → se usa en relaciones **@ManyToMany** (y algunas unidireccionales).  

---

## 🔒 5. `optional` y `nullable`

- `@JoinColumn(nullable = false)` → Regla de **base de datos/DDL**: la columna FK es NOT NULL.
La validación “dura” sucede en la **BD** (ConstraintViolation) al hacer `flush` o `commit`. Asegura el esquema físico de BD.

- `@ManyToOne(optional = false)` → Regla **de modelo JPA** (runtime): esta asociación **no puede ser null**.
Le da pistas a Hibernate para **validar antes, optimizar joins y evitar selects innecesarios** con LAZY.
No garantiza que en la BD la FK sea NOT NULL.

Se aconseja su uso conjunto.

| Atributo | Capa | Significado |
|-----------|-------|-------------|
| `optional` | Hibernate (en tiempo de ejecución) | Si la referencia puede ser `null` en memoria. |
| `nullable` | Base de datos (DDL) | Si la columna FK permite `NULL`. |

> 🧩 Si la relación es obligatoria, usa **los dos**:  
> `optional = false` + `@JoinColumn(nullable = false)`

### Solo `nullable = false` sin poner `optional = false`

Persistes una entindad con la relación a `null` → Hibernate hace `INSERT ...` con `relacion_id` = null → **la BD lanza el error** (ConstraintViolation) en el `flush`. Has hecho un viaje a BD para nada y el mensaje suele ser menos expresivo.

Si hubiera puesto los dos, Hibernate **sabe que es ilegal en el modelo y puede lanzar** una `PropertyValueException` antes o durante la preparación del insert, sin depender de la BD. Mensaje más claro y te ahorras roundtrip.

### Solo `optional = false` sin poner `nullable = false`

**Modelo JPA**: marcas la relación como obligatoria; si llega `venue == null`, el proveedor fallará (normalmente al `flush`) con una excepción de JPA/Hibernate.

**Esquema de BD: no garantizas nada**. La columna puede seguir siendo NULL en la tabla si no has puesto `@JoinColumn(nullable=false)` (y no debes asumir que el proveedor de BD traducirá optional=false a NOT NULL en el DDL; no está garantizado por la especificación). 

**Integridad real**: si alguien inserta/actualiza por fuera de JPA, por ejemplo, de forma manual directamente en la BD, **podrá meter NULLs en la FK y romper tu invariante**.

**Efectos colaterales**: si acaban entrando NULLs, tus consultas que asumen relación obligatoria (INNER JOIN, filtros, etc.) pueden dar **resultados incoherentes**.

### Recomendación usar las dos `optional = false` y `nullable = false`

Garantiza:    
- Plan de consulta y rendimiento    
- Coherencia entre capas y herramientas   


### 🧠 Ejemplo obligatorio
```java
@ManyToOne(optional = false, fetch = FetchType.LAZY)
@JoinColumn(name = "departamento_id", nullable = false)
private Departamento departamento;

@ManyToOne(fetch = FetchType.LAZY, optional = false)     // Regla en el modelo JPA (runtime/optimización)
@JoinColumn(name = "venue_id", nullable = false)          // Regla física en BD (NOT NULL)
private Venue venue;
```

📌 En colecciones (`@OneToMany`, `@ManyToMany`) **no se usa `optional`**, solo `nullable` en el lado hijo si hay `JoinColumn`.

---

## ✅ 6. Resumen visual

| Concepto | Aplica a | Efecto principal | Capa |
|-----------|-----------|------------------|-------|
| `fetch` | Todas | Controla cuándo se carga la relación | ORM |
| `cascade` | Todas | Propaga operaciones (guardar, eliminar...) | ORM |
| `orphanRemoval` | OneToOne / OneToMany | Elimina hijos al quitarlos del padre | ORM |
| `optional` | OneToOne / ManyToOne | Define si la referencia puede ser null | ORM |
| `nullable` | Todas (FK) | Define si la columna permite null | BD |
| `mappedBy` | OneToOne / OneToMany / ManyToMany | Marca el lado inverso de la relación | ORM |

---

## 💥 Problema N+1 Queries en Hibernate

El **problema N+1 queries** es uno de los errores de rendimiento más comunes al usar Hibernate.  
Sucede cuando Hibernate ejecuta **una query para obtener la entidad principal** y luego **N queries adicionales**, una por cada entidad relacionada.

---

## ⚠️ ¿Qué significa “N+1”?

- `1` → la primera query para obtener la lista principal.  
- `N` → una query adicional por cada elemento para cargar sus relaciones.

**Ejemplo:**  
Si cargas 10 departamentos, Hibernate ejecutará **1 (departamentos) + 10 (empleados)** = **11 queries**.

---

## 🧩 Ejemplo clásico

```java
@Entity
public class Departamento {
    @Id @GeneratedValue
    private Long id;
    private String nombre;

    @OneToMany(mappedBy = "departamento", fetch = FetchType.LAZY)
    private List<Empleado> empleados;
}

@Entity
public class Empleado {
    @Id @GeneratedValue
    private Long id;
    private String nombre;

    @ManyToOne
    @JoinColumn(name = "departamento_id")
    private Departamento departamento;
}
```

Código en el servicio:

```java
List<Departamento> departamentos = session.createQuery(
    "FROM Departamento", Departamento.class).getResultList();

for (Departamento d : departamentos) {
    System.out.println(d.getNombre() + " → " + d.getEmpleados().size());//aquí hará una consulta para traerse el dpto.
}
```

### 🔍 Lo que pasa internamente

1️⃣ Hibernate lanza:
```sql
SELECT * FROM departamento;
```

2️⃣ Luego, por **cada departamento**:
```sql
SELECT * FROM empleado WHERE departamento_id = ?;
```

➡️ Resultado: **N+1 queries** (una por cada departamento).  
➡️ En listas grandes, esto puede ser **muy lento**.

---

## 🚀 Cómo evitar el N+1

### ✅ Opción 1: Usar `JOIN FETCH`

```java
List<Departamento> deps = session.createQuery(
    "SELECT d FROM Departamento d JOIN FETCH d.empleados",
    Departamento.class
).getResultList();
```

Hibernate genera:
```sql
SELECT d.*, e.*
FROM departamento d
LEFT JOIN empleado e ON e.departamento_id = d.id;
```

➡️ Todo se carga en una sola query.  

> 💡 Es la **solución más directa y recomendada** para evitar el problema N+1.

---

### ✅ Opción 2: Cambiar `fetch = EAGER` (con precaución)

```java
@OneToMany(mappedBy = "departamento", fetch = FetchType.EAGER)
private List<Empleado> empleados;
```

Hibernate hará automáticamente el JOIN, pero:  
⚠️ Puede generar **joins masivos e innecesarios** si hay muchas relaciones.  
Se recomienda solo cuando **siempre necesitas los datos relacionados**.

---