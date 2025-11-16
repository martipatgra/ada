# Patrón DAO - Data access object

Si bien es posible inyectar el acceso a la fuente de datos donde los necesitemos, no es una buena idea _-de hecho resulta horrible-_ ir repartiendo su uso por el código de nuestro proyecto sin seguir criterio alguno. Crearemos un caos que violará los principios de encapsulación y desacoplamiento de la programación orientado a objetos.

Incluso habrá ocasiones en las que necesitaremos tener más de una fuente de datos o la fuente de datos que tenemos variará, lo que nos obligaría a refactorizar gran parte del código.

La solución estándar consiste en recurrir al **patrón de diseño Data Access Object (objeto de acceso a datos), más conocido por las siglas DAO**. Las clases DAO son las responsables de implementar todas las operaciones con una fuente y\o almacenamiento de datos. Fuera de ellas, el código no tiene conocimiento sobre cómo se realiza la persistencia; puede ser una base de datos relacional o «no SQL», ficheros de texto, etcétera. 

![dao](../img/ud3/11dao.png)

Exponemos una API y todo lo demás **queda encapsulado y abstraído en los DAOs**, los cuales, generalmente, obtendremos con una factoría. Por lo común, cuando la fuente de datos es una base de datos relacional, una clase DAO contiene todas las operaciones centradas en una tabla, es decir, crearemos un DAO para cada entidad que lo requiera.

## Transacciones en la capa service, no en DAOs

El límite de la transacción (abrir, confirmar, deshacer) **debe estar en la capa de servicio** (caso de uso), no dentro de cada método DAO. Así, un único @Transactional envuelve todas las operaciones que forman el caso de uso y o bien se confirman todas, o se deshacen todas.

### ¿Por qué así?

**Un caso de uso suele tocar varios DAOs**. Si cada DAO hiciera su propia transacción, tendrías commits parciales y perderías atomicidad.

Centralizar la transacción en el servicio simplifica el DAO (solo hace CRUD) y la regla de negocio queda claramente delimitada.

El DAO debe ser **fino y reutilizable**; **no debe decidir reglas de negocio**. Encapsula acceso a datos y propaga (o traduce) errores técnicos

## 🪀 1. Creación de las interfaces DAO

Lo primero que haremos será **crear las interfaces de las entidades que requieran acceso a la base de datos**. Creamos interfaces para exponerlas en forma de API sus operaciones. Ya que la implementación de éstas estará en otras clases para encapsular las operaciones.

Usaremos los ejemplos de clase `Person` y `Address` que tenían una relación muchos a muchos.

```java title="PersonDao.java"
public interface PersonDao {

    Optional<Person> findById(Session s, Long id);

    void saveNew(Session s, Person person);

    void update(Session s, Person person);

    void deleteById(Session s, Long id);

    void delete(Session s, Person person);
}
```

!!! note "Nota"
    En el DAO suelen ir las operaciones comunes usadas para la entidad. La nomenclatura suele ser nombre de la entidad seguido de la palabra `Dao`.

## 🪀 2. Implementación de las interfaces DAO

Cada interfaz DAO tendrá su implementación. Las clases que implementan las interfaces serán nombradas como entidad + DAO + Impl: `PersonDaoImpl`.

```java title="PersonDaoImpl.java"
public class PersonDaoImpl implements PersonDao {

    @Override
    public Optional<Person> findById(Session s, Long id) {
        return Optional.ofNullable(s.get(Person.class, id));
    }

    @Override
    public void saveNew(Session s, Person person) {
        s.persist(person); // INSERT en flush/commit
    }

    @Override
    public void update(Session s, Person person) {
        s.merge(person);   // devuelve instancia managed (si la necesitas, cambia la firma a Person)
    }

    @Override
    public void deleteById(Session s, Long id) {
        Person ref = s.byId(Person.class).getReference(id); // sin SELECT
        s.remove(ref);
    }

    @Override
    public void delete(Session s, Person person) {
        if (s.contains(person)) s.remove(person);
        else s.remove(s.merge(person)); // asegura managed
    }
}
```

Si hiciéramos lo mismo para la entidad `Address`, es decir, creáramos la interfaz `AddressDao` y su implementación `AddressDaoImpl` nos daríamos cuenta de que las clases serían muy parecidas a `PersonDao` y `PersonDaoImpl`, ya que contendría los mismos métodos con la única diferencia de que cambia la entidad. Entonces estaríamos creando mucha cantidad de código redundante.

## 🪀 3. Creación de un DAO genérico

**Para mejorar la reusabilidad y legilibidad del código** deberíamos hacer uso de los genéricos que nos ofrece Java. Por tanto, se ha de crear un DAO general que incluya las funcionalidades más genéricas de los DAO, `GenericDao`.

```java title="GenericDao.java"

public interface GenericDao<T, ID extends Serializable> {

    Optional<T> findById(Session s, ID id);

    List<T> findAll(Session s);

    T saveNew(Session s, T entity);    // nuevos -> persist

    T update(Session s, T entity);     // detached/upsert -> merge (devuelve managed)

    void deleteById(Session s, ID id);

    boolean delete(Session s, T entity);

    boolean existsById(Session s, ID id);

    long count(Session s);
}
```

Todos los DAOs heredarán de `GenericDao`, lo que quiere decir que todos los DAO contendrán esos métodos, cumplirán con esas funciones.

## 🪀 4. Implementación del DAO genérico

```java title="GenericDaoHibernate.java"
public class GenericDaoHibernate<T, ID extends Serializable>
        implements GenericDao<T, ID> {

    private final Class<T> entityClass;

    public GenericDaoHibernate(Class<T> entityClass) {
        this.entityClass = entityClass;
    }

    @Override
    public Optional<T> findById(Session s, ID id) {
        return Optional.ofNullable(s.find(entityClass, id));
    }

    @Override
    public List<T> findAll(Session s) {
        String hql = "from " + entityClass.getName();
        return s.createQuery(hql, entityClass).getResultList();
    }

    @Override
    public T saveNew(Session s, T entity) {
        s.persist(entity);
        s.flush();
        return (ID) s.getIdentifier(entity);
    }

    @Override
    public T update(Session s, T entity) {
        return s.merge(entity);
    }

    @Override
    public void delete(Session s, T entity) {
        s.remove(entity);
    }

    @Override
    public boolean deleteById(Session s, ID id) {
        T found = s.find(entityClass, id);
        if (found != null) {
            s.remove(found);
            return true;
        }
        return false;
    }

    @Override
    public boolean existsById(Session s, ID id) {
        // opción rápida sin traer toda la entidad
        String hql = "select 1 from " + entityClass.getName() + " e where e.id = :id";
        Integer one = s.createQuery(hql, Integer.class)
                       .setParameter("id", id)
                       .setMaxResults(1)
                       .uniqueResult();
        return one != null;
    }

    @Override
    public long count(Session s) {
        String hql = "select count(e) from " + entityClass.getName() + " e";
        return s.createQuery(hql, Long.class).getSingleResult();
    }
}
```

Para usarlo en el service:  

```java
GenericDao<Person, Long> personDao = new GenericDaoHibernate<>(Person.class);
    personDao.saveNew(s, new Person("Ana", "ana@example.com"));
```

La creación de esta clase genérica conlleva los siguientes cambios en las clases DAO:

```java title="AddressDao.java"
public interface AddressDao extends GenericDao<Address, Long> {

}
```

```java title="AddressDaoImpl.java"
public class AddressDaoImpl extends GenericDaoHibernate<Address, Long> implements AddressDao {

    public AddressDaoImpl() {
        super(Address.class);
    }
}
```

Ahora mismo la clase `AddressDao` no definiría ningún método nuevo, solo los que ya hereda de GenericDAO. Por tanto, ¿para qué nos sirve tener esta clase? Ahora mismo, podríamos eliminarla, ya que no tiene ninguna funcionalidad extra, pero en un futuro si queremos realizar una operación muy específica, o una query relacionada con esa tabla, deberemos definir ese método dentro de `AddressDaoImpl`.
