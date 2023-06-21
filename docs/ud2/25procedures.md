# 📘Procedimientos y funciones almacenadas

Un procedimiento almacenado es un procedimiento o subprograma que está almacenado en la base de datos.
Muchos sistemas gestores de bases de datos los soportan, por ejemplo: MySQL, Oracle, etc.

Además, estos procedimientos suelen ser de dos clases:

- [x] Procedimientos almacenados.
- [x] Funciones, las cuales devuelven un valor que se puede emplear en otras sentencias SQL.

## 📁 Procedimientos almacenados

La ejecución de procedimientos almacenados sigue la misma estructura que cualquiera de las sentencias SQL de los ejemplos anteriores, con la excepción de que usaremos la clase `CallableStatement` para representar al procedimiento y el método `execute()` de la misma para ejecutarlo.

`CallableStatement` extiende de `PreparedStatement`.

```java title="Procedure.java"
CallableStatement stmt=con.prepareCall("{call insertUser(?,?)}");  
stmt.setInt(1,1011);  
stmt.setString(2,"Patricia");  
stmt.execute(); 
```

## 📁 Funciones

En el caso de las funciones almacenadas, para su ejecución seguiremos la misma estructura que hemos visto en el caso de las consultas SQL para el caso concreto de las funciones agregadas, ya que nuestras funciones nos devolverán siempre un solo valor (o null en el caso de que no devuelvan nada).

``` sql title="Function.sql"
create or replace function sum4  
(n1 in number,n2 in number)  
return number  
is   
temp number(8);  
begin  
temp :=n1+n2;  
return temp;  
end;  
/  
```

``` java title="Sum.java"
CallableStatement stmt=con.prepareCall("{?= call sum4(?,?)}");  
stmt.setInt(2,10);  
stmt.setInt(3,43);  
stmt.registerOutParameter(1,Types.INTEGER);  //valor de retorno
stmt.execute();
  
System.out.println(stmt.getInt(1));  
```
