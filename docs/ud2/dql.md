# 🔎 Consultas y manipulación de resultados con JDBC (DQL)

## 📘 Sentencias SELECT

Las consultas de lectura en SQL se realizan mediante la sentencia **SELECT**.  
Ejemplos:

```sql
SELECT * FROM alumnos;

SELECT nombre, edad FROM alumnos WHERE edad > 18;

SELECT COUNT(*) FROM alumnos;
```

En JDBC, las sentencias `SELECT` se ejecutan con el método **`executeQuery()`**, que devuelve un objeto `ResultSet`.

---

## 📊 Objeto ResultSet

El objeto **`ResultSet`** representa la tabla de resultados de una consulta SQL.  
Permite recorrer fila a fila los registros devueltos por la consulta y acceder a las columnas por nombre o por índice.

Ejemplo:

```java
String sql = "SELECT id, nombre, edad FROM alumnos";

try (Connection con = DriverManager.getConnection(url, user, password);
     Statement st = con.createStatement();
     ResultSet rs = st.executeQuery(sql)) {

    while (rs.next()) {
        int id = rs.getInt("id");
        String nombre = rs.getString("nombre");
        int edad = rs.getInt("edad");

        System.out.println(id + " - " + nombre + " - " + edad);
    }

} catch (SQLException e) {
    e.printStackTrace();
}
```

---

## 🔄 Tipos de cursores

El `ResultSet` se comporta como un cursor que recorre los resultados.  

Existen distintos tipos según cómo se puedan mover, pero nosotros usaremos el **Forward-only (por defecto)**:  
- Solo permite avanzar hacia adelante (`next()`).  
- Es el más rápido y ligero.

## 🧩 Mapeo de resultados a objetos

En aplicaciones reales, los datos obtenidos con `ResultSet` suelen mapearse a **objetos de Java**.  
Esto permite trabajar de forma más natural con la información.

Ejemplo: mapeo a una clase `Alumno`.

```java
public class Alumno {
    private int id;
    private String nombre;
    private int edad;

    // Constructor, getters y setters...
}
```

Uso en la consulta:

```java
List<Alumno> lista = new ArrayList<>();

String sql = "SELECT id, nombre, edad FROM alumnos";

try (Connection con = DriverManager.getConnection(url, user, password);
     Statement st = con.createStatement();
     ResultSet rs = st.executeQuery(sql)) {

    while (rs.next()) {
        Alumno a = new Alumno();
        a.setId(rs.getInt("id"));
        a.setNombre(rs.getString("nombre"));
        a.setEdad(rs.getInt("edad"));
        lista.add(a);
    }

} catch (SQLException e) {
    e.printStackTrace();
}

System.out.println("Alumnos cargados: " + lista.size());
```

---

## 🛠️ Statement vs PreparedStatement en consultas SELECT

En las consultas DQL (`SELECT`) se pueden usar tanto `Statement` como `PreparedStatement`, pero presentan diferencias importantes:

### 📄 Statement
- Adecuado para consultas **simples y estáticas**.  
- Ejemplo:
```java
Statement st = con.createStatement();
ResultSet rs = st.executeQuery("SELECT * FROM alumnos");
```
- ✅ Ventaja: sencillo de usar en pruebas o consultas fijas.
- ❌ Inconveniente: vulnerable a SQL Injection si concatenamos parámetros en la cadena SQL.

## 🔒 PreparedStatement
- Recomendado para consultas con parámetros dinámicos.
- Ejemplo:
```java
String sql = "SELECT * FROM alumnos WHERE edad > ?";
PreparedStatement ps = con.prepareStatement(sql);
ps.setInt(1, 18);
ResultSet rs = ps.executeQuery();
```
- ✅ Ventajas:
- Previene ataques de SQL Injection.
- Mejor rendimiento en consultas repetidas (se precompila en el SGBD).
- Código más claro y mantenible.

---

## 📌 Resumen

- Las consultas SQL se realizan con `SELECT`.  
- En JDBC, se ejecutan con `executeQuery()`, que devuelve un `ResultSet`.  
- `ResultSet` permite recorrer los registros y acceder a los valores por nombre o índice.  
- Los cursores pueden ser **forward-only** o **scrollable**.  
- Es buena práctica mapear los resultados a objetos Java para facilitar el manejo de datos en la aplicación.
