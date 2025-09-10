# 🔹 Configuración e instalación de Hibernate

En este bloque aprenderemos a **instalar y configurar Hibernate**, además de explorar dos formas de definir el mapeo: mediante **ficheros XML** y mediante **anotaciones en las clases Java**.

---

## ⚙️ 1.Instalación y configuración de Hibernate

### Dependencias
La forma más habitual es incluir Hibernate y JPA en un proyecto **Maven** o **Gradle**.

**Ejemplo con Maven (`pom.xml`):**

```xml
<dependencies>
<dependency>
  <groupId>org.hibernate.orm</groupId>
  <artifactId>hibernate-core</artifactId>
  <version>6.5.2.Final</version>
</dependency>

<dependency>
  <groupId>jakarta.persistence</groupId>
  <artifactId>jakarta.persistence-api</artifactId>
  <version>3.1.0</version>
</dependency>

<!-- MySQL JDBC Driver -->
<dependency>
    <groupId>com.mysql</groupId>
    <artifactId>mysql-connector-j</artifactId>
    <version>8.4.0</version>
</dependency>

<!-- Logging sencillo para ver SQL en consola (opcional en demos) -->
<dependency>
  <groupId>org.slf4j</groupId>
  <artifactId>slf4j-simple</artifactId>
  <version>2.0.13</version>
</dependency>
</dependencies>
```

### 2. Configuración mediante `hibernate.cfg.xml`

El fichero debe estar en el **classpath** (Maven/Gradle los copian a `target/classes/`).
El fichero .cfg.xml debe crearse y hibernate debe colocarse en el classpath del proyecto, en:

- **Hibernate (clásico)** → `src/main/resources/hibernate.cfg.xml`  

Es el fichero principal de configuración de Hibernate cuando trabajas sin JPA puro.
En él defines:  
- Conexión a la base de datos (driver, URL, usuario, contraseña).
- Dialecto de Hibernate.
- Estrategia de creación/actualización del esquema (hbm2ddl.auto).
- Otras propiedades opcionales (show_sql, format_sql, etc.).

```xml
<?xml version='1.0' encoding='utf-8'?>
<!DOCTYPE hibernate-configuration PUBLIC
        "-//Hibernate/Hibernate Configuration DTD 3.0//EN"
        "http://www.hibernate.org/dtd/hibernate-configuration-3.0.dtd">

<hibernate-configuration>
  <session-factory>
    <!-- JDBC -->
    <property name="hibernate.connection.driver_class">com.mysql.cj.jdbc.Driver</property>
    <property name="hibernate.connection.url">jdbc:mysql://localhost:3306/mibd</property>
    <property name="hibernate.connection.username">patri</property>
    <property name="hibernate.connection.password">1234</property>

    <!-- Dialecto -->
    <property name="hibernate.dialect">org.hibernate.dialect.MySQLDialect</property>

    <!-- Generación de esquema (solo desarrollo) -->
    <property name="hibernate.hbm2ddl.auto">update</property>

    <!-- Logging de SQL (para aprender / depurar) -->
    <property name="hibernate.show_sql">true</property>
    <property name="hibernate.format_sql">true</property>

    <!-- Zona horaria consistente -->
    <property name="hibernate.jdbc.time_zone">UTC</property>

    <!-- Pool de conexiones (HikariCP) -->
    <property name="hibernate.connection.provider_class">
      org.hibernate.hikaricp.internal.HikariCPConnectionProvider
    </property>
    <property name="hibernate.hikari.maximumPoolSize">10</property>

    <!-- Entidades anotadas -->
    <mapping class="com.ejemplo.Usuario"/>
  </session-factory>
</hibernate-configuration>
```

## 🏷️ Mapeo basado en anotaciones

El mapeo basado en anotaciones consiste en definir la correspondencia clase ↔ tabla y atributo ↔ columna directamente en el código Java usando anotaciones de JPA/Jakarta

### 📄 1 Entidad básica

- `@Entity` → Marca la clase como persistente.
- `@Table(name="...")` → Cambia el nombre de la tabla y permite uniqueConstraints/indexes.

Identidad:   
- `@Id` → Clave primaria.
- `@GeneratedValue(strategy=…)` → Estrategia de generación:  
    - IDENTITY (auto-increment en MySQL),
    - SEQUENCE (PostgreSQL/Oracle; suele usarse con @SequenceGenerator),
    - AUTO (deja que el proveedor decida),
    - TABLE (tabla de secuencias).

Mapeo de columnas:  
- `@Column(name="...", nullable=…, length=…, unique=…)` → Personaliza la columna.
- `@Transient` → No persistir el atributo.
- `@Enumerated(EnumType.STRING)` → Guarda el nombre del enum (mejor que ORDINAL).
- `@Lob` → CLOB/BLOB para textos o binarios grandes.

```java
@Entity 
@Table(name = "usuarios")
public class Usuario {
  @Id 
  @GeneratedValue(strategy = GenerationType.IDENTITY) // MySQL
  private Long id;

  @Column(nullable = false, length = 100)
  private String nombre;

  @Column(nullable = false, unique = true, length = 150)
  private String email;
}
```

### 2 Relaciones
**N:1 (muchos-a-uno)**
```java
@ManyToOne(fetch = FetchType.LAZY, optional = false)
@JoinColumn(name = "usuario_id", nullable = false)
private Usuario usuario;
```

**1:N (uno-a-muchos)**
```java
@OneToMany(mappedBy = "usuario", cascade = CascadeType.ALL, orphanRemoval = true)
private List<Pedido> pedidos = new ArrayList<>();
```

**1:1 (uno-a-uno)**
```java
@OneToOne(fetch = FetchType.LAZY, optional = false)
@JoinColumn(name = "perfil_id", unique = true)
private Perfil perfil;
```

**N:N (muchos-a-muchos) sin atributos en la relación**
```java
@ManyToMany
@JoinTable(name = "usuarios_roles",
  joinColumns = @JoinColumn(name = "usuario_id"),
  inverseJoinColumns = @JoinColumn(name = "rol_id"))
private Set<Rol> roles = new HashSet<>();
```

**N:N (muchos-a-muchos) con atributos que salen de la relación**
```java
@Entity @Table(name = "cursos")
public class Curso {
  @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
  private Long id;

  private String titulo;

  @OneToMany(mappedBy = "curso", cascade = CascadeType.ALL, orphanRemoval = true)
  private Set<Matricula> matriculas = new HashSet<>();
}
@Entity @Table(name = "alumnos")
public class Alumno {
  @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
  private Long id;

  private String nombre;

  @OneToMany(mappedBy = "alumno", cascade = CascadeType.ALL, orphanRemoval = true)
  private Set<Matricula> matriculas = new HashSet<>();
}

@Embeddable
public class MatriculaId implements Serializable {
  private Long alumnoId;
  private Long cursoId;
}
@Entity @Table(name = "matriculas")
public class Matricula {

  @EmbeddedId
  private MatriculaId id;

  @ManyToOne(fetch = FetchType.LAZY)
  @MapsId("alumnoId") // usa alumno.id para rellenar la parte de la PK
  @JoinColumn(name = "alumno_id", nullable = false)
  private Alumno alumno;

  @ManyToOne(fetch = FetchType.LAZY)
  @MapsId("cursoId") // usa curso.id para rellenar la parte de la PK
  @JoinColumn(name = "curso_id", nullable = false)
  private Curso curso;
}
```

### `fetch (LAZY vs EAGER)`

Controla cuándo se cargan las relaciones.

LAZY: no se carga la relación hasta que la accedes (proxy).  
- ✔️ Ahorra queries y memoria.
- ⚠️ Si accedes fuera de la sesión/EM: LazyInitializationException.

EAGER: se carga ya junto con la entidad principal.  
- ✔️ Cómodo si siempre lo necesitas.
- ⚠️ Puede disparar N+1 consultas o traer “medio grafo” sin querer.

### `orphanRemoval`

Hace que se elimine de la BD el “hijo” cuando deja de estar asociado al “padre” (se queda “huérfano” en la relación). Útil en agregados y cuando el hijo no tiene sentido sin el padre.
Funciona con @OneToOne y @OneToMany (no con @ManyToMany).

Diferente de cascade = REMOVE:  
- cascade REMOVE borra hijos cuando borras el padre.
- orphanRemoval = true borra el hijo cuando lo quitas de la colección o rompes la referencia, aunque el padre siga existiendo.

### `cascade`

Indica qué operaciones del EntityManager se propagan desde una entidad a las relacionadas (en la dirección de la relación donde lo declaras).

Tipos más usados:   
- `PERSIST` → al guardar el padre, también guarda los hijos nuevos.
- `MERGE` → al hacer merge del padre, aplica cambios a los hijos.
- `REMOVE` → al borrar el padre, borra los hijos.
- `REFRESH` → recarga también los hijos desde BD.
- `DETACH` → los hijos pasan a detached cuando el padre se separa.
- `ALL` → equivale a todas las anteriores.

### 📌 Buenas prácticas
- Prefiere **`LAZY`** y controla cargas con `JOIN FETCH`/entity graphs.  
- Define **índices** en FKs/columnas de búsqueda.  
- Evita bidireccionales innecesarias; no metas colecciones en `equals/hashCode`.  
- Usa **DTOs** para lecturas complejas; no expongas entidades en controladores.
- La **PK** (id) identifica a la entidad en BD.
- Para `equals()`/`hashCode()`:
  - Si tienes una **clave natural inmutable** (p.ej., `dni`, `email` único), puedes usarla.
  - Si usas **PK autogenerada**, no la uses antes de existir; una estrategia común es:
    - Si `id == null`, usa identidad de objeto (super) o un UUID efímero.
    - Si `id != null`, compara por `id`.

> Objetivo: que un objeto no “cambie de identidad” dentro de colecciones (`Set`, `Map`) al persistirlo.

- `@ManyToOne`, `@OneToMany`, `@OneToOne`, `@ManyToMany` con `@JoinColumn`/`@JoinTable`.
- **Dueño** de la relación = quien **tiene la FK**.
- **Colecciones**: por defecto usa `Set` (sin duplicados). Usa `List + @OrderColumn` si el **orden es parte del negocio**.
- Helpers para mantener la bidireccionalidad:
  ```java
  public void addPedido(Pedido p){ pedidos.add(p); p.setUsuario(this); }
  public void removePedido(Pedido p){ pedidos.remove(p); p.setUsuario(null); }
  ```

---
