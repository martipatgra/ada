# 🧾 Ficheros JSON en Java

En esta sección nos centraremos en trabajar con ficheros **JSON** (JavaScript Object Notation) como formato ligero de intercambio de datos en Java. Aunque existen otros como XML o YAML, JSON es el **estándar actual en la mayoría de APIs** por su simplicidad, legibilidad y facilidad de uso.

---

## ✅ ¿Por qué centrarnos en JSON?

- Es **ligero**, legible y fácil de generar.
- Tiene soporte nativo en la mayoría de lenguajes de programación.
- Es ampliamente usado en **servicios web y APIs REST**.
- Hay muchas librerías Java para leer y escribir JSON.

---

## 📦 Estructura de un JSON

```json
{
  "nombre": "Juan",
  "edad": 30,
  "hobbies": ["leer", "programar", "jugar"],
  "activo": true
}
```

- Objetos: pares clave/valor delimitados por llaves `{}`.
- Arrays: listas ordenadas delimitadas por corchetes `[]`.
- Valores: cadenas, números, booleanos, null, objetos o arrays.

---

## 🧭 Tipos de datos disponibles

1. Números: Se permiten números negativos y opcionalmente pueden contener parte fraccional separada por puntos. Ejemplo: 123.456
2. Cadenas: Representan secuencias de cero o más caracteres. Se ponen entre doble comilla y se permiten cadenas de escape. Ejemplo: "Hola"
3. Booleanos: Representan valores booleanos y pueden tener dos valores: true y false
4. null: Representan el valor nulo.
5. Array: Representa una lista ordenada de cero o más valores los cuales pueden ser de cualquier tipo. Los valores se separan por comas y el vector se mete entre corchetes.
6. Objetos: Son colecciones no ordenadas de pares de la forma `nombre:valor` separados por comas y puestas entre llaves. El nombre tiene que ser una cadena entre comillas dobles. El valor puede ser de cualquier tipo.

## 🧠 Ejemplo de fichero JSON

```json
# Fichero JSON 

{"web-app": {
  "servlet": [   
    {
      "servlet-name": "cofaxCDS",
      "servlet-class": "org.cofax.cds.CDSServlet",
      "init-param": {
        "configGlossary:installationAt": "Philadelphia, PA",
        "configGlossary:adminEmail": "p.marti2@edu.gva.es",
        "configGlossary:poweredBy": "Patricia Marti",
        "configGlossary:poweredByIcon": "/images/cofax.gif",
        "configGlossary:staticPath": "/content/static",
        "templateProcessorClass": "org.cofax.WysiwygTemplate",
        "templateLoaderClass": "org.cofax.FilesTemplateLoader",
        "templatePath": "templates",
        "templateOverridePath": "",
        "defaultListTemplate": "listTemplate.htm",
        "defaultFileTemplate": "articleTemplate.htm",
        "useJSP": false,
        "dataStoreMaxConns": 100
        }
    },
    null,
    {
      "servlet-name": "cofaxEmail",
      "servlet-class": "org.cofax.cds.EmailServlet",
      "init-param": {
        "mailHost": "mail1",
        "mailHostOverride": "mail2"
      }
    }],
  "servlet-mapping": {
    "cofaxCDS": "/",
    "cofaxEmail": "/cofaxutil/aemail/*",
    "cofaxAdmin": "/admin/*",
    "fileServlet": "/static/*",
    "cofaxTools": "/tools/*"
    },
  "taglib": {
    "taglib-uri": "cofax.tld",
    "taglib-location": "/WEB-INF/tlds/cofax.tld"
    }
}
}
```

El fichero anterior se puede traducir como un array de objetos, por ejemplo de la clase `Servlet`, donde la clase `Servlet` estará compuesta por atributos como: servlet-name, servlet-class, etc.

---

## 📚 Librerías Java recomendadas

| Librería  | Ventajas principales                      |
|-----------|-------------------------------------------|
| Gson      | Simple, ligera, de Google                 |
| Jackson   | Más completa, versátil y robusta           |
| org.json  | Muy directa, sin anotaciones              |

A continuación veremos **Gson** y **Jackson**, por ser las más utilizadas.

---

## 🔧 Usando Gson (Google)

Es necesario agregar la dependencia de gson al proyecto para poder usarla.

### 1. Crear clase Java
```java
public class Persona {
    private String nombre;
    private int edad;
    private List<String> hobbies;
    private boolean activo;

    // Constructor, getters y setters
}
```

### 2. Serializar a JSON
```java
Gson gson = new Gson();
//si quiero imprimir el json en formato "bonito" (tabulado e indentado)
/*Gson gson = new GsonBuilder()
                .setPrettyPrinting()
                .create();*/

Persona p = new Persona("Ana", 25, List.of("leer", "bailar"), true);
String json = gson.toJson(p);
System.out.println(json);
```

### 3. Deserializar desde JSON
```java
String entrada = "{\"nombre\":\"Ana\",\"edad\":25," +
                  "\"hobbies\":[\"leer\",\"bailar\"],\"activo\":true}";
Persona p2 = gson.fromJson(entrada, Persona.class);
System.out.println(p2.getNombre());
```

---

## 🧰 Usando Jackson

Es necesario agregar la dependencia de jackson al proyecto para poder usarla.

### 1. Crear clase Java
```java
public class Persona {
    private String nombre;
    private int edad;
    private List<String> hobbies;
    private boolean activo;

    // Constructor, getters, setters
}
```

### 2. Serializar a JSON
```java
ObjectMapper mapper = new ObjectMapper();
Persona p = new Persona("Luis", 28, List.of("cine", "deporte"), false);
String json = mapper.writeValueAsString(p);
System.out.println(json);
```

### 3. Deserializar desde JSON
```java
String entrada = "{\"nombre\":\"Luis\",\"edad\":28," +
                  "\"hobbies\":[\"cine\",\"deporte\"],\"activo\":false}";
Persona p2 = mapper.readValue(entrada, Persona.class);
System.out.println(p2.getEdad());
```

---

## 📝 Comparativa Gson vs Jackson

| Característica          | Gson            | Jackson            |
|--------------------------|------------------|--------------------|
| Curva de aprendizaje     | Muy sencilla     | Media              |
| Velocidad                | Rápida           | Más rápida en general |
| Anotaciones avanzadas    | Limitadas        | Muy completas      |
| Soporte de formatos      | Solo JSON        | JSON, XML, YAML... |

---