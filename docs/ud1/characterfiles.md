# 📝 Operaciones con ficheros de caracteres en Java

Los **ficheros de caracteres** almacenan datos en formato de **texto**, codificados habitualmente en UTF-8. Son los más usados cuando trabajamos con logs, ficheros de configuración, resultados de programas, etc.

Para trabajar con ellos, utilizamos **flujos de caracteres** (character streams), en lugar de flujos binarios.

---

## ✍️ Clases principales para flujos de caracteres (`java.io`)

| Operación         | Clase usada               | Descripción                                  |
|-------------------|--------------------------|-----------------------------------------------|
| Lectura           | `FileReader`             | Lee caracteres desde un fichero               |
| Escritura         | `FileWriter`             | Escribe caracteres en un fichero              |
| Lectura con buffer| `BufferedReader`         | Mejora el rendimiento al leer texto           |
| Escritura con buffer| `BufferedWriter`       | Escribe texto más rápidamente               |
| Escritura formateada| `PrintWriter`          | Para escribir texto formateado (print, println) |

---

## 🗂️ Operaciones con `java.io`

### 📥 Lectura de texto (`java.io`)

#### 1. `FileReader`

```java
try (FileReader fr = new FileReader("entrada.txt")) {
    int c;
    while ((c = fr.read()) != -1) {
        System.out.print((char) c);
    }
} catch (IOException e) {
    e.printStackTrace();
}
```

- ❌ Lento para ficheros grandes. Aunque `FileReader` internamente usa un buffer en memoria, para ficheros grandes es más eficiente usar un `BufferedReader`, porque este maneja buffers más grandes y está optimizado
- ✅ Adecuado para ejemplos sencillos o pruebas

---

#### 2. `BufferedReader` (lectura eficiente)

```java
try (BufferedReader br = new BufferedReader(new FileReader("entrada.txt"))) {
    String linea;
    while ((linea = br.readLine()) != null) {
        System.out.println(linea);
    }
} catch (IOException e) {
    e.printStackTrace();
}
```

- ✅ Muy eficiente
- ✅ Ideal para lectura línea a línea

---

### ✍️ Escritura de texto (`java.io`)

#### 1. `FileWriter`

```java
try (FileWriter fw = new FileWriter("salida.txt")) {
    fw.write("Hola mundo\n");
} catch (IOException e) {
    e.printStackTrace();
}
```

- ✅ Adecuado para ejemplos sencillos o pruebas
- ❌ Menos eficiente que `BufferedWriter`. No aporta un buffer grande de caracteres como BufferedWriter. Pero si tiene un buffer/caché del sistema operativo y un buffer de codificación interno (con OutputStreamWriter), que puede retener bytes.     
- Sin **close()/flush()**, **no garantizas que todo lo escrito haya salido de esos buffers hacia el fichero**.  
- Al salir del try-with-resources, se ejecuta automáticamente `fw.close()`, que internamente hace un flush() → ahí se asegura que todo lo que estaba en memoria sí se escriba en el fichero.

---

#### 2. `BufferedWriter`

```java
try (BufferedWriter bw = new BufferedWriter(new FileWriter("salida.txt"))) {
    bw.write("Hola mundo");
    bw.newLine();
    bw.write("Otra línea");
} catch (IOException e) {
    e.printStackTrace();
}
```

- ✅ Escritura más rápida y controlada. Mantiene un buffer más grande en memoria.   
- **Solo cuando el buffer se llena o haces flush()/close() escribe en disco**.      

---

#### 3. `PrintWriter`

```java
try (PrintWriter pw = new PrintWriter("log.txt")) {
    pw.println("Inicio del log");
    pw.printf("Valor: %.2f", 12.345);
} catch (IOException e) {
    e.printStackTrace();
}
```

- ✅ Permite formateo avanzado de texto

---

## 🗂️ Operaciones con `java.nio.file.Files`

### 📥 Lectura de texto (`java.nio`)

#### 1. `Files.readAllLines()`

```java
Path ruta = Paths.get("entrada.txt");
List<String> lineas = Files.readAllLines(ruta);
```

- ✅ Muy cómodo para archivos pequeños o medianos
- ⚠️ Carga todo a memoria

---

#### 2. `Files.lines()` (stream)

Devuelve un **Stream de líneas**, útil para procesar ficheros grandes. No carga todo el fichero en memoria de golpe. Va leyendo línea a línea bajo demanda, a medida que el stream se consume.

```java
try (Stream<String> lineas = Files.lines(Paths.get("entrada.txt"))) {
    lineas.forEach(System.out::println);
}
```

- ✅ No carga todo en memoria: lectura **perezosa (lazy)**, ideal para ficheros grandes
- ✅ Puedes usar operaciones con Stream (`filter`, `map`, etc.), filtros y transformaciones
- ✅ Devuelve un Stream<String> que está respaldado por un BufferedReader interno.
- ⚠️ Necesita `try-with-resources` porque abre un recurso

---

#### 3. `Files.readString()` (Java 11+)

```java
try {
    String contenido = Files.readString(Paths.get("entrada.txt"));
    System.out.println(contenido);
} catch (IOException e) {
    e.printStackTrace();
}
```

- ✅ Ideal si necesitas todo el contenido como una sola cadena
- ✅ Muy simple y directo
- ⚠️ Carga todo a memoria

---

#### 4. `Files.newBufferedReader()`

```java
try (BufferedReader br = Files.newBufferedReader(Paths.get("entrada.txt"))) {
    String linea;
    while ((linea = br.readLine()) != null) {
        System.out.println(linea);
    }
} catch (IOException e) {
    e.printStackTrace();
}
```

- ✅ Lectura eficiente sin necesidad de envolver `FileReader`
- ✅ Permite pasar `Charset` como segundo argumento opcional, lo cual es útil si quieres controlar la codificación del texto (por ejemplo, UTF-8, ISO-8859-1, etc.)

---

### ✍️ Escritura de texto (`java.nio`)

#### 1. `Files.write(List<String>)` 

Sobrescribe por defecto el texto si no pasas opciones como `StandardOpenOption.APPEND`.

```java
List<String> contenido = List.of("Línea 1", "Línea 2");
Files.write(Paths.get("salida.txt"), contenido);

//otra opción para no sobrescribir el texto
Files.write(Paths.get("salida.txt"), contenido, StandardOpenOption.CREATE, StandardOpenOption.APPEND);
```

- ✅ Fácil de usar, sobrescribe por defecto
- ✅ Codificación UTF-8 por defecto
- ⚠️ Carga todo en memoria

---

#### 2. `Files.writeString()` (Java 11+)

```java
try {
    Files.writeString(Paths.get("salida.txt"), "Texto de ejemplo");
} catch (IOException e) {
    e.printStackTrace();
}
```

- ✅ Muy cómodo para una sola cadena
- ✅ Puedes pasar un `Charset`
- ⚠️ Carga todo en memoria

---

#### 3. `Files.newBufferedWriter()`

```java
try (BufferedWriter bw = Files.newBufferedWriter(Paths.get("salida.txt"))) {
    bw.write("Primera línea\nSegunda línea");
    bw.newLine();  // salto de línea
} catch (IOException e) {
    e.printStackTrace();
}
```

- ✅ Similar a `BufferedWriter`, pero más moderno y flexible
- ✅ Permite pasar `Charset` como segundo argumento opcional, lo cual es útil si quieres controlar la codificación del texto (por ejemplo, UTF-8, ISO-8859-1, etc.)

---

## 🧠 Tabla resumen: ¿Qué clase usar según la situación?

| Herramienta                         | ¿Cuándo usarla?                                     | ¿Buffer incluido? | ¿Requiere cerrar? | Comentario                                 |
| ----------------------------------- | --------------------------------------------------- | ----------------- | ----------------- | ------------------------------------------ |
| `BufferedReader` / `BufferedWriter` | Lectura/escritura eficiente de texto línea a línea  | ✅ Sí              | ✅ Sí              | Ideal para logs o lectura línea a línea    |
| `PrintWriter`                       | Cuando necesitas imprimir texto con formato         | ✅ Sí              | ✅ Sí              | Ofrece métodos print/println/printf        |
| `FileReader` / `FileWriter`         | Acceso directo a caracteres, sin buffer             | ❌ No              | ✅ Sí              | Puede ser ineficiente si no se envuelve    |
| `Files.readAllLines()`              | Lectura sencilla si el archivo es pequeño o mediano | ❌ No              | ❌ No              | Devuelve `List<String>`                    |
| `Files.lines()`                     | Lectura de grandes archivos como `Stream<String>`   | ✅ Sí              | ✅ Sí              | Lectura perezosa, se debe cerrar el stream |
| `Files.readString()`                | Lectura completa como cadena (Java 11+)             | ❌ No              | ❌ No              | Muy cómodo si necesitas todo el texto      |
| `Files.newBufferedReader()`         | Lectura eficiente con Charset opcional              | ✅ Sí              | ✅ Sí              | Alternativa moderna a `BufferedReader`     |
| `Files.write(List)`                 | Escritura rápida de texto si ya tienes las líneas   | ❌ No              | ❌ No              | Simple y cómodo                            |
| `Files.writeString()`               | Escritura de cadena completa (Java 11+)             | ❌ No              | ❌ No              | Muy directo y moderno                      |
| `Files.newBufferedWriter()`         | Escritura eficiente y configurable                  | ✅ Sí              | ✅ Sí              | Permite especificar codificación           |

---

## ✅ Recomendaciones

- Usa `BufferedReader` / `BufferedWriter` cuando leas o escribas muchas líneas.
- Usa `Files.readAllLines()` o `Files.write()` para tareas sencillas y ficheros pequeños.
- Usa `Files.lines()` para procesar ficheros grandes sin cargarlos enteros a memoria.
- Usa `PrintWriter` si necesitas formatear tu salida con precisión.
- Usa `Files.readString()` y `Files.writeString()` si trabajas con cadenas completas (Java 11+).
- Usa `Files.newBufferedReader()` y `Files.newBufferedWriter()` si quieres un enfoque moderno con control de codificación.

---