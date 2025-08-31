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

try (final Connection con = DriverManager.getConnection(url, user, password);
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

try (final Connection con = DriverManager.getConnection(url, user, password);
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

## 🧾 Funciones almacenadas

Una **función almacenada** es similar a un procedimiento, pero con diferencias importantes:

- Siempre devuelve **un único valor** con `RETURN`.  
- Solo acepta parámetros de **entrada (IN)**.  
- Puede usarse **dentro de consultas SQL**, igual que las funciones nativas (`SUM`, `AVG`, `NOW`, etc.).  
- Es ideal para realizar cálculos o transformaciones sobre datos.

---

### 📘 Ejemplo en MySQL

```sql
DELIMITER //
CREATE FUNCTION calcularEdadMedia() RETURNS DECIMAL(5,2)
BEGIN
    DECLARE media DECIMAL(5,2);
    SELECT AVG(edad) INTO media FROM alumnos;
    RETURN media;
END //
DELIMITER ;
```

---

### 💻 Ejemplo en Java con CallableStatement

En JDBC se puede invocar una función almacenada usando `CallableStatement`.
La sintaxis utiliza `? = call ...` para capturar el valor devuelto.

```java
String sql = "{ ? = call calcularEdadMedia() }";

try (final Connection con = DriverManager.getConnection(url, user, password);
     CallableStatement cs = con.prepareCall(sql)) {

    // Registrar el parámetro de salida
    cs.registerOutParameter(1, java.sql.Types.DECIMAL);

    // Ejecutar
    cs.execute();

    // Recuperar el valor devuelto
    double media = cs.getDouble(1);
    System.out.println("Edad media: " + media);

} catch (SQLException e) {
    e.printStackTrace();
}
```

---

## 📌 Diferencia con procedimientos

- **Procedimiento almacenado** → puede tener IN, OUT, INOUT y devolver conjuntos de resultados.
- **Función almacenada** → solo IN, y devuelve un valor único con RETURN.

---

## 📌 Resumen

- Los **procedimientos y funciones almacenados** permiten encapsular lógica SQL en la base de datos.  
- Se ejecutan desde Java con `CallableStatement`.
- Son útiles para mejorar el rendimiento y la reutilización de código.
