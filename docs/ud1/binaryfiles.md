# 💾 Operaciones con ficheros binarios en Java

Los **ficheros binarios** almacenan datos en forma de **bytes**. Se utilizan para archivos **no de texto**, como imágenes, audio, vídeo, ficheros comprimidos, bases de datos, etc.

Para leer y escribir este tipo de ficheros, se usan **flujos de entrada/salida de bytes**.

---

## ✍️ Clases principales para flujos binarios (`java.io`)

| Operación         | Clase usada                     | Descripción                                 |
|-------------------|----------------------------------|---------------------------------------------|
| Lectura binaria   | `FileInputStream`               | Lee byte a byte desde un fichero            |
| Escritura binaria | `FileOutputStream`              | Escribe byte a byte en un fichero           |
| Lectura rápida    | `BufferedInputStream`           | Añade un búfer para mayor eficiencia        |
| Escritura rápida  | `BufferedOutputStream`          | Búfer de salida para rendimiento            |

Estas clases forman parte de la API clásica `java.io`.

---

### 📥 Lectura binaria con `FileInputStream`

```java
try (FileInputStream fis = new FileInputStream("entrada.bin")) {
    int byteLeido;
    while ((byteLeido = fis.read()) != -1) {
        System.out.println(byteLeido);
    }
} catch (IOException e) {
    e.printStackTrace();
}
```
🔎 Este código lee byte a byte. Para archivos grandes, es mejor usar un buffer.

---

### 📥 Escritura binaria con `FileOutputStream`

```java
try (FileOutputStream fos = new FileOutputStream("salida.bin")) {
    byte[] datos = {65, 66, 67}; // A, B, C
    fos.write(datos);
} catch (IOException e) {
    e.printStackTrace();
}
```
✏️ También puedes escribir byte a byte con fos.write(int b);

---

### Operaciones con buffers

Un **buffer** es una memoria intermedia que **almacena temporalmente datos** antes de leerlos o escribirlos. Esto hace que el programa:

- Haga **menos accesos al disco** (que son lentos)
- Sea **más eficiente y rápido**

Este búfer almacena los bytes antes de que se escriban en el disco. Recuerda que un búfer es un bloque de memoria que se usa para ensamblar datos antes de que se escriban todos a la vez.

Cuando se cierra el stream `close()`, es como si pincháramos en guardar de forma gráfica, es decir, que si no cerramos el stream es como si el fichero estuviera vacío. También la operación `flush()` que guarda en el fichero lo que contiene el buffer actualmente.

#### 🧱 Ejemplo escritura sin buffer

```java
try (FileOutputStream fos = new FileOutputStream("datos.bin")) {
    for (int i = 0; i < 1000; i++) {
        fos.write(i); // Se escribe en disco 1000 veces, esto es muy lento
    }
}
```
⚠️ Cada `write()` escribe directamente al disco. Muy lento.

#### ✅ Ejemplo de escritura con BufferedOutputStream (buffer)

```java
try (BufferedOutputStream bos = new BufferedOutputStream(
        new FileOutputStream("datos.bin"))) {
    for (int i = 0; i < 1000; i++) {
        bos.write(i); // Se acumulan en memoria y luego se escriben de golpe cuando se hace el close automático
    }
}
```
✅ El buffer agrupa muchos bytes y los escribe en bloque, mejorando el rendimiento. Si queremos guardar en un punto del programa podemos hacer `bos.flush()` que realiza un volcado del buffer al fichero, guarda.

#### ✅ Ejemplo de lectura con BufferedInputStream (buffer)

```java
try (BufferedInputStream bis = new BufferedInputStream(
        new FileInputStream("datos.bin"))) {
    int byteLeido;
    while ((byteLeido = bis.read()) != -1) {
        System.out.println(byteLeido);
    }
}
```
`BufferedInputStream` crea un buffer interno. Cuando llamas por primera vez a `bis.read()`, no lee 1 byte del disco, sino que:      

- Lee miles de bytes del disco de golpe (llamada costosa de E/S).
- Los guarda en memoria (en su buffer interno).
- Luego, cada llamada a `.read()` devuelve el siguiente byte del buffer, lo cual es muy rápido porque accede a memoria, no al disco.
- Cuando el buffer se vacía, se vuelve a rellenar con otros 8192 bytes del fichero.

---

### 🧻 ¿Qué son los wrappers y por qué usar buffers?

En Java, muchas clases de entrada/salida se **envuelven unas dentro de otras** para mejorar su funcionamiento. Esto se llama **patrón decorador** (decorator pattern) y permite **añadir funcionalidades** sin modificar la clase original.

La clase `BufferedInputStream`/`BufferedOuputStream` necesita siempre envolver otro `InputStream`, por lo tanto no se puede usar sola sin un wrapper.

#### 🔍 ¿Por qué?

Porque la clase `BufferedInputStream` o `BufferedOuputStream` no sabe de dónde leer/escribir datos por sí misma. Su función es añadir un búfer a otro flujo de entrada (por ejemplo, FileInputStream/FileOutputStream), pero necesita que alguien le proporcione los datos reales.

```java
try (BufferedInputStream bis = new BufferedInputStream(
        new FileInputStream("archivo.bin"))) {
    // leer datos
}
```
Aquí, `FileInputStream` accede al archivo y `BufferedInputStream` añade un buffer por encima.

---

## ✍️ Leer y escribir ficheros binarios con `java.nio.file.Files` (New I/O, desde Java 1.4)

En Java, la clase `java.nio.file.Files` ofrece formas muy sencillas y potentes de **leer y escribir ficheros** binarios o de texto. Aquí veremos las principales formas y **las diferencias entre ellas** para que puedas elegir la mejor según tu caso.

---

### 📤 Escribir en un fichero con `Files`

#### 1. `Files.write(Path path, byte[] bytes)` (todo el contenido de golpe)

Escribe todos los bytes en un fichero. **Sobrescribe** si ya existe el fichero.

```java
Path salida = Paths.get("salida.bin");
try {
    byte[] datos = {10, 20, 30, 40, 50};
    Files.write(salida, datos);
} catch (IOException e) {
    e.printStackTrace();
}
```

- ✅ Muy simple
- ⚠️ Sobrescribe el fichero por defecto

#### 3. `Files.write(Path path, byte[] bytes, OpenOption...)`

Permite **modificar el comportamiento** de la escritura. Por ejemplo, al usar la opción `StandardOpenOption.APPEND` se añade al final y no sobrescribe el fichero:

```java
Files.write(Paths.get("datos.log"), datos, StandardOpenOption.APPEND);
```

- ✅ Puedes añadir `StandardOpenOption.APPEND`, `CREATE`, `TRUNCATE_EXISTING`, etc.
- ✅ Ideal para logs u operaciones controladas

---

### 📥 Leer desde un fichero con `Files`

#### 1. `Files.readAllBytes(Path path)` (todo el contenido de golpe)

Lee el contenido del fichero como un array de bytes.

```java
Path ruta = Paths.get("salida.bin");
try {
    byte[] contenido = Files.readAllBytes(ruta);
    System.out.println("Tamaño: " + contenido.length);
    System.out.println(Arrays.toString(contenido));
} catch (IOException e) {
    e.printStackTrace();
}
```

- ✅ Ideal para ficheros **pequeños o medianos**
- ⚠️ Carga todo el archivo a memoria

---

!!! Warning
    `Files.readAllBytes()` y `Files.write()` no necesitan el uso de try-with-resources, internamente estas funciones abren y cierran automáticamente el recurso durante la operación. Pero sí debes capturar la excepción `IOException`, ya que puede producirse si el archivo no existe, no hay permisos, etc.

---
## 🧠 Tabla resumen: ¿Qué clase/método usar según la situación?

| Herramienta                          | ¿Cuándo usarla?                                                                 | ¿Buffer incluido? | ¿Requiere cerrar? | Comentario útil                                   |
|--------------------------------------|----------------------------------------------------------------------------------|--------------------|--------------------|---------------------------------------------------|
| `BufferedInputStream` / `BufferedOutputStream` | Cuando necesitas rendimiento en lectura/escritura binaria byte a byte.         | ✅ Sí               | ✅ Sí               | Útil para ficheros grandes y lectura frecuente.   |
| `FileInputStream` / `FileOutputStream` | Para acceso básico y directo a bytes.                                           | ❌ No               | ✅ Sí               | Bajo nivel, necesita envolverse si quieres buffer.|
| `Files.readAllBytes()`               | Lectura rápida de archivos binarios pequeños/medianos.                          | ❌ No               | ❌ No               | Carga todo el archivo a memoria.                  |
| `Files.write(byte[])`               | Escritura sencilla y rápida de arrays de bytes.   

---

## ✅ Conclusión

- Usa `BufferedInputStream` y `BufferedOutputStream` si vas a leer o escribir muchos datos en bucle.
- **No hace falta usar buffer** si usas métodos como `Files.readAllBytes()` o `Files.write()`, porque **ya internamente usan buffers**.
- Siempre cierra bien los recursos con `try-with-resources`.

---
