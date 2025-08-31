# 📚 Fundamentos de JDBC: conexión, recursos y ejecución

Un **conector** o **driver** es un mecanismo que permite a un lenguaje de programación conectarse, y trabajar, contra una base de datos. Se encarga de mantener el diálogo con la base de datos, para poder llevar a cabo el acceso y manipulación de los datos.

---

## 📥 Carga del driver

Para que Java pueda comunicarse con una base de datos necesitamos el **driver JDBC** correspondiente.  
Ese driver se añade normalmente como un archivo `.jar` en el proyecto.

Para descargar el driver JDBC para MySQL podemos hacerlo desde el repositorio de Maven:
[MySQL JDBC](https://mvnrepository.com/artifact/com.mysql/mysql-connector-j)

En versiones antiguas de Java era necesario cargar explícitamente el driver con `Class.forName()`:

```java
Class.forName("com.mysql.cj.jdbc.Driver");
```

Hoy en día, si el driver está en el classpath, se carga automáticamente, por lo que no suele ser necesario llamar a Class.forName.

---

## 🛠️ Clase DriverManager

La clase DriverManager es la encargada de gestionar los drivers JDBC instalados y proporcionar conexiones a la base de datos.

Método más utilizado:

```java
Connection con = DriverManager.getConnection(url, usuario, contraseña);
```

Parámetros:

- **url**: cadena con el protocolo y los datos de conexión, por ejemplo  
  `jdbc:mysql://localhost:3306/mi_bd`  
- **usuario**: nombre del usuario de la base de datos.  
- **contraseña**: la clave de acceso.  

---

## 🧩 Interfaces y clases principales de JDBC

Las más utilizadas en cualquier aplicación JDBC son:

- `Connection` → Representa la conexión activa con la base de datos.
- `Statement` → Ejecuta sentencias SQL estáticas **(sin parámetros)**. Útil para DDL o consultas sencillas
- `PreparedStatement` → Versión mejorada de Statement, permite parametrizar con el símbolo "?". Más seguro (previene SQL Injection) y **eficiente en ejecuciones repetidas**.
- `CallableStatement` → Se utiliza para ejecutar **procedimientos almacenados** en la base de datos.
- `ResultSet` → Representa el **conjunto de resultados devuelto** por una consulta SELECT. Permite recorrer las filas y leer los datos columna a columna.

---

## 🔌 Objeto Connection

`Connection` representa una conexión activa con la base de datos.  
Con este objeto podremos:

- Crear sentencias (`Statement`, `PreparedStatement`, `CallableStatement`).  
- Ejecutar consultas y modificaciones.  
- Controlar transacciones (commit, rollback).  
- Cerrar la conexión cuando ya no sea necesaria.  

Ejemplo básico:

```java
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.SQLException;

...

public static void main(String[] args) {
    String user = "patricia";
    String password = "marti";
    String url = "jdbc:mysql://localhost/severo_ad";

    try (final Connection connection = DriverManager.getConnection(url, user, password)) {
        System.out.println(connection.getCatalog());
    } catch (SQLException ex) {
        System.out.println("SQLException: " + ex.getMessage());
        System.out.println("SQLState: " + ex.getSQLState());
        System.out.println("VendorError: " + ex.getErrorCode());
    }
}
```

Con `try-with-resources` la conexión se cerrará automáticamente al finalizar el bloque.

---

## 🤓 SQLException

Es la excepción que se lanza cuando ocurre un problema entre la base de datos y el programa Java JDBC.  
Algunos de sus métodos más importantes son:

- `.getMessage()` → Descripción del error.  
- `.getSQLState()` → Código SQL estándar ISO/ANSI que identifica el error. [SQLState Official](https://dev.mysql.com/doc/connector-j/8.0/en/connector-j-reference-error-sqlstates.html)  
- `.getErrorCode()` → Código específico del proveedor de BD.  
- `.getCause()` → Devuelve la excepción original que provocó el error (si existe).  
- `.getNextException()` → Permite recorrer la cadena de excepciones encadenadas.  

```java
try (final Connection con = DriverManager.getConnection(url, user, password);
     Statement st = con.createStatement()) {

    ResultSet rs = st.executeQuery("SELECT * FROM tabla_inexistente");

} catch (SQLException e) {
    System.out.println("Error: " + e.getMessage());
    System.out.println("SQLState: " + e.getSQLState());
    System.out.println("Código de error: " + e.getErrorCode());
}
```

## 🔒 Liberación de recursos en JDBC

Los objetos de JDBC (`Connection`, `Statement` y `ResultSet`) consumen recursos importantes tanto en el cliente como en el servidor.  
Por ello, deben **cerrarse siempre** cuando ya no se necesiten.  

---

### ⚡ ¿Por qué cerrarlos?

- `Connection` mantiene la conexión abierta con el SGBD y consume recursos del servidor.  
- `Statement` almacena información de ejecución y mantiene cursores abiertos.  
- `ResultSet` ocupa memoria y punteros sobre los resultados de una consulta.  

Si no se cierran manualmente:  
- El **garbage collector** los liberará más tarde, pero mientras tanto siguen consumiendo recursos.  
- Se pueden producir **fugas de memoria** o saturar el servidor de BD con conexiones abiertas.

---

### 📑 Orden recomendado de cierre

1. `ResultSet`  
2. `Statement`  
3. `Connection`  

⚠️ Si se cierra la conexión antes que el statement, al intentar cerrarlo después lanzará una `SQLException`.  

Además:  
- Cerrar un `Statement` cierra automáticamente su `ResultSet` asociado.  
- Pero cerrar una `Connection` **no** cierra automáticamente todos los statements.

---

```java
public class CierreRecursos {
    public static void main(String[] args) {
        String url = "jdbc:mysql://localhost:3306/mi_bd";
        String user = "root";
        String password = "1234";

        try (final Connection con = DriverManager.getConnection(url, user, password);
             Statement st = con.createStatement();) {//al cerrar el Statement se cierra automáticamente el ResultSet
            
            ResultSet rs = st.executeQuery("SELECT id, nombre FROM alumnos");
            while (rs.next()) {
                System.out.println(rs.getInt("id") + " - " + rs.getString("nombre"));
            }

        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
}
```

---

## ⚡ Métodos principales para ejecutar SQL en JDBC

En JDBC disponemos de tres métodos principales para ejecutar sentencias SQL desde las interfaces `Statement` o `PreparedStatement`.  
Cada uno está pensado para un tipo distinto de operación:

---

### 🔍 `executeQuery(String sql)`

- Uso: **consultas de lectura (SELECT)**.  
- Devuelve: un objeto `ResultSet` con las filas obtenidas.  
- Ejemplo:

```java
ResultSet rs = st.executeQuery("SELECT * FROM alumnos");
while (rs.next()) {
    System.out.println(rs.getString("nombre"));
}
```

---

### ✍️ `executeUpdate(String sql)`

- Uso: **operaciones de modificación de datos (DML)** y **sentencias DDL**.  
  - DML: `INSERT`, `UPDATE`, `DELETE`.  
  - DDL: `CREATE`, `ALTER`, `DROP`.  
- Devuelve: un `int` con el número de filas afectadas (en DML) o `0` en DDL.  
- Ejemplo:

```java
int filas = st.executeUpdate("INSERT INTO alumnos VALUES (1, 'Ana', 20)");
System.out.println("Filas insertadas: " + filas);
```

---

### 🌀 `execute(String sql)`

- Uso: **genérico**. Puede ejecutar cualquier tipo de sentencia SQL.  
- Devuelve:  
  - `true` → si la operación devuelve un `ResultSet`.  
  - `false` → si devuelve solo un número de filas afectadas.  
- Ejemplo:

```java
boolean tieneResultados = st.execute("SELECT * FROM alumnos");
if (tieneResultados) {
    ResultSet rs = st.getResultSet();
    while (rs.next()) {
        System.out.println(rs.getString("nombre"));
    }
} else {
    int filas = st.getUpdateCount();
    System.out.println("Filas modificadas: " + filas);
}
```

---

### 📌 Resumen de uso

| Método              | Uso principal                         | Devuelve                       |
|---------------------|---------------------------------------|--------------------------------|
| `executeQuery(sql)` | Consultas `SELECT`                    | `ResultSet`                    |
| `executeUpdate(sql)`| `INSERT`, `UPDATE`, `DELETE`, DDL     | Nº filas afectadas (`int`)     |
| `execute(sql)`      | Cualquier sentencia (menos habitual)  | `boolean` + `ResultSet`/`int`  |
