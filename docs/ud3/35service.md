# 🔮 Capa de servicios

Es conveniente separar la lógica del código de la aplicación con las operaciones que acceden a la base de datos. Para ello, se puede crear una capa intermedia llamada **service**.

En nuestro proyecto deberemos crear un paquete llamado `service` que contendrá las clases necesarias para interactuar con los objetos DAO.

Dentro del paquete service, creamos la clase `PersonalDataService` con atributos de tipo DAO y métodos para realizar diferentes operaciones necesarias en la aplicación que además añadirán lógica de negocio.

```java title="PersonalDataService.java"
public class PersonService {
    private final SessionFactory sf;
    private final PersonDao personDao;

    public PersonService(SessionFactory sf, PersonDao personDao) {
        this.sf = sf; this.personDao = personDao;
    }

    public Long create(Person p) {
        Transaction tx = null;
        try (Session s = sf.openSession()) {      // abres la sesión aquí
            tx = s.beginTransaction();            // y la transacción aquí
            personDao.saveNew(s, p);              // pasas la sesión a los DAOs
            tx.commit();
            return p.getId();
        } catch (RuntimeException e) {
            if (tx != null && tx.isActive()) tx.rollback();
            throw e;
        }
    }

    public Optional<Person> findById(Long id) {
        Transaction tx = null;
        try (Session s = sf.openSession()) {
            tx = s.beginTransaction();
            Optional<Person> res = personDao.findById(s, id);
            tx.commit();
            return res;
        } catch (RuntimeException e) {
            if (tx != null && tx.isActive()) tx.rollback();
            throw e;
        }
    }

    public void update(Person p) {
        Transaction tx = null;
        try (Session s = sf.openSession()) {
            tx = s.beginTransaction();
            personDao.update(s, p);
            tx.commit();
        } catch (RuntimeException e) {
            if (tx != null && tx.isActive()) tx.rollback();
            throw e;
        }
    }

    public void deleteById(Long id) {
        Transaction tx = null;
        try (Session s = sf.openSession()) {
            tx = s.beginTransaction();
            personDao.deleteById(s, id);
            tx.commit();
        } catch (RuntimeException e) {
            if (tx != null && tx.isActive()) tx.rollback();
            throw e;
        }
    }

    public void delete(Person p) {
        Transaction tx = null;
        try (Session s = sf.openSession()) {
            tx = s.beginTransaction();
            personDao.delete(s, p);
            tx.commit();
        } catch (RuntimeException e) {
            if (tx != null && tx.isActive()) tx.rollback();
            throw e;
        }
    }
}
```

- SessionFactory = se crea 1 vez al inicio, se comparte; no se crea por método.
- El service recibe el `SessionFactory` (y los DAOs) por constructor y abre/cierra la transacción por cada caso de uso.

---

## 🔮 Testeo de la aplicación

Para probar el código simplemente tendremos que crear un objeto de la clase service y ejecutar los métodos convenientes:

```java title="Test.java"
public class Main {
  public static void main(String[] args) {
    SessionFactory sf = HibernateUtil.getSessionFactory();       // <-- aquí nace
    PersonDao personDao = new PersonDaoImpl(sf);                 // se inyecta
    PersonService service = new PersonService(sf, personDao);    // se inyecta

    // usar el servicio (abrirá transacciones con sf)
    service.create(new Person("Ana", "ana@example.com"));

    HibernateUtil.close(); // al terminar la app
  }
}
```