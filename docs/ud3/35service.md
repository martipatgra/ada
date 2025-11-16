# 🔮 Capa de servicios

Es conveniente separar la lógica del código de la aplicación con las operaciones que acceden a la base de datos. Para ello, se puede crear una capa intermedia llamada **service**.

En nuestro proyecto deberemos crear un paquete llamado `service` que contendrá las clases necesarias para interactuar con los objetos DAO.

Dentro del paquete service, creamos la clase `PersonalDataService` con atributos de tipo DAO y métodos para realizar diferentes operaciones necesarias en la aplicación que además añadirán lógica de negocio.

```java title="PersonalDataService.java"
public class PersonService {
    private final SessionFactory sf;
    private final PersonDao personDao;

    public PersonService() {
        this.sf = HibernateUtil.getSessionFactory();
        this.personDao = new PersonDaoImpl();
    }

    public Long create(Person p) {
        Transaction tx = null;
        try {
            Session s = sf.getCurrentSession();      
            tx = s.beginTransaction();            
            personDao.saveNew(s, p);              
            tx.commit();
            return p.getId();
        } catch (PersistenceException e) {
            if (tx != null && tx.isActive()) tx.rollback();
            throw e;
        }
    }

    public Optional<Person> findById(Long id) {
        Transaction tx = null;
        try {
            Session s = sf.getCurrentSession();
            tx = s.beginTransaction();
            Optional<Person> res = personDao.findById(s, id);
            tx.commit();
            return res;
        } catch (PersistenceException e) {
            if (tx != null && tx.isActive()) tx.rollback();
            throw e;
        }
    }

    public void update(Person p) {
        Transaction tx = null;
        try {
            Session s = sf.getCurrentSession();
            tx = s.beginTransaction();
            personDao.update(s, p);
            tx.commit();
        } catch (PersistenceException e) {
            if (tx != null && tx.isActive()) tx.rollback();
            throw e;
        }
    }

    public void deleteById(Long id) {
        Transaction tx = null;
        try {
            Session s = sf.getCurrentSession();
            tx = s.beginTransaction();
            personDao.deleteById(s, id);
            tx.commit();
        } catch (PersistenceException e) {
            if (tx != null && tx.isActive()) tx.rollback();
            throw e;
        }
    }

    public void delete(Person p) {
        Transaction tx = null;
        try {
            Session s = sf.getCurrentSession();
            tx = s.beginTransaction();
            personDao.delete(s, p);
            tx.commit();
        } catch (PersistenceException e) {
            if (tx != null && tx.isActive()) tx.rollback();
            throw e;
        }
    }
}
```

- SessionFactory = se crea 1 vez al inicio, se comparte; no se crea por método.
- El service recibe el `SessionFactory` (y los DAOs) por constructor y abre/cierra la transacción por cada caso de uso.

---

## ⚙️ DAO vs Service – Estructura recomendada

### 🧩 DAO (Data Access Object)     
- 📄 Uno por entidad o agregado (SpaceDao, UserDao, BookingDao…)        
- 🎯 Responsabilidad: acceso a datos (persist, find, remove, queries).      
- 🚫 Sin transacciones ni lógica de negocio.        
- 🔄 Reutilizable desde distintos servicios.        

### 🧠 Service (Capa de negocio)        
- 🧩 Debe existir uno por caso de uso / lógica de negocio, **no necesariamente por entidad**.           
- 🎯 Responsabilidad: Agrupa y orquesta varios DAOs.         
- 💡 Define casos de uso completos, no solo operaciones CRUD.       
- 🧾 Contiene la transacción (beginTransaction / commit / rollback).        
- 🔐 Aplica validaciones y reglas de negocio.       

---

## 🔮 Testeo de la aplicación

Para probar el código simplemente tendremos que crear un objeto de la clase service y ejecutar los métodos convenientes:

```java title="Test.java"
public class Main {
  public static void main(String[] args) {
    try {
        PersonService service = new PersonService();

        // usar el servicio (abrirá transacciones con sf)
        service.create(new Person("Ana", "ana@example.com"));
        
    } catch(PersistenceException e) {
        //imprimo la excepción
    } finally {
        HibernateUtil.close(); // al terminar la app
    }
  }
}
```