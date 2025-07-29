# 🥳 try-with-resources en Java

La sentencia **`try-with-resources`** es una versión especial de `try` que se usa cuando trabajamos con **recursos que deben cerrarse**, como archivos, streams, sockets o conexiones de BD.

> 🧠 Un recurso es cualquier objeto que necesita ser cerrado cuando ya no se usa, como un `FileWriter`, `Scanner` o `BufferedReader`, etc. La instrucción try-with-resources garantiza que cada recurso se cierre al final de la instrucción.

## ✅ ¿Cuándo SÍ usar try-with-resources?

Cuando trabajas con objetos que implementan `java.lang.AutoCloseable`. Esto incluye todos los objetos que implementan o extienden de la interfaz `java.io.Closeable`.

Al usar try-with-resources no sería necesario realizar la sentencia `recurso.close()`. 🔒 Java se encarga de llamar automáticamente a `.close()` por ti cuando termina el bloque `try`, **incluso si ocurre una excepción**.

Antes de Java SE 7 (antes de try-with-resources), se podía usar un bloque `finally` para asegurarse de que un recurso se cerraba, independientemente de si el try generaba excepción o no.

### Ejemplo de clases que implementan AutoCloseable:
- `BufferedReader`
- `BufferedWriter`
- `FileReader` / `FileWriter`
- `InputStream` / `OutputStream`
- `Scanner`
- `DirectoryStream`
- Conexiones JDBC (`Connection`, `Statement`, `ResultSet`)

---

## 😓 Antes de Java 7: cerrar recursos a mano

El siguiente ejemplo muestra cómo se cerraban los recursos antes de que apareciera try-with-resources. Creamos el objeto `FileWriter` en la línea 2, en vez de hacerlo en la 4 porque el `finally` de la línea 8 se encuentra en otro scope y la variable `fw` no existe.

```java hl_lines="2 4 8"
public static void main(String[] args) {
        FileWriter fw = null;
        try {
            fw = new FileWriter("prueba.txt");
            fw.write("Hola mundo");
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            try {
                if (fw != null) {
                    System.out.println("El fichero se cierra");
                    fw.close();
                }
            }catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
```
🙄 Este enfoque es largo, propenso a errores, y puede ocultar excepciones importantes.

Al hacer el `close()` manualmente, vemos que el código queda bastante engorroso, ya que la sentencia close lanza una excepción que hay que capturar.

## 🚀 Desde Java 7: try-with-resources

El mismo ejemplo usando try-with-resources. Vemos que el código queda mucho más limpio y seguro ✅:

```java
public static void main(String[] args) {
    try (FileWriter fw = new FileWriter("prueba.txt")) {
        fw.write("Hola mundo");
    } catch (IOException e) {
        e.printStackTrace();
    }
}
```

## 🧪 ¿Y si quiero usar más de un recurso?

```java
public static void main (String[] args) throws IOException {
    try(FileWriter fw = new FileWriter("prueba.txt");
        FileWriter fw2 = new FileWriter("prueba2.txt")) {
        fw.write("texto de prueba");
        fw2.write("texto de prueba");
    } catch (IOException e) {
        e.printStackTrace();
    }
}
```

!!! note "Nota 🤓"
    Podemos declarar varios recursos dentro de un mismo try como se ve en el ejemplo.

---

## ⚠️ IMPORTANTE - Diferencia clave entre try con finally y try-with-resources 🤔

Supongamos que se lanza una excepción dentro del bloque try y otra en el método close(). ¿Cuál de las dos excepciones se propaga?

### En un try con finally:

La excepción lanzada en el **finally** (al hacer close()) sobrescribe la excepción original que se lanzó primero dentro del bloque try.

Es decir:
```java
...
try {
    fw = new FileWriter("prueba.txt");
    fw.write("texto de prueba");
} catch (IOException e) {
    System.out.println("Error al escribir o crear el fichero");
} finally {
    try {
        if (fw != null) { fw.close(); }
    } catch (IOException e) {
        System.out.println("Error al cerrar el fichero");
    }
}
```
→ Se propagará **solo la excepción del **``: "Error en close".

La excepción más importante (la que causó el fallo original) **se pierde**.

### En `try-with-resources`:

Java mantiene la excepción original (la del bloque `try`) y **añade la del bloque close como excepción suprimida**.

```java
try (FileWriter fw = new FileWriter("prueba.txt")) {
    fw.write("Hola mundo");
} catch (IOException e) {
    System.out.println("Error al crear o escribir el fichero");
} // Al cerrar, también lanza IOException("Error en close")  
```

→ Se propagará **la excepción original** generada en el try (`Error al crear o escribir el fichero`), y la del `close()` quedará registrada como **"*suppressed exception*".**

✅ Esto permite que **la excepción más relevante no se pierda** y que el programador pueda acceder a toda la información posible de las causas de error.

!!! important "Conclusión clave" - `try` con `finally`: se pierde la excepción original si `close()` lanza otra. - `try-with-resources`: se mantiene la excepción del `try` y se añaden las suprimidas.

---

## 🧨 ¿Por qué debemos cerrar los recursos en Java?

Cerrar los recursos en Java (como archivos, streams, sockets o conexiones a bases de datos) es **fundamental** por varias razones:

### 1. Fugas de memoria o recursos

Cuando abres un recurso y no lo cierras, **ese recurso permanece activo en el sistema** (por ejemplo, un archivo abierto o una conexión sin liberar).

- Puede agotar el número de **descriptores de archivo** del sistema operativo.
- En programas grandes o de larga duración, esto provoca que **se queden sin recursos disponibles** y empiecen a fallar.

> Ejemplo real: abrir muchos archivos sin cerrar → `java.io.IOException: Too many open files`

### 2. Bloqueos de archivos

Si un archivo está abierto por un proceso y no se cierra, **otro proceso o hilo no podrá acceder a él** correctamente.

- En algunos sistemas operativos (como Windows), **no puedes volver a abrir el archivo si sigue abierto.**
- Esto bloquea escritura, renombrado o borrado.

### 3. Datos no guardados (buffers sin volcar)

Muchos recursos (como `BufferedWriter` o `FileOutputStream`) usan **buffers de memoria**.

- Si no cierras el recurso, el buffer podría no volcarse al disco.
- Resultado: **el archivo se queda incompleto o vacío**.

### 4. Conexiones a bases de datos colapsadas

Si no cierras una conexión JDBC (`Connection`), esta sigue ocupando espacio en el **pool de conexiones**.

- Al poco tiempo, **ya no se pueden abrir más conexiones**.
- El sistema queda bloqueado hasta reiniciar o liberar manualmente.

---

### 🧠 En resumen

| Problema si no cierras | Consecuencia           |
| ---------------------- | ---------------------- |
| Fugas de recursos      | Memoria llena, errores |
| Archivos bloqueados    | No se pueden usar      |
| Datos sin guardar      | Archivos corruptos     |
| Conexiones abiertas    | Fallos de conexión     |

---

Por eso, **try-with-resources es la forma recomendada** para evitar estos problemas automáticamente.