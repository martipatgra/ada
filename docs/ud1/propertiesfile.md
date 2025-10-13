# 📑 Ficheros de propiedades en Java

En Java es habitual guardar algunos parámetros de configuración de nuestro programa en un fichero de propiedades.

Un fichero `.properties` en Java:

- Es un fichero de **texto plano** con extensión `.properties`.
- Cada línea contiene una pareja **clave=valor**.
- Los comentarios se indican con `#`.
- Se usa para **parametrizar** aplicaciones sin cambiar el código fuente.

Java nos proporciona la clase `Properties`, para leer de forma sencilla los ficheros de configuración.

Aquí se muestra un ejemplo de fichero de configuración, llamado `datasource.properties`, que almacena información sobre la base de datos:

Ejemplo: `datasource.properties`

```properties
# Configuración de la base de datos
db.username=admin
db.password=secret
db.port=3306
```

## 📥 Cargar propiedades del fichero

Lo primero que haremos será inicializar nuestro objeto Properties. Este objeto será el contenedor en memoria de todas las claves/valores.

```java
Properties properties = new Properties();
```

Esta clase tiene un método `load()` que permite cargar el fichero. No tenemos más que pasarle un `InputStream` o un `Reader` de java.

```java
try (Reader/BufferedReader reader = Files.newBufferedReader(Paths.get("datasource.properties"))) {
    properties.load(reader); //carga el fichero en el objeto properties
} catch (IOException e) {
    e.printStackTrace();
}
```

## 📖 Leer una propiedad

El método `properties.getProperty(String)` nos permite, pasándole una clave, obtener el valor asociado a ella. En nuestro ejemplo, pasando como clave *"db.username"*, obtendríamos el valor asociado a ella *"admin"* (siempre como String, aunque sea un número).

Si la clave no existe, obtendremos `null` como resultado. Sin embargo, tenemos una variante de `getProperty()` que permite obtener un valor por defecto en caso de que no exista la clave, como en el siguiente código:

```java
properties.getProperty("db.username", "default value"));
```

Al método `getProperty()` le pasamos como primer parámetro la clave cuyo valor queremos obtener, y como segundo parámetro el valor que queremos por defecto, en caso de que la clave no tenga valor asociado.

## 🔁 Leer todas las propiedades cargadas

La clase `Properties` tiene varios métodos que nos permiten obtener todas las claves que hay en el fichero. Para ello recurriremos a un objeto `Enumeration` que nos permitirá iterar sobre ellas.

```java
Enumeration<Object> keys = properties.keys(); 
while (keys.hasMoreElements()) {
    Object key = keys.nextElement(); 
    System.out.println(key + " = "+ properties.get(key));
}

//otra forma
properties.forEach((key, value) -> System.out.println(key + " = " + value));
```

## ✍️ Añadir o modificar una propiedad (esto no modifica el fichero)

Para añadir/modificar el valor de una propiedad, la clase `Properties` tiene un método llamado `setProperty(String key, String value)` que te permite añadir una pareja clave/valor nuevas o modificar una ya existente.

```java
properties.setProperty("db.port", "4020");
```

## 📦 Guardar las propiedades en el fichero

Una vez que hemos modificado/añadido propiedades, tendremos que guardarlas en el fichero. Para ello la clase `Properties` tiene dos métodos: `save()` y `store()`. El método `save()` está obsoleto, por lo que no se aconseja su uso. Para guardar los cambios, debemos llamar a `store()` pasándole un `OutputStream` o un `Writer` de java.

```java
try (Writer/BufferedWriter writer = Files.newBufferedWriter(Paths.get("datasource.properties"))) {
    properties.store(writer, "Added database port");
} catch (IOException e) {
    //manejar excepción
}
```

El método `store()` admite un segundo parámetro que es un comentario que se añadirá como una línea de cabecera en el fichero.
