# ❓ Anotación @Query y 🆗 Response

## ❓ La anotación '@Query' tiene 4 usos

**1 Consultas personalizadas**: Puedes definir consultas personalizadas que van más allá de los métodos CRUD básicos proporcionados por Spring Data JPA. Esto es útil cuando necesitas realizar operaciones más complejas o específicas que no se cubren automáticamente.

```java
    @Query("SELECT t FROM Todo t WHERE t.completed = true")//lenguaje objeto
    List<Todo> encontrarTodosCompletados();

    @Query(value = "select t from Todo t where t.title = ?1")
    Todo findByTitleQuery(String title);
```

**2 Consultas nativas**: Puedes ejecutar consultas SQL nativas utilizando la anotación @Query. Esto es útil cuando necesitas aprovechar funcionalidades específicas del motor de base de datos que no están cubiertas por JPQL.

```java
    @Query(value = "SELECT * FROM todo WHERE title LIKE %:texto%", nativeQuery = true)
    List<Todo> encontrarPorTextoEnTitulo(String texto);

    @Query(value = "select * from todo t where t.title = ?1", nativeQuery = true)
    Todo findByTitle(String title);
```

**3 Actualizaciones y eliminaciones personalizadas**: Puedes usar @Query para definir consultas de actualización o eliminación personalizadas.

```java
    @Modifying //Indica que la consulta modificará el estado de la base de datos.
    @Transactional //Indica que la transacción debe estar activa para ejecutar esta consulta.
    @Query("UPDATE Todo t SET t.title = :nuevoTitulo WHERE t.id = :todoId")
    int updateTitleById(Long todoId, String nuevoTitulo);
```

**4 Mapeo de resultados personalizados**: Puedes especificar cómo se deben mapear los resultados de la consulta a objetos Java específicos como DTOs.

```java
    @Query("SELECT new es.severo.ud4.dto.TodoDTO(t.id, t.title) FROM Todo t WHERE t.completed = false")
    List<TodoDTO> encontrarTodosNoCompletados();
```

## 🆗 Response de una API rest

Para responder a una petición de una api de un endpoint tenemos varias formas: ResponseEntity, @ResponeStatus y @ResponseBody.

- **ResponseEntity**: es una clase que de forma flexible nos permite manejar la respuesta HTTP. Te permite controlar tanto el cuerpo de la respuesta como los encabezados y el estado HTTP. Útil con códigos personalizados

- **@ResponeStatus**: es una anotación que se puede utilizar en métodos de controlador para especificar un código de estado predeterminado para todas las respuestas generadas por ese método. Es útil cuando deseas establecer un código de estado específico para todas las respuestas sin necesidad de usar ResponseEntity.

- **@ResponseBody**: es una anotación que se puede usar para indicar que el valor de retorno  de un método de controlador debe ser serializado directamente en el cuerpo de la respuesta HTTP. Es útil cuando simplemente deseas devolver el cuerpo de la respuesta sin personalización adicional.
