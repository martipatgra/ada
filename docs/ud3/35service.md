# 🔮 Capa de servicios

Es conveniente separar la lógica del código de la aplicación con las operaciones que acceden a la base de datos. Para ello, se puede crear una capa intermedia llamada **service**.

En nuestro proyecto deberemos crear un paquete llamado `service` que contendrá las clases necesarias para interactuar con los objetos DAO.

Dentro del paquete service, creamos la clase `PersonalDataService` con atributos de tipo DAO y métodos para realizar diferentes operaciones necesarias en la aplicación que además añadirán lógica de negocio.

```java title="PersonalDataService.java"
public class PersonalDataService {

    private final PersonDAO personDAO;
    private final GenericDAO<Person> personGenericDAO;

    public PersonalDataService() {
        this.personDAO = new PersonDAOImpl();
        this.personGenericDAO = new PersonDAOImpl();
    }

    public Optional<Person> getPersonById(Long id) {
        return personGenericDAO.findById(id);
    }

    public void createPerson(Person person, List<Address> addresses) {
        if (addresses != null) {
            for (Address address: addresses) {
                person.getAddresses().add(address);
            }
        }
        personDAO.create(person);//se podría usar el atributo personGenericDAO
    }
}
```

## 🔮 Testeo de la aplicación

Para probar el código simplemente tendremos que crear un objeto de la clase service y ejecutar los métodos convenientes:

```java title="Test.java"
public static void main(String[] args) {
    PersonalDataService p = new PersonalDataService();
    p.createPerson(new Person("Patricia"), new ArrayList<Address>(
              Arrays.asList(new Address("Elche","19AB"))));
    System.out.println(p.getPersonById(9L));
}
```