# Taller-Grupal-2-Persistencia-de-datos-en-archivos


## Pasos del Taller

### 1. Generar el archivo CSV

Crea un archivo `estudiantes.csv` con el siguiente contenido:

```csv
nombre,edad,calificacion,genero
Andrés,10,20,M
Ana,11,19,F
Luis,9,18,M
Cecilia,9,18,F
Katy,11,15,F
Jorge,8,17,M
Rosario,11,18,F
Nieves,10,20,F
Pablo,9,19,M
Daniel,10,20,M
```

### 2. Genere la tabla en MYSQL para esta información.

```sql
CREATE TABLE estudiantes (
    nombre VARCHAR(50),
    edad INT,
    calificacion INT,
    genero varchar(1)
);
```

### 3. Elabore un programa que inyecte los datos del archivo CSV a la base de datos. 

```scala
object EstudiantesDAO {
  def insert(estudiante: Estudiante): ConnectionIO[Int] = {
    sql"""
       INSERT INTO estudiantes (nombre, edad, calificacion, genero)
       VALUES (
         ${estudiante.nombre},
         ${estudiante.edad},
         ${estudiante.calificacion},
         ${estudiante.genero.toString}
       )
     """.update.run
  }
  def insertAll(estudiantes: List[Estudiante]): IO[List[Int]] = {
    Database.transactor.use { xa =>
      estudiantes.traverse(t => insert(t).transact(xa))
    }
  }

```


### 4. En el mismo programa agregue la funcionalidad para obtener de la base de datos todos los registros de Estudiantes. 

Datos en el DAO:

```sql
//Obtener todos los estudiantes de la base de datos
  def getAll: IO[List[Estudiante]] = {
    val query = sql"""
        SELECT nombre, edad, calificacion, genero
        FROM estudiantes
      """.query[Estudiante].to[List] // Ejecuta la consulta y mapea a una lista de Estudiantes
    Database.transactor.use { xa =>
      query.transact(xa)
    }
  }
```

Inovacion en el main:

```sql
  // Secuencia de operaciones IO usando for-comprehension
  def run: IO[Unit] = for {
    result <- EstudiantesDAO.insertAll(estudiantes)  // Inserta datos y extrae resultado con <-
    _ <- IO.println(s"Registros insertados: ${result.size}")  // Imprime cantidad

    // Obtener todos los registros y los imprime
    allStudents <- EstudiantesDAO.getAll
    _ <- allStudents.traverse(s => IO.println(s))
  } yield ()  // Completa la operación
```
