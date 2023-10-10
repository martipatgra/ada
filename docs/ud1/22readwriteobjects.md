# Escritura y lectura de objetos en Java

Para escribir un objeto y leer esos datos directamente de vuelta a un objeto
Java nos proporciona dos clases: 
- ObjectInputStream
- ObjectOutputStream

Para entender esto correctamente es necesario saber el significado de **serialización**.

## Serialization

**El proceso de traducir una estructura de datos u objecto, a un formato que pueda ser
almacenado en un fichero**, se llama serialización. Solo las instancias de clases 
Serializables pueden ser serializadas, lo que significa que la clase debe implementar
la interfaz Serializable.

Esta interfaz no tiene ningún método, sólo se utiliza para marcar que la clase como
serializable. Todos los subtipos de una clase serializable son a su vez seriazables.

## Deserialization

La serialización por defecto escribe la clase del objeto, la firma de la clase, y los 
valores de los campos no estáticos.
Estos elementos se utilizan para restaurar el objeto y su estado durante la operación 
de escritura. A este proceso se le conoce como reconstitución de los datos o deserialización.

¿Qué pasa si cambiamos un campo de int a long y volvemos a leer?
Da error de serialUID.

### ¿Qué es el campo serialVersionUID?

El campo serialVersionUID es un campo que crea el compilador implícitamente 
en tiempo de ejecución si no se declara explícitamente, para las clases serializables.
Se basa en detalles de la clase como el número de campos, sus tipos y declaraciones.
Por tanto, cambiar un campo como hemos hecho antes, generará un UID diferente.
Cuando leemos un objeto de un stream, el runtime comprueba el serialVersionUID almacenado.
Que se almacena con el objeto escrito en el fichero .dat y lo compara con el compilado
de la clase. Si no coinciden, entonces hay un problema de compatibilidad y el runtime
lanza esa excepción de clase inválida.

También pasa mucho que diferentes compiladores pueden generar diferentes versiones de UID.
Incluso en nuevas versiones de Java también pasa. Y puede ser que no seamos capaces
de deserializar nuestros datos.

Para asegurarnos que esto no pase, es encarecidamente recomendable incluir el campo de 
serialVersionUID como un campo estático de la clase como se muestra a continuación:
private final static long serialVersionUID = 1L;
Tiene que ser long, se puede ver como una especie de número de versión de la clase.

Ahora si volvemos a probar podemos pensar que funciona, pero da otro error de tipos
incompatibles.
Aunque hayamos pasado de int a long, que en teoría cabe en conversiones. Pero tiene 
repercusiones. Por eso es importante entender las reglas de la serialización.

¿Qué puede ser un cambio incompatible? ¿Qué no me va a dejar deserializar?
1. Cambiar el tipo declarado de un campo primitivo. (Es porque los tipos de datos primitivos
toman cierta cantidad de espacio y si eso cambia cuando vayamos a leer tendremos que cambiar
cuántos bytes leemos)
2. Eliminar campos.
3. Cambiar un campo de no estático a estático.
4. Cambiar la clase de jerarquía.
5. Hay más que podemos encontrar en la documentación de java en incompatible-changes.

¿Qué cambios son compatibles con serialization-deserilization?
1. Añadir campos.
2. Cambiar el acceso a un campo. Private, public, etc.
3. Cambiar un campo de estático a no-estático. Es como añadir un campo a la clase.