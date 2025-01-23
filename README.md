**JDBC (Java Database Connectivity)** es una API de Java que permite conectar aplicaciones a bases de datos relacionales. Proporciona un conjunto estándar de interfaces y clases que se utilizan para interactuar con bases de datos, enviar consultas SQL y procesar los resultados.

### Componentes principales de JDBC:
1. **Driver Manager**: Gestiona una lista de controladores de bases de datos y establece conexiones con bases de datos solicitadas.
2. **Driver**: Un controlador que implementa la interfaz JDBC para interactuar con una base de datos específica.
3. **Connection**: Representa una conexión activa con una base de datos.
4. **Statement**: Permite ejecutar consultas SQL estáticas.
5. **PreparedStatement**: Permite ejecutar consultas SQL parametrizadas, aumentando la seguridad y eficiencia.
6. **ResultSet**: Maneja los datos resultantes de una consulta SQL.
7. **SQLException**: Maneja excepciones relacionadas con la interacción con bases de datos.

---

### Librerías de Scala para conectarse a bases de datos relacionales

| **Librería**       | **Descripción**                                                                 | **Diferencias**                                                                                 |
|---------------------|---------------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------|
| **Slick**          | Un marco de trabajo funcional para trabajar con bases de datos relacionales en Scala. Soporta generación de consultas tipo DSL. | Enfocado en la abstracción funcional de las bases de datos. Proporciona una capa de alto nivel. |
| **Doobie**         | Una librería pura y funcional para trabajar con JDBC en Scala, compatible con Cats. | Se enfoca en control manual sobre JDBC, ofreciendo flexibilidad para personalizar consultas.   |

---

### Documentación: Establecer conexión a MySQL desde Scala

#### Paso 1: Generar una base de datos en MySQL
Ejecute los siguientes comandos SQL en MySQL:
```sql
CREATE DATABASE prueba_scala;
USE prueba_scala;

CREATE TABLE usuarios (
    id INT AUTO_INCREMENT PRIMARY KEY,
    nombre VARCHAR(100) NOT NULL,
    edad INT NOT NULL
);

INSERT INTO usuarios (nombre, edad) VALUES 
('Alice', 25),
('Bob', 30),
('Charlie', 22);
```

#### Paso 2: Configurar el entorno en Scala
Agregue las dependencias necesarias en su archivo `build.sbt`:
```scala
libraryDependencies ++= Seq(
  "mysql" % "mysql-connector-java" % "8.0.34",
  "org.tpolecat" %% "doobie-core" % "1.0.0-RC2",
  "com.typesafe.slick" %% "slick" % "3.4.1"
)
```

#### Paso 3: Conexión a la base de datos y consulta en Scala
Ejemplo usando **Doobie**:
```scala
import cats.effect.IO
import doobie._
import doobie.implicits._

object MySQLConnection {
  val transactor: Transactor[IO] = Transactor.fromDriverManager[IO](
    "com.mysql.cj.jdbc.Driver", // Driver JDBC
    "jdbc:mysql://localhost:3306/prueba_scala", // URL de conexión
    "root", // Usuario
    "password" // Contraseña
  )

  def main(args: Array[String]): Unit = {
    val query = sql"SELECT * FROM usuarios".query[(Int, String, Int)].to[List]
    val result = query.transact(transactor).unsafeRunSync()

    result.foreach { case (id, nombre, edad) =>
      println(s"ID: $id, Nombre: $nombre, Edad: $edad")
    }
  }
}
```

Ejemplo usando **Slick**:
```scala
import slick.jdbc.MySQLProfile.api._

import scala.concurrent.Await
import scala.concurrent.duration._

object MySQLConnectionSlick {
  case class Usuario(id: Int, nombre: String, edad: Int)

  class Usuarios(tag: Tag) extends Table[Usuario](tag, "usuarios") {
    def id = column[Int]("id", O.PrimaryKey, O.AutoInc)
    def nombre = column[String]("nombre")
    def edad = column[Int]("edad")
    def * = (id, nombre, edad) <> (Usuario.tupled, Usuario.unapply)
  }

  val usuarios = TableQuery[Usuarios]
  val db = Database.forURL(
    "jdbc:mysql://localhost:3306/prueba_scala",
    driver = "com.mysql.cj.jdbc.Driver",
    user = "root",
    password = "password"
  )

  def main(args: Array[String]): Unit = {
    val query = usuarios.result
    val result = Await.result(db.run(query), 10.seconds)

    result.foreach { usuario =>
      println(s"ID: ${usuario.id}, Nombre: ${usuario.nombre}, Edad: ${usuario.edad}")
    }
  }
}
```

Ambos enfoques son funcionales y permiten realizar consultas fácilmente.
