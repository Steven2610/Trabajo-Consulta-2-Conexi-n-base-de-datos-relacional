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
    edad INT NOT NULL,
    correo VARCHAR(100) NOT NULL,
    fecha_registro VARCHAR(20) NOT NULL,
    activo BOOLEAN NOT NULL DEFAULT TRUE
);

INSERT INTO usuarios (nombre, edad, correo, fecha_registro, activo) VALUES 
('Alice', 25, 'alice@example.com', '2025-01-01', TRUE),
('Bob', 30, 'bob@example.com', '2025-01-10', FALSE),
('Charlie', 22, 'charlie@example.com', '2025-01-15', TRUE),
('Diana', 28, 'diana@example.com', '2025-01-18', TRUE),
('Evan', 35, 'evan@example.com', '2024-12-30', TRUE),
('Fiona', 27, 'fiona@example.com', '2025-01-05', FALSE),
('George', 40, 'george@example.com', '2024-12-20', TRUE),
('Hannah', 29, 'hannah@example.com', '2025-01-12', TRUE),
('Irene', 32, 'irene@example.com', '2025-01-19', FALSE),
('Jack', 26, 'jack@example.com', '2025-01-20', TRUE);

```
![image](https://github.com/user-attachments/assets/77e8cd37-fb7b-44e4-bff1-66936a84e52e)

#### Paso 2: Configurar el entorno en Scala
Agregue las dependencias necesarias en su archivo `build.sbt`:
```scala
import scala.collection.Seq

ThisBuild / version := "0.1.0-SNAPSHOT"

ThisBuild / scalaVersion := "2.13.12"
//2.13.12
lazy val root = (project in file("."))
  .settings(
    name := "untitled",
    libraryDependencies ++=Seq(

      "mysql" % "mysql-connector-java" % "8.0.33",
      "org.tpolecat" %% "doobie-core" % "1.0.0-RC1",
      "com.typesafe.slick" %% "slick" % "3.4.1"
    )
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
![image](https://github.com/user-attachments/assets/132855ad-82e9-4219-a17e-69ea5969ca78)

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

bibliografia
Doobie. (n.d.). Doobie Documentation. Recuperado de https://tpolecat.github.io/doobie/

Slick. (n.d.). Slick Documentation. Recuperado de https://scala-slick.org/doc/

MySQL. (n.d.). MySQL Documentation. Recuperado de https://dev.mysql.com/doc/

Cats Effect. (n.d.). Cats Effect Documentation. Recuperado de https://typelevel.org/cats-effect/
