# 🏗️ Definición de estructuras de BD y ejecución de sentencias DDL

## 📘 Sentencias SQL DDL

El **lenguaje de definición de datos (DDL)** permite crear, modificar y eliminar la estructura de las bases de datos.  
Las sentencias más importantes son:

- **CREATE** → Crea bases de datos, tablas, índices o vistas.  
- **ALTER** → Modifica estructuras existentes (añadir columnas, cambiar tipos, etc.).  
- **DROP** → Elimina bases de datos u objetos (tablas, índices, vistas).  

Ejemplo:

```sql
CREATE TABLE alumnos (
    id INT PRIMARY KEY,
    nombre VARCHAR(50),
    edad INT
);

ALTER TABLE alumnos ADD COLUMN email VARCHAR(100);

DROP TABLE alumnos;
```

---

## 💻 Ejecución en Java con Statement

En JDBC, las sentencias DDL se ejecutan usando el método `executeUpdate()` de la interfaz `Statement`.

```java
public class CrearTabla {
    public static void main(String[] args) {
        String url = "jdbc:mysql://localhost:3306/mi_bd";
        String user = "root";
        String password = "1234";

        String sql = "CREATE TABLE alumnos (" +
                     "id INT PRIMARY KEY," +
                     "nombre VARCHAR(50)," +
                     "edad INT)";

        try (final Connection con = DriverManager.getConnection(url, user, password);
             Statement st = con.createStatement()) {

            st.executeUpdate(sql);
            System.out.println("Tabla creada correctamente.");

        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

---

## 🗑️ Creación y borrado de tablas desde Java

Además de crear tablas, también podemos borrarlas con `DROP TABLE` desde nuestro programa Java:

- ⚠️ Atención: `DROP TABLE` borra la tabla y todos sus datos de forma permanente.

```java
String sql = "DROP TABLE alumnos";
st.executeUpdate(sql);
System.out.println("Tabla eliminada.");
```

---

### 📌 Resumen

- Las sentencias DDL (CREATE, ALTER, DROP) definen la estructura de la base de datos.
- En Java, se ejecutan mediante `Statement.executeUpdate(sql)`.
- Podemos crear o eliminar tablas directamente desde código Java.

---

## 🛠️ Statement vs PreparedStatement en DDL

- En **DDL se usa `Statement`** porque las sentencias de creación de estructuras suelen ser fijas (no llevan parámetros dinámicos). Es decir, no se le pasa ningún parámetro o variable a la consulta.
- En **DML se recomienda `PreparedStatement`**, ya que permite parametrizar (`?`), evitar inyección SQL y reutilizar la sentencia compilada.  
