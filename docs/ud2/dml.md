# ✍️ Modificación de datos con JDBC

## 📘 Sentencias SQL DML

El **lenguaje de manipulación de datos (DML)** permite insertar, modificar y eliminar registros en las tablas de una base de datos.  
Las sentencias más utilizadas son:

- **INSERT** → Añade registros.  
- **UPDATE** → Modifica registros existentes.  
- **DELETE** → Elimina registros.  

Ejemplos:

```sql
INSERT INTO alumnos (id, nombre, edad) VALUES (1, 'Ana', 20);

UPDATE alumnos SET edad = 21 WHERE id = 1;

DELETE FROM alumnos WHERE id = 1;
```

---

## 💻 Uso con Statement

Podemos ejecutar sentencias DML en Java con el método `executeUpdate()` de la interfaz `Statement`. 

Ejemplo:

```java
try (Connection con = DriverManager.getConnection(url, user, password);
     Statement st = con.createStatement()) {

    String sql = "INSERT INTO alumnos (id, nombre, edad) VALUES (2, 'Luis', 22)";
    int filas = st.executeUpdate(sql);

    System.out.println("Filas insertadas: " + filas);

} catch (SQLException e) {
    e.printStackTrace();
}
```

⚠️ **No es recomendable su uso**. Por seguridad y eficiencia es mejor usar `PreparedStatement` para DML.

---

## 🛠️ Uso con PreparedStatement

`PreparedStatement` es una versión mejorada de `Statement` que permite **parametrizar sentencias SQL** mediante el uso de `?`.  
Esto evita errores, facilita la reutilización de la sentencia y protege contra inyecciones SQL.  

Ejemplo:

```java
String sql = "INSERT INTO alumnos (id, nombre, edad) VALUES (?, ?, ?)";

try (Connection con = DriverManager.getConnection(url, user, password);
     PreparedStatement ps = con.prepareStatement(sql)) {

    ps.setInt(1, 3);
    ps.setString(2, "María");
    ps.setInt(3, 19);

    int filas = ps.executeUpdate();
    System.out.println("Filas insertadas: " + filas);

} catch (SQLException e) {
    e.printStackTrace();
}
```

---

## 🔒 Prevención de SQL Injection

El uso de `PreparedStatement` es la mejor práctica para prevenir **SQL Injection**.  
Ejemplo de riesgo con `Statement`:

```java
String usuario = "pepe";
String password = "' OR '1'='1";

String sql = "SELECT * FROM usuarios WHERE user='" + usuario + "' AND pass='" + password + "'";
// Devuelve todos los usuarios, ya que la condición siempre es verdadera.
```

Con `PreparedStatement`:

```java
String sql = "SELECT * FROM usuarios WHERE user=? AND pass=?";

try (PreparedStatement ps = con.prepareStatement(sql)) {
    ps.setString(1, usuario);
    ps.setString(2, password);
    ResultSet rs = ps.executeQuery();
}
```

En este caso, los parámetros se envían al SGBD de forma segura, sin alterar la estructura de la sentencia.

---

## 📌 Resumen

- El DML (`INSERT`, `UPDATE`, `DELETE`) modifica registros de la base de datos.  
- `PreparedStatement` es preferible porque:  
  - Permite parametrizar valores.  
  - Mejora el rendimiento en ejecuciones repetidas.  
  - Previene ataques de SQL Injection.  

---
