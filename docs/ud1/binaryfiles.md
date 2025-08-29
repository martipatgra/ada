# 💾 Operaciones con ficheros binarios en Java

Los **ficheros binarios** almacenan datos en forma de **bytes**. Se utilizan para archivos **no de texto**, como imágenes, audio, vídeo, ficheros comprimidos, bases de datos, etc.

Para leer y escribir este tipo de ficheros, se usan **flujos de entrada/salida de bytes**.

---

### 🧱 Clases principales para flujos binarios (`java.io`)

| Operación         | Clase usada                     | Descripción                                 |
|-------------------|----------------------------------|---------------------------------------------|
| Lectura binaria   | `FileInputStream`               | Lee byte a byte desde un fichero            |
| Escritura binaria | `FileOutputStream`              | Escribe byte a byte en un fichero           |
| Lectura rápida    | `BufferedInputStream`           | Añade un búfer para mayor eficiencia        |
| Escritura rápida  | `BufferedOutputStream`          | Búfer de salida para rendimiento            |

Estas clases forman parte de la API clásica `java.io`.

---

## 📥 Lectura binaria con `FileInputStream`

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

## 📥 Escritura binaria con `FileOutputStream`

```java
try (FileOutputStream fos = new FileOutputStream("salida.bin")) {
    byte[] datos = {65, 66, 67}; // A, B, C
    fos.write(datos);
} catch (IOException e) {
    e.printStackTrace();
}
```
✏️ También puedes escribir byte a byte con fos.write(65);

---

## Operaciones con buffers

Un **buffer** es una memoria intermedia que **almacena temporalmente datos** antes de leerlos o escribirlos. Esto hace que el programa:

- Haga **menos accesos al disco** (que son lentos)
- Sea **más eficiente y rápido**

Este búfer almacena los bytes antes de que se escriban en el disco. Recuerda que un búfer es un bloque de memoria que se usa para ensamblar datos antes de que se escriban todos a la vez.

Cuando se cierra el stream `close()`, es como si pincháramos en guardar de forma gráfica, es decir, que si no cerramos el stream es como si el fichero estuviera vacío. También la operación flush() que guarda en el fichero lo que contiene el buffer actualmente.

### 🧱 Ejemplo escritura sin buffer

```java
try (FileOutputStream fos = new FileOutputStream("datos.bin")) {
    for (int i = 0; i < 1000; i++) {
        fos.write(i); // Se escribe en disco 1000 veces, esto es muy lento
    }
}
```
⚠️ Cada write() escribe directamente al disco. Muy lento.

### ✅ Ejemplo con BufferedOutputStream (buffer)

```java
try (BufferedOutputStream bos = new BufferedOutputStream(
        new FileOutputStream("datos.bin"))) {
    for (int i = 0; i < 1000; i++) {
        bos.write(i); // Se acumulan en memoria y luego se escriben de golpe cuando se hace el close automático
    }
}
```
✅ El buffer agrupa muchos bytes y los escribe en bloque, mejorando el rendimiento. Si queremos guardar en un punto del programa podemos hacer bos.flush() que realiza un volcado del buffer al fichero, guarda.

---



