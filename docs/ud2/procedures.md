# 📘 Procedimientos almacenados y CallableStatement

## 📘 ¿Qué es un procedimiento almacenado?

Un **procedimiento almacenado** es un conjunto de sentencias SQL que se guarda en la base de datos y puede ejecutarse cuando se necesite.  
Se utiliza para:

- Reutilizar lógica de negocio en el servidor.  
- Mejorar el rendimiento (menos tráfico entre aplicación y BD).  
- Centralizar validaciones o cálculos.  

Ejemplo en MySQL:

```sql
DELIMITER //
CREATE PROCEDURE obtenerAlumnosMayores(IN edad_min INT)
BEGIN
    SELECT * FROM alumnos WHERE edad > edad_min;
END //
DELIMITER ;
```

---

## 💻 Ejecución en Java con `CallableStatement`

Para invocar un procedimiento almacenado en JDBC se utiliza la interfaz **`CallableStatement`**.  
Se obtiene a través de la conexión con el método `prepareCall()`.
Ejemplo básico:

```java
String sql = "{ call obtenerAlumnosMayores(?) }";

try (Connection con = DriverManager.getConnection(url, user, password);
     CallableStatement cs = con.prepareCall(sql)) {

    // Parámetro de entrada
    cs.setInt(1, 18);

    // Ejecutar procedimiento
    ResultSet rs = cs.executeQuery();

    while (rs.next()) {
        System.out.println(rs.getString("nombre") + " - " + rs.getInt("edad"));
    }

} catch (SQLException e) {
    e.printStackTrace();
}
```

---

### 🔄 Parámetros de entrada y salida

Los procedimientos almacenados pueden tener **parámetros de entrada (IN)**, **de salida (OUT)** o ambos (**INOUT**).  

Ejemplo de procedimiento con salida en MySQL:

```sql
DELIMITER //
CREATE PROCEDURE contarAlumnos(IN edad_min INT, OUT total INT)
BEGIN
    SELECT COUNT(*) INTO total FROM alumnos WHERE edad > edad_min;
END //
DELIMITER ;
```

Uso en Java:

```java
String sql = "{ call contarAlumnos(?, ?) }";

try (Connection con = DriverManager.getConnection(url, user, password);
     CallableStatement cs = con.prepareCall(sql)) {

    cs.setInt(1, 18); // Parámetro IN
    cs.registerOutParameter(2, java.sql.Types.INTEGER); // Parámetro OUT

    cs.execute();

    int total = cs.getInt(2);
    System.out.println("Número de alumnos mayores de 18: " + total);

} catch (SQLException e) {
    e.printStackTrace();
}
```

---

## 📌 Resumen

- Los **procedimientos almacenados** permiten encapsular lógica SQL en la base de datos.  
- Se ejecutan desde Java con `CallableStatement`.  
- Pueden tener parámetros de **entrada (IN)**, **salida (OUT)** o ambos (**INOUT**).  
- Son útiles para mejorar el rendimiento y la reutilización de código.  

---
