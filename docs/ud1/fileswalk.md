# 📁 Recorrido de directorios con Java NIO

En esta página exploramos las distintas formas modernas de recorrer directorios utilizando la API java.nio.file.

---

## 📌 ¿Por qué usar java.nio para recorrer directorios?

Java NIO ofrece una forma más **moderna, flexible y funcional** de acceder a ficheros y directorios, evitando la necesidad de estructuras clásicas más verbosas como `File`.

Sus ventajas:

- Uso de **streams** para trabajar con los ficheros como flujos de datos.
- Posibilidad de aplicar **filtros y operaciones funcionales**.
- Facilidad para recorrer subdirectorios.
- Mejor manejo de excepciones.

---

## 📚 Métodos principales de recorrido

### ✅ `Files.list(Path)`

- Lista solo los archivos y carpetas de primer nivel.
- Devuelve un `Stream<Path>`.
- ⚠️ No entra en subdirectorios.

```java
try (Stream<Path> stream = Files.list(Paths.get("miDirectorio"))) {
    stream.forEach(System.out::println);
}
```

---
### ✅ `Files.walk(Path)`

- Recorre **recursivamente** todos los subdirectorios.
- Devuelve un `Stream<Path>`.
- Admite profundidad máxima opcional.

```java
try (Stream<Path> stream = Files.walk(Paths.get("miDirectorio"))) {
    stream.forEach(System.out::println);
}
```

El método walk contempla un segundo argumento, para limitar la profundidad:

```java
Files.walk(Paths.get("miDirectorio"), 2)
```

---

### ✅ `Files.find(Path, int, BiPredicate)`

- Recorre recursivamente con **filtros personalizados**.
- 🔍 Permite especificar una función lambda como predicado que se puede usar para buscar por nombre, tamaño, tipo, fechas, etc.

```java
try (Stream<Path> stream = Files.find(
        Paths.get("miDirectorio"),
        Integer.MAX_VALUE,
        (path, attr) -> attr.isRegularFile() && path.toString().endsWith(".txt")
)) {
    stream.forEach(System.out::println);
}
```

---

### ✅ `Files.newDirectoryStream(Path)`

- Método clásico de NIO para recorrer directorios de un solo nivel.
- Devuelve un `DirectoryStream<Path>` que se puede recorrer con un bucle `for`.
- Consume poca memoria y permite aplicar filtros.

```java
try (DirectoryStream<Path> stream = Files.newDirectoryStream(Paths.get("miDirectorio"))) {
    for (Path path : stream) {
        System.out.println(path);
    }
} catch (IOException e) {
    e.printStackTrace();
}
```

También permite usar un filtro directamente:

```java
try (DirectoryStream<Path> stream = Files.newDirectoryStream(Paths.get("miDirectorio"), "*.java")) {
    for (Path path : stream) {
        System.out.println(path);
    }
}
```

---

## 🧲 Tabla comparativa de métodos

| Método        | Recursivo | Permite filtrar | Devuelve         | Comentario útil                        |
|---------------|-----------|------------------|------------------|----------------------------------------|
| `Files.list()`      | ❌ No      | ⚠️ No directo    | `Stream<Path>`   | Rápido para listar raíz                |
| `Files.walk()`      | ✅ Sí      | ✅ Sí            | `Stream<Path>`   | Perfecto para árboles de carpetas      |
| `Files.find()`      | ✅ Sí      | ✅ Muy flexible  | `Stream<Path>`   | Potente para búsquedas personalizadas  |
| `Files.newDirectoryStream()`      | ❌ No      | ✅ Sí  | `DirectoryStream<Path>`   | Alternativa clásica y eficiente  |

---

## 📎 Recomendación final

Usa:
- `Files.list()` si solo necesitas el contenido directo del directorio
- `Files.walk()` si necesitas recorrer subdirectorios
- `Files.find()` si quieres aplicar condiciones de búsqueda
- `DirectoryStream` si prefieres no usar streams o necesitas menor consumo

---

## 🧠 USO de `try-with-resources`

Los métodos anteriores devuelven objetos que **abren recursos del sistema**, como descriptores de directorios. 

**Si no los cierras correctamente, puedes provocar:**
- Fugas de memoria
- Archivos bloqueados
- Excepciones en sistemas como Windows

🔒 Por eso **se recomienda SIEMPRE** usarlos con `try-with-resources`, incluso aunque Java no te lo exija.

---