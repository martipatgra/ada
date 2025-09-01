# 🧠 Mejora del rendimiento y buenas prácticas

Optimizar el acceso a BD no va solo de “hacer menos queries”, sino de **gestionar bien las conexiones** y **ejecutar SQL eficientemente**. Estos son los puntos clave:

---

## 🔌 Conexión: ¿singleton o pool?

🔹 Singleton  
- Patrón de diseño que asegura que solo exista **una instancia** de una clase en toda la aplicación.

🔹 Pool de conexiones  
- Conjunto de **múltiples conexiones abiertas** y gestionadas por un `DataSource`.

- ❌ **Evita un `Connection` “singleton”** en aplicaciones con concurrencia:
    - Una `Connection` **no es thread-safe** → cuellos de botella y errores.
    - Punto único de fallo (si cae, cae todo).
    - Conexiones largas pueden caducar o quedar “colgadas”.

- ✅ **Usa un *pool de conexiones*** (`DataSource`) en su lugar:
    - Reutiliza conexiones abiertas (muy rápido).
    - Gestiona caducidad, validación, tamaño del pool y timeouts.
    - Ideal tanto para apps de escritorio con cierta concurrencia como para aplicaciones web.

### Ejemplo con HikariCP (recomendado)

```java
import com.zaxxer.hikari.HikariConfig;
import com.zaxxer.hikari.HikariDataSource;
import javax.sql.DataSource;

public final class DataSourceSingleton {
    private static final HikariDataSource ds;

    static {
        HikariConfig cfg = new HikariConfig();
        cfg.setJdbcUrl("jdbc:mysql://localhost:3306/severo?useSSL=false");
        cfg.setUsername("patricia");
        cfg.setPassword("marti");

        // Configuración del pool
        cfg.setMaximumPoolSize(10);
        cfg.setMinimumIdle(2);
        cfg.setConnectionTimeout(10000); // ms
        cfg.setIdleTimeout(600000);      // ms
        cfg.setMaxLifetime(1800000);     // ms

        ds = new HikariDataSource(cfg);
    }

    private DataSourceSingleton() {}

    public static DataSource getDataSource() { return ds; }
}
```

- Hay un único HikariDataSource (singleton) por cada origen de datos.  
    - Ese DataSource gestiona internamente un pool de conexiones.  
    - Cada vez que llamas a `getConnection()` de `DataSource`, no abre un socket nuevo:  
       - ✅ Si hay una conexión libre en el pool → te la entrega.  
       - ✅ Si no, crea una nueva hasta llegar al maximumPoolSize.
       - ✅ Cuando cierras la conexión (try-with-resources), no se destruye, sino que vuelve al pool para reutilizarse. Ese préstamo/devolución es muy rápido y thread-safe.

Uso:
```java
try (Connection con = DataSourceSingleton.getDataSource().getConnection();
     PreparedStatement ps = con.prepareStatement("SELECT id, user_name FROM login WHERE id < ?")) {

    ps.setInt(1, 10);
    try (ResultSet rs = ps.executeQuery()) {
        while (rs.next()) {
            System.out.println(rs.getInt("id") + " - " + rs.getString("user_name"));
        }
    }
}
```

---

### 📦 ¿Cómo añadir HikariCP?

HikariCP no forma parte de Java SE, hay que añadirlo como dependencia externa.

#### Con Maven
```xml
<dependency>
    <groupId>com.zaxxer</groupId>
    <artifactId>HikariCP</artifactId>
    <version>5.1.0</version>
</dependency>
```

#### Con Gradle
```gradle
implementation 'com.zaxxer:HikariCP:5.1.0'
```

#### Manualmente
- Descargar el `.jar` de [Maven Central](https://mvnrepository.com/artifact/com.zaxxer/HikariCP).  
- Añadirlo al **classpath** del proyecto en IntelliJ/Eclipse.  
- También se necesita el **driver JDBC** de la BD (ej: `mysql-connector-j`).

---

## 🧵 Multihilo y conexiones

- Un **hilo ≠ una conexión fija**. Pide la conexión al pool **cuando la necesites** y ciérrala lo antes posible.  
- No compartas `Connection`, `Statement` o `ResultSet` entre hilos.  
- Mantén las **transacciones cortas** para reducir bloqueos.

---

## ⚙️ PreparedStatement y caché

- Usa `PreparedStatement` para casi todo tipo de SQL (también `SELECT`).  
- Previene **SQL Injection** y mejora el rendimiento gracias al **plan cache** del SGBD.
- Activa la caché de prepareds en MySQL con:  
  - `cachePrepStmts=true&prepStmtCacheSize=256&prepStmtCacheSqlLimit=2048`  

---

## 🚀 Inserciones masivas con batch

Para grandes volúmenes de datos, agrupa operaciones con `addBatch()`:

```java
String sql = "INSERT INTO login(user_name, password) VALUES (?, ?)";
try (final Connection con = DataSourceSingleton.getDataSource().getConnection();
     PreparedStatement ps = con.prepareStatement(sql)) {

    con.setAutoCommit(false);
    for (Login l : lista) {
        ps.setString(1, l.getUsername());
        ps.setString(2, l.getPassword());
        ps.addBatch();
    }
    ps.executeBatch(); // 🚀 se envía todo junto en un lote
    con.commit();
}
```

---

## 🔒 Transacciones

- Desactiva `autoCommit` solo cuando necesites agrupar operaciones.  
- Confirma con `commit()` y revierte con `rollback()` en caso de error.  
- Ejemplo:
```java
con.setAutoCommit(false);
try {
    // operaciones SQL
    con.commit();
} catch (SQLException e) {
    con.rollback();
}
```

---

## 🧹 Otras buenas prácticas

- Selecciona solo las columnas necesarias (`SELECT columna1, columna2`).
- Usa índices adecuados en el SGBD.
- Define `setQueryTimeout` para queries largas.
- Cierra siempre recursos en orden: `ResultSet` → `Statement` → `Connection`. Usa `try-with-resources`.

