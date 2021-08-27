# Hibernate - Ejercicio 1

## 1 - Conexión a Base de datos
Haremos una prueba de conexión a la base de datos. Sin utilizar ficheros de configuración ni maven. Directamente con las librerías ORM de Hibernate.

 - Descargar librerías Hibernate
   - Vamos a la web oficial de Hibernate www.hibernate.com
   - Hibernate ORM -> Latest stable -> Download Zip archive
   - Utilizaremos solo las que están en lib/required
 - Descargar conector MySQL
   - https://dev.mysql.com/downloads/connector/j/
   - Seleccionamos platform independent y descargamos zip
 - Crear carpeta lib en la raiz del nuevo proyecto java
   - Añadir en carpeta librerías
     - Todas las librerias del zip que están en la ruta `hibernate-release-5.5.5.Final/lib`
     - El conector mysql
   - Configurar IntelliJ para que use estas librerías
     - File -> Project Structure...
       - Librerías
       - Modules
 - Crear paquete y clase main con conexión a la base de datos.


## 2 - Escribir datos de la base de datos
Crearemos una tabla y añadiremos datos a ella desde nuestra aplicación

 - Lo primero que vamos a hacer es crear una tabla dentro de nuestra base de datos

   ```sql
      create table clientes(
         id int primary key auto_increment,
         nombre varchar(50),
         apellidos varchar(50),
         edad int,
         direccion varchar(50)
      );
   ```

   - Comprobamos que se ha creado correctamente

    ```sql
     show tables;
    ```
   - Necesitamos un fichero de configuración para hibernate
     - Podemos buscar en google un fichero [hibernate.cfg.xml](http://www.cursohibernate.es/doku.php?id=unidades:02_hibernate:03_configurando)
     - Lo copiamos y añadimos a nuestra raiz [src](/src)
     - No vamos a necesitar las líneas de mapping puesto que mapearemos directamente en las clases en vez de en ficheros xml
     ```xml
     <?xml version="1.0" encoding="UTF-8"?>
     <!DOCTYPE hibernate-configuration PUBLIC "-//Hibernate/Hibernate Configuration DTD 3.0//EN" "http://www.hibernate.org/dtd/hibernate-configuration-3.0.dtd">
     <hibernate-configuration>
      <session-factory>
          <!-- Configurar cuatro elementos sobre la conexión de la base de datos: driverClass url nombre de usuario contraseña -->
          <property name="connection.driver_class">com.mysql.jdbc.Driver</property>
          <property name="connection.url">jdbc:mysql://localhost:3306/pruebas_hibernate?useSSL=false</property>
          <property name="connection.username">root</property>
          <property name="connection.password">root</property>
          <!-- El dialecto -->
          <property name="dialect">org.hibernate.dialect.MySQL5Dialect</property>
          <!-- Se puede mostrar la instrucción SQL enviada a la base de datos -->
          <property name="hibernate.show_sql">true</property>
          <!-- Formatear instrucción SQL-->
          <property name="hibernate.format_sql">true</property>
       </session-factory>
     </hibernate-configuration>
     ```

 - Creamos un nuevo paquete [es.eoi.ejercicios.conexionHibernate](src/es/eoi/ejercicios/conexionHibernate)
   - Creamos una clase con el mismo nombre que la tabla [Clientes.java](src/es/eoi/ejercicios/conexionHibernate/Clientes.java)
   - Creamos los campos acorde a la tabla
   - Añadimos las anotaciones de Hibernate `@Entity`, `@Table`, `@Id`, `@Column` (si hace falta), constructor, toString y los getter y setters
 - Cremos una clase que se encargará de guardar un cliente 
   - [GuardaCliente.java](src/es/eoi/ejercicios/conexionHibernate/GuardaCliente.java)
   - Con un try catch para controlar posibles errores abriremos la sesión y haremos el commit del nuevo cliente
   ```java
        public static void main(String[] args){

        SessionFactory miFactory = new Configuration()
                .configure("hibernate.cfg.xml")
                .addAnnotatedClass(Clientes.class)
                .buildSessionFactory();

        Session miSession = miFactory.openSession();
    
        Session miSession = miFactory.openSession();

        try {
            Clientes cliente1 = new Clientes("Maria", "Perez", 25, "Plaza Mayor");

            //Comenzamos la transaccion de Hibernate
            miSession.beginTransaction();

            // Guardamos el objeto de tipo cliente en nuestra base de datos
            miSession.save(cliente1);

            //Ahora o bien hacemos commit y lo persistimos en la BBDD o rollback y aquí no ha pasado nada :)
            miSession.getTransaction().commit();

            //Si ha ido bien nos mostrará el mensaje
            System.out.println("Cliente " +  cliente1.getNombre() + " almacenado en base de datos!!");

            // Lectura de registro
            miSession.getTransaction();

            System.out.println("Lectura del registro con id " + cliente1.getId()); //No veremos que es un 0, no lo estamos recuperando

            System.out.println(cliente1);

        } catch (Exception e){
            e.printStackTrace();
        } finally {
            System.out.println("Cerramos sesión y factory");
            miSession.close();
            miFactory.close();
        }
    }
   ```

 - Es el momento de ir al a base de datos y comprobar que se ha almacenado correctamente:

    ```sql
      select * from clientes;
    ```

## 3 - Leer datos de la base de datos

 - Añadimos que imprima el id y veremos que siempre es 0
 - Hay que añadir al id para que lo lea
   ```java
   @GeneratedValue(strategy = GenerationType.IDENTITY)
   ```
   - Ahora ya recupera el id que acabamos de insertar
 - Creamos una clase similar a guardaCliente pero esta vez que lea `LeeCliente.java'
   - [LeeCliente.java](src/es/eoi/ejercicios/conexionHibernate/LeeCliente.java)
   ```java
    // Leemos el objeto de tipo cliente de nuestra base de datos
    Clientes clienteLeido = miSession.get(Clientes.class, 2);
   ```


## 4 - Leer lista de clientes con hql

 - Lo primero que debemos hacer es buscar en internet "hibernate hql" y ver qué posibilidades tenemos
 - Creamos una nueva clase `LeerClientes.java` e introducimos una query hql para leer los clientes.
   - [LeerClientes.java](src/es/eoi/ejercicios/conexionHibernate/LeerClientes.java)
   ```java
   // consulta de clientes con HQL
   String hql = "from Clientes";
   List<Clientes> clientesLeidos = miSession.createQuery(hql).getResultList();

   System.out.println("- Hemos obtenido " + clientesLeidos.size() + " clientes de la base de datos!!");

   //Si ha ido bien nos mostraremos la lista de todos clientes
   for(Clientes clienteLeido: clientesLeidos) {
       System.out.println(clienteLeido);
   }

   //Ahora consulta con los que su apellido sea igual a Martinez
   hql = "from Clientes c1 where c1.apellidos='Martinez'";
   clientesLeidos = miSession.createQuery(hql).getResultList();
   System.out.println("- Hay " + clientesLeidos.size() + " con ese apellido");

   for(Clientes clienteLeido: clientesLeidos) {
       System.out.println(clienteLeido);
   }
   ```


## 5 - Eliminar varios de clientes con hql

- Creamos una nueva clase EliminarClientes, muy parecida a leerClientes pero esta vez para eliminarlos. Para hacer el eliminado más interesante borraremos sólo los que empiecen por la letra G. Introducimos una query hql para eliminar los clientes.
  - [EliminaClientes.java](src/es/eoi/ejercicios/conexionHibernate/EliminarClientes.java)
  ```java
        try {
            //Comenzamos la transaccion de Hibernate
            miSession.beginTransaction();

            // eliminación de clientes con HQL
            String hql = "delete Clientes where direccion LIKE 'G%' ";
            miSession.createQuery(hql).executeUpdate();

            miSession.getTransaction().commit();
            System.out.println("- Clientes eliminados");

        }catch (Exception e){
            e.printStackTrace();
        } finally {
            System.out.println("Cerramos sesión y factory");
            miSession.close();
            miFactory.close();
        }
  ```
