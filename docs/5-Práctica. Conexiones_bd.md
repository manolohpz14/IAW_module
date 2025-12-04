## Práctica Guiada 5. Conexiones a BD

### 1-Breve introducción a la programación orientada a objetos en PHP

La Programación Orientada a Objetos (POO) es un paradigma de programación , es decir, una forma de organizar y estructurar el código, que se basa en el uso de objetos. Un objeto es una entidad que agrupa datos (propiedades o atributos) y comportamientos (métodos o funciones) que pertenecen a esa entidad. La idea principal es modelar partes del mundo real usando estructuras de código más cercanas a cómo pensamos las cosas.

Por ejemplo, imaginemos que queremos modelas a una persona mediante el uso de un objeto:

- Una persona tiene nombre, edad, DNI : son propiedades.
- Una persona camina(), habla(), saluda() : son métodos.

Cuando usamos la POO combinamos ambas cosas dentro de una estructura llamada clase. A continuación vemos las definciones más básicas de la POO que veremos a lo largo de esta práctica.

**- Clase**
 Una clase es una <u>plantilla</u> que define cómo serán los objetos creados a partir de ella. Ejemplo: La clase Coche describe qué datos tiene un coche (propiedades) como la marca, el modelo o el año, y qué puede hacer (métodos) como arrancar y frenar.

**- Objeto**
 Un objeto es una instancia real creada a partir de una clase. Ejemplo: Un coche concreto, como un Seat Ibiza de 2018, es un objeto creado a partir de la clase Coche, es decir, una instancia.

![image-20251121105609040](./img/coche.png)

**- Propiedades**
 Son variables que representan la información que guarda un objeto. Ejemplo: Un coche tiene propiedades como color='rojo' o kilómetros=120000 o velocidad=40km/h.

**- Métodos**
 Son funciones que representan acciones que un objeto puede realizar.  Ejemplo: Un coche puede ejecutar acciones (métodos) como arrancar() o acelerar(), que afectarán a propiedades como la velocidad o los kilómetros.

**- Propiedades estáticas**
 Son propiedades que pertenecen a la clase en general, no a cada objeto. Ejemplo (programado en lo siguiente): La clase Persona podría tener un contador de cuántas personas se han creado en total, independientemente de quiénes sean esas personas.

---

#### 1.1-Primer ejemplo de POO en PHP.

Nombre del script: primera_clase.php

En este script, vemos el concepto de: Clase, Objeto (Instancia de la clase), Propiedades de un objeto y propiedades estáticas de una clase. 

- La pantilla serán las personas. Cada persona tendrá dos propiedades, nombre y edad.
- Los objetos serán las personas que se crean a partir de la plantilla clase. Al crear una persona deberemos asignar un valor a las propiedades nombre y edad
- Además, la plantilla (la clase) tendrá una propiedad común a todas las personas, que será el número de personas creadas.

Os recomiendo ejecutar el siguiente código en el intérprete de php. PHP necesita saber qué propiedades tiene la clase antes de crear el objeto. El constructor no crea propiedades, solo les da valores.

```php
<?php
class Persona
{
    // propiedades de la clase, que se definiran cuando la instanciemos
    public string $nombre;
    public int $edad;

    // contador de instancias de objeto, cada vez que creeemos a una persona subiremos en 1
    public static int $contador = 0;

    public function __construct(string $nombre, int $edad)
    {
        $this->nombre = $nombre;
        $this->edad = $edad;
        self::$contador++;
    }
}
```

##### This vs self

$this se usa cuando quieres trabajar con un objeto específico. Representa a la instancia actual de la clase, es decir, al objeto que se creó con new. En cambio, self se usa cuando quieres trabajar con algo que pertenece a la clase en general, no a cada objeto. Esto es justo lo que pasa cuando le damos valor en el constructor de la clase a la propiedad estática contador, que necesitamos sí o sí acceder poniendo self.

Con lo anterior, tenemos la plantilla de los objetos que vamos a crear, la idea ahora es ver como creamos los objetos de dicha clase, lo vemos aqui:

##### **Instanciando objetos**

```php
<?php
$persona1 = new Persona("Carlos", 30);
$persona2 = new Persona("Ana", 25);
$persona3 = new Persona("Luis", 40);

echo $persona1->nombre . ", " . $persona1->edad . "\n";
echo $persona2->nombre . ", " . $persona2->edad . "\n";
echo $persona3->nombre . ", " . $persona3->edad . "\n";

echo Persona::$contador  . "\n"; // resultado es 3

$persona1->nombre="Paco";
echo $persona1->nombre . ", " . $persona1->edad . "\n";

Persona::$contador=28;
echo Persona::$contador . "\n";
```

Como podemos observar el código anterior tiene ciertos problemas. Se puede instanciar un objeto y posteriormente cambiar las propiedades del objeto sin piedad (como el nombre y la edad), incluso también podría cambiar la propiedad estática de la clase,  propiedad que como hemos comentado antes,  es compartida entre todos los objetos instanciados.

---

#### 1.2-Segundo ejemplo de POO en PHP. Uso de propiedades privadas.

En la segunda versión del código (ver más abajo), haremos que nadie pueda cambiar el valor de la propiedad estática (el número de personas), pasándola a privada. Además, al ser privada, ya nadie podrá acceder a ella haciendo:

```php
Persona::$contador
```

y por tanto, la única forma que tendrá un usuario de ver el valor de esta propiedad estática será definiendo una función dentro de la clase que nos otorgue su valor, que se denominará `getter`, ya que nos da el valor de una propiedad (estática en este caso):

```php
<?php
class Persona
{
    public string $nombre;
    public int $edad;
    private static int $contador = 0;

    public function __construct(string $nombre, int $edad)
    {
        $this->nombre = $nombre;
        $this->edad = $edad;
        self::$contador++;
    }

    public static function getContador(): int
    {
        return self::$contador;
    }
}

$persona1 = new Persona("Carlos", 30);
$persona2 = new Persona("Ana", 25);
$persona3 = new Persona("Luis", 40);

echo $persona1->nombre . ", " . $persona1->edad . "\n";
echo $persona2->nombre . ", " . $persona2->edad . "\n";
echo $persona3->nombre . ", " . $persona3->edad . "\n";

try {echo Persona::$contador . "\n";} //ya no podemos acceder ni editar así, hay que utilizar el getter
catch (Throwable $e) {echo $e->getMessage() . "\n";}
echo Persona::getContador() . "\n";

//Por desgracia, podemos cambiar el nombre a Paco, cosa que no deberíamos poder hacer.
$persona1->nombre="Paco";
echo $persona1->nombre . ", " . $persona1->edad . "\n"; // Carlos, 30

```

Como vemos al final del script siguen habiendo inconsistencias, como que se pueda cambiar el nombre de la persona (o incluso la edad). Esto se debe a que son propiedades públicas.

---

#### 1.3-Tercer ejemplo de POO en PHP. Getters y Setters. Encapsulamiento

En este último ejemplo, rizamos un poco más el rizo, lo que vemos ahora es encapsulación. Es decir, vamos a arreglar lo anterior. Para ello, no expondremos directamente al público las propiedades "edad" y "nombre" si no que estas se podrán editar u obtener únicamente a través de getters y setters.

**- Getters y Setters**
 Son métodos que permiten leer o modificar propiedades privadas de forma controlada. Ejemplo: Para saber la edad de una persona usamos getEdad(), y para cambiarla usamos setEdad(), que comprueba que sea un número válido.

Cuando un getter o un setter es privado, no puede ser accedido desde una instancia, si no que solo se puede acceder desde la propia clase

```php
<?php
declare(strict_types=1);
class Persona
{
    // ahora todas las propiedades son privadas
    private string $nombre;
    private int $edad;
    private static int $contador = 0;

    public function __construct(string $nombre, int $edad)
    {
        $this->setNombre($nombre);
        $this->setEdad($edad);
        self::$contador++;
    }
    public function getNombre(): string {return $this->nombre;}
    private function setNombre(string $nombre): void
    {
        if (empty($nombre)) {
            throw new Exception("El nombre no puede estar vacío.");
        }
        $this->nombre = $nombre;
    }
    public function getEdad(): int {return $this->edad;}
    private function setEdad(int $edad): void
    {
        if ($edad < 0) {
            throw new Exception("La edad no puede ser negativa.");
        }
        $this->edad = $edad;
    }
    public static function getContador(): int {return self::$contador;}
}

$persona1 = new Persona("Carlos", 30);
$persona2 = new Persona("Ana", 25);
$persona3 = new Persona("Luis", 40);
echo $persona1->getNombre() . ", " . $persona1->getEdad() . "\n";
```

Ojo, si ponemos  declare (strict_types=1); como en lo anterior, es conveniente tener un control de errores correcto.

```php
<?php
try {
    $persona1 = new Persona("Carlos", 30);
    $persona2 = new Persona("Ana", 25);
    $persona3 = new Persona("Luis", -40); 
}
catch (Exception $e) {
    echo "Error al crear persona: " . $e->getMessage();
}

echo $persona1->getNombre() . ", " . $persona1->getEdad();
```

recordad esto:

```makefile
Throwable
├── Exception
└── Error
    ├── TypeError
    ├── ParseError
    ├── ValueError
    └── ...
```

Por último, si hubiéramos querido, a la clase anterior lo podríamos haber añadido algún método público para que, por ejemplo, una persona pueda cumplir años:

```php
public function addAge(): int {
    $this->edad+=1;
}
```

De esta forma, cualquier instancia puede cumplir años sin problema.

---

#### Práctica PT1. Clase básica con propiedades públicas 

-Crea un archivo llamado **coches_1.php**. En este archivo define una clase Coche con:

- Propiedades públicas:
  
  - marca (string)
  - modelo (string)
  - velocidad (int)
  
- Una propiedad estática pública llamada contador, que registre cuántos coches se han creado.

- Un constructor que reciba los valores de marca, modelo y velocidad inicial y los inicialice


**Se pide:**

- Crear tres objetos tipo Coche, mostrar sus propiedades sobre las instancias, mostrar el contador de coches sobre la case, modificar “a mano” alguna propiedad pública (por ejemplo cambiar la marca o la velocidad) y modificar “a mano” la propiedad estática.

---

#### Práctica PT2. Clase básica con propiedades públicas 

-Crea un archivo **coches_2.php**. En este archivo define una clase Coche con:

- Todas las propiedades serán privadas:
  - marca (string)
  - modelo (string)
  - velocidad(int)
  - pintura(string)
- La propiedad estática contador será privada y tendrá un getter (getContador) para ver su valor.
- En el constructor, asigna marca, modelo y pintura usando setters (definidos más abajo) y no directamente en el constructor.
- La velocidad inicial debe ser siempre 0, independientemente de lo que se pase y se pondrá así en el constructor.
- Crea getters para ver los valores de las tres propiedades mencionadas

-Reglas del setter de marca y modelo (setModelo, setMarca):

- No pueden estar vacíos y deben ser cadenas de texto.
- Además, un coche no puede cambair de marca y modelo luego deben ser setter privados y no se pueden acceder desde fuera.

-Reglas del setter de pintura (setPintura):

- Un coche podrá cambiar de pintura luego puede ser público (solo aceptara los colores rojo, verde, azul o negro.)

-Crear el método velocidad (addVelocidad) que suma o resta velocidad a un coche:

- La velocidad resultante no puede ser negativa y no puede superar los 250 km/h

**Se pide:**

- Crear tres objetos tipo Coche, mostrar sus propiedades con los getters, mostrar el contador de coches con su getter.

  Intenta modificar “a mano” alguna propiedad pública (por ejemplo cambiar la marca o la velocidad) ¿Qué pasa?

  Usar el método addVelocidad para añadir 10km/h de velocidad y cambiar la pintura al coche, que solo aceptara los colores rojo, verde, azul o negro.

---

### 2-Instalación de mySQL, creación de usuarios y privilegios

En este apartado vamos a ver cómo podemos conectarnos a bases de datos desde nuestra aplicación.

En primer lugas, instalamos mysqlserver

```bash
sudo apt update
sudo apt install mysql-server
```

Una vez instalado, abrelo como root:

```bash
sudo mysql -u root
```

Crea una BD

```bash
CREATE DATABASE testdb;
```

y úsala:

```bash
USE testdb;
```

Crea la tabla usuarios

```bash
CREATE TABLE usuarios (
    id INT AUTO_INCREMENT PRIMARY KEY,
    nombre VARCHAR(100) NOT NULL,
    email VARCHAR(150) UNIQUE NOT NULL,
    password VARCHAR(255) NOT NULL,
    creado_en TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

Después, vamos a crear un usuario con privilegios mínimos, ya que cuando el cliente se conecte a la BD no lo hará desde root. Además, este usuario sólo se podrá conectar desde localhost y nunca de forma remota, es decir, sólo s epodrá conectar desde el servidor, al servidor, lo que significa que sólo lo hará desde los scripts de php (en nuestro caso). Esto dependerá en gran medida de la configuración de nuestro servidor, en este caso, al ser nuestra aplicación monolítica, nos vale.

```bash
CREATE USER 'www-data'@'localhost' IDENTIFIED BY '123456';

```

Le damos al usuario únicamente privilegios sobre la tabla que queramos, en este caso, la tabla de usuarios

```bash
GRANT SELECT, INSERT, UPDATE, DELETE ON testdb.usuarios TO 'www-data'@'localhost';
```

Para aplciar los privilegios anteriores:

```bash
FLUSH PRIVILEGES;
```

---

### 3-Conexión a la BD en un script. Login de un usuario en la BBDD (Login.php)

Vamos a aprovechar el formulario de la pŕactica anterior (este):

![image-20251121105609040](./img/image-20251121105609040.png)

y vamos a editarlo un poco. Para ello, vamos a dejar solo tres formularios (dos POST's y un GET quitamos el resto) que mandarán peticiones a scripts que hagan cosas interesantes (En el punto 6 tenéis el código html de estos formularios).

En primer lugar, el primer POST se conecta al primer script, que llamaremos (Login.php) que nos devuelve información del usuario ya registrado en la base de de datos (en el siguiente punto veremos como registar a un usuario. Por facilidad prefiero que veais primero el login), la primera parte del script contendrá esto:

```php
<?php
$host = getenv("DB_HOST");
$db   = getenv("DB_NAME");
$user = getenv("DB_USER");
$pass = getenv("DB_PASS");

try{
    $pdo = new PDO($dsn, $user, $pass, [
            PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
            PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC, asociativo
            PDO::ATTR_EMULATE_PREPARES => false 
        ]);
} catch (PDOException $e) {
    die("Error de conexión: " . $e->getMessage());
}

// Recibir datos por un formulario (url-enconded)
$email = $_POST['email'] ?? null;
$pass_input = $_POST['password'] ?? null;

if (!$email || !$pass_input) {
    die("Faltan datos");
}
```

**doc oficial usada**: https://www.php.net/manual/en/pdo.connections.php

Según la documentación oficial, el constructor de PDO recibe:

1. DSN: cadena de conexión (driver + host + dbname + charset, etc.)
2. Usuario
3. Contraseña
4. Opciones (un array asociativo de atributos)

Hasta aquí, todo entendible. Para la conexión a la BD no queremos exponer que en el script se vea el valor de user ni el valor de pass, es por eso que se recogen desde variables de entorno (un fichero donde estas variables están definidas de forma que nadie pordrá verlas)

En concreto, estas variables las ponéis en:
/etc/apache2/envvars

y escribís:

![image-20251121180114532](./img/image-20251121180114532.png)

Por tanto, los tres primeros puntos de la conexión, son evidentes con la captura anterior, saltemos al 4º, en específico, nos vamos al array asociativo:

---

#### 3.1 Opciones del array asociativo al conectar a PDO

**doc oficial usada**: https://www.php.net/manual/en/pdo.constants.php

##### 3.1.1. PDO::ATTR_ERRMODE

En la documentación oficial se dice:*"PDO::ATTR_ERRMODE: Sets the error reporting mode."* Los modos posibles más comunes son los siguiente

- `PDO::ERRMODE_SILENT` (modo silencioso — por defecto)
- `PDO::ERRMODE_WARNING`
- `PDO::ERRMODE_EXCEPTION`

La que hemos escogido le dice a PDO que arroje excepciones (`PDOException`) cuando ocurra un error, en lugar de:

- guardar el error internamente (modo silencioso), o
- emitir warnings.

Esto hace más fácil manejar errores y deputar

##### 3.1.2. PDO::ATTR_DEFAULT_FETCH_MODE 

En la documentación oficial se dice: *"PDO::ATTR_DEFAULT_FETCH_MODE: Set default fetch mode for fetch methods."*

Modos posibles:

- `PDO::FETCH_ASSOC` (solo arrays asociativos)
- `PDO::FETCH_NUM`
- `PDO::FETCH_BOTH` (por defecto)
- `PDO::FETCH_OBJ`, etc.

PDO te devuelve por defecto dos copias del mismo dato al hacer:

```php
<?php
$stmt->fetch();
```

Es decir, vemos algo como

```
[
    0        => "Juan",
    "nombre" => "Juan"
]
```

La opcion que hemos escogido establece que cada vez que hagas: el resultado tendrá solo claves asociativas (`$fila['nombre']`), no índice numérico.

##### 3.1.3. PDO::ATTR_EMULATE_PREPARES

En la documentación oficial se dice: *"PDO::ATTR_EMULATE_PREPARES: Enable or disable emulated prepared statements."*

Lo que en realidad estás pidiendo es que el trabajo de preparar la consulta lo haga MySQL directamente, y no PHP. Por ejemplo, si haces una consulta como:

```php
<?php
$stmt = $pdo->prepare("SELECT * FROM usuarios WHERE email = ?");
$stmt->execute([$email]);
```

Si la emulación está activada, PDO toma ese `?`, lo reemplaza por el valor del email, arma la consulta completa como una cadena de texto y se la manda ya lista a MySQL. Es decir, PHP hace el trabajo. Pero si usas prepared statements nativos, PDO envía la consulta con el `?` tal cual, y luego manda el valor del email por separado. En ese caso, es MySQL quien prepara la consulta y la ejecuta de forma segura,

---

#### 3.2 Creación de consultas

Seguimos rellenando el script con la parte de la consulta:

```php
$sql = "SELECT id, nombre, email, password FROM usuarios WHERE email = ?";
$stmt = $pdo->prepare($sql);
$stmt->execute([$email]);
$userData = $stmt->fetch();
```

¿Qué singifica todo el bloque anterior? Vamos por partes

##### 3.2.1 Consulta

```php
$sql = "SELECT id, nombre, email, password FROM usuarios WHERE email = ?";
```

El ? es un hueco en donde luego vamos a poner el valor del email del usuario. No lo ponemos directamente en el string. Lo vamos a mandar aparte, de forma segura.

##### 3.2.2 Preparar la consulta

```php
$stmt = $pdo->prepare($sql);
```

- `$conn`: es la conexión a MySQL (`new PDO(...)`).
- `prepare()`: método de la clase PDO que devuelve un objeto del tipo PDOStatement, que es la clase de PDO para manejar sentencias preparadas.
- `execute([$email])`: envía el valor del email

##### 3.2.3 Dando valores a la consulta

**doc oficial usada:** https://www.php.net/manual/en/pdo.prepared-statements.php

Por último, falta dar valor a `?`: esto se hace así:

```bash
$stmt->execute([$email])
```

Otra forma de hacerlo sería así;

```php
<?php
$stmt = $pdo->prepare("SELECT * FROM usuarios WHERE email = :email");

$stmt->execute([
    ':email' => $email
]);

```

##### 3.2.4 Obteniendo los valores de la consulta

Por último, veamos como se obtienen los resultados:

```php
$userData = $stmt->fetch();
```

`fetch()` devuelve UN array asociativo POR LLAMADA, es decir, si el resultado tuviera más de una fila, tendríamos que hacer algo así:

```php
while ($usuario = $stmt->fetch()) {
 echo $usuario["nombre"] . "<br>";
}
```

Si no hay más filas, devuelve false. No importa si la tabla tiene 10 filas o 10 millones, siempre estás usando muy poca memoria, porque solo tienes una fila en memoria a la vez. De esta forma, en casa iteración obtengo una fila distinta, cosa que no es necesaria en este caso. Si quisieras cargar todas las filas en un array gigante usarías: -> fetchAll

Por ejemplificar lo que se obtiene en cada llamada podemos ver el siguiente ejemplo, donde en la tabla de usuarios sólo hay: 

| id   | nombre | email                                       |
| ---- | ------ | ------------------------------------------- |
| 5    | Juan   | [Juan@mail.com](mailto:test@mail.com)<br /> |
| 7    | Pepa   | [Pepa@mail.com](mailto:test@mail.com)<br /> |

Al usar $result->fetch(), sobre la tabla anterior, cada llamada se convierte esto en un array asociativo que podremos manejar sin problemas en php (es decir, tendríamos algo tal que así:)

```text
[
    "id" => 5,
    "nombre" => "Juan",
    "email" => "test@mail.com"
]
```


El código completo de (Login.php) queda así:

```php
<?php
header("Content-Type: application/json");
$host = getenv("DB_HOST");
$db   = getenv("DB_NAME");
$user = getenv("DB_USER");
$pass = getenv("DB_PASS");

try {
    $dsn = "mysql:host=$host;dbname=$db;charset=utf8mb4";
    $pdo = new PDO($dsn, $user, $pass, [
        PDO::ATTR_ERRMODE            => PDO::ERRMODE_EXCEPTION,
        PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC,
        PDO::ATTR_EMULATE_PREPARES   => false
    ]);

    $email = $_POST['email'] ?? null;
    $pass_input = $_POST['password'] ?? null;

    if (!$email || !$pass_input) {
        throw new Exception("Faltan datos");
    }

    $stmt = $pdo->prepare("SELECT id, nombre, email, password FROM usuarios WHERE email = ?");
    $stmt->execute([$email]);
    $userData = $stmt->fetch();

    if (!$userData) {
        throw new Exception("Usuario no encontrado");
    }

    if (!password_verify($pass_input, $userData['password'])) {
        throw new Exception("Contraseña incorrecta");
    }

    unset($userData['password']);

    echo json_encode([
        "status"  => "success",
        "message" => "Login exitoso",
        "data"    => $userData
    ]);

} catch (PDOException $e) {

    echo json_encode([
        "status"  => "error",
        "message" => "Error de base de datos: " . $e->getMessage()
    ]);

} catch (Throwable $e) {

    echo json_encode([
        "status"  => "error",
        "message" => $e->getMessage()
    ]);
}
```

---

### 4-Conexión a la BBDD.POST. Registrar a un usuario para probar el loggin anterior. (Register.php)

Ya sabemos un poco sobre conexión a base de datos según lo visto en el punto anterior. Ahora vamos al revés, os doy el script entero de (Register.php) y lo que haremos será desmenuzar los puntos que no comprendamos.

```php
<?php
header("Content-Type: application/json");
$host = getenv("DB_HOST");
$db   = getenv("DB_NAME");
$user = getenv("DB_USER");
$pass = getenv("DB_PASS");

try {
    $dsn = "mysql:host=$host;dbname=$db;charset=utf8mb4";
    $pdo = new PDO($dsn, $user, $pass, [
        PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
        PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC,
        PDO::ATTR_EMULATE_PREPARES => false
    ]);

    $nombre = $_POST['nombre'] ?? null;
    $email = $_POST['email'] ?? null;
    $pass_input = $_POST['password'] ?? null;

    if (!$nombre || !$email || !$pass_input) {
        throw new Exception("Faltan datos");
    }

    $hashedPassword = password_hash($pass_input, PASSWORD_DEFAULT);

    $sql = "INSERT INTO usuarios (nombre, email, password) 
            VALUES (:nombre, :email, :password)";

    $stmt = $pdo->prepare($sql);
    $stmt->execute([
        ':nombre'   => $nombre,
        ':email'    => $email,
        ':password' => $hashedPassword
    ]);

    echo json_encode([
        "status" => "success",
        "message" => "Usuario registrado correctamente",
        "user_id" => $pdo->lastInsertId()
    ]);

} catch (PDOException $e) {

    echo json_encode([
        "status" => "error",
        "message" => "Error de base de datos",
        "mysql_error" => $e->getMessage()
    ]);

} catch (Throwable $e) {

    echo json_encode([
        "status" => "error",
        "message" => $e->getMessage() //Ojo a la pedazo de mala práctica
    ]);
}
```

Vamos la cosas que todavía pueden resultar confusas porque no se han visto antes;

###### ¿Qué es PASSWORD_DEFAULT?

Es una constante predefinida por PHP cuyo valor es un número entero que identifica el algoritmo de hasehado de contraseña. No lo ves como texto porque PHP ya lo interpreta. si haces:

```php
echo PASSWORD_DEFAULT;
```

obtienes
```php
1 #metodo de hasehado por defecto. esto no se suele cambiar ni pediré que lo hagáis
```

---

### 5-Conexión a la BBDD. GET que devuelve la info de todos los usuarios. (All_users.php)

En este caso, no hace falta explicar tanto ya que hemos visto muchas cosas, este formulario es el ejemplo vivo de un get en el que no se envía ninguna información desde el cliente (obviamente si desde el servidor). A continuación os dejo el código. 
Recordar que en un formulario html que use get, los parámetros se envían por query-param:

```php
<?php
header("Content-Type: application/json");
$host = getenv("DB_HOST");
$db   = getenv("DB_NAME");
$user = getenv("DB_USER");
$pass = getenv("DB_PASS");

try {
    $dsn = "mysql:host=$host;dbname=$db;charset=utf8mb4";
    $pdo = new PDO($dsn, $user, $pass, [
        PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
        PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC,
        PDO::ATTR_EMULATE_PREPARES => false
    ]);
} catch (PDOException $e) {
    http_response_code(500);
    die(json_encode([
        "status" => "error",
        "message" => "Error de conexión",
        "mysql_error" => $e->getMessage()
    ]));
}

if ($_SERVER['REQUEST_METHOD'] !== 'GET') {
    http_response_code(405);
    die(json_encode([
        "status" => "error",
        "message" => "Método no permitido"
    ]));
}

try {
    $stmt = $pdo->prepare("SELECT id, nombre, email FROM usuarios");
    $stmt->execute();
    $usuarios = $stmt->fetchAll();

    echo json_encode([
        "status" => "success",
        "message" => "Usuarios obtenidos correctamente",
        "data" => $usuarios
    ]);

} catch (PDOException $e) {

    echo json_encode([
        "status" => "error",
        "message" => "Error al obtener usuarios",
        "mysql_error" => $e->getMessage()  //Ojo a la pedazo de mala práctica
    ]);
}
```

---

### 6- Esquema del HTML

A modo esquemático, el body html hará las peticiones así a los tres scripts anteriores será el siguiente:

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>


    <style>
        body {
            display: flex;
            justify-content: center;
            align-items: center;
        }

        .contenedor {
            display: flex;
            flex-direction: column;
        }

        .carta {
            border: 1px solid rgb(1, 1, 1);
            text-align: center;
            margin-top: 1rem;
            border-radius: 1rem;
            padding: 1rem;
            background-color: rgb(222, 255, 252);
            max-width: 320px;
        }

        form {
            display: flex;
            flex-direction: column;
            margin-top: 0.5rem;

        }
        input {
            max-width:2,5rem;
            margin: 0px auto;
            margin-top:0.5rem;
        }

        button {
            max-width:2,5rem;
            margin: 0px auto;
            margin-top:0.5rem;
        }
    </style>



</head>


<body>
    <div class="contenedor">

        <div class="carta">
            <form action="Login.php" method="POST">
                <h2>Esto sería un primer POST de login</h2>
                <input type="text" name="email" placeholder="email">
                <input type="password" name="password" placeholder="Contraseña">
                <button type="submit">Enviar</button>
            </form>
        </div>

        <div class="carta">
            <form action="Register.php" method="POST" enctype="multipart/form-data">
                <h2>Esto sería un POST de register</h2>
                <input type="text" name="nombre" placeholder="Nombre">
                <input type="text" name="email" placeholder="Email">
                <input type="password" name="password" placeholder="Contraseña">
                <input type="file" name="foto">
                <button type="submit">Enviar </button>
            </form>
        </div>


        <div class="carta">
            <form action="All_users.php" method="GET">
                <h2>Esto sería un GET de todos los usuarios</h2>
                <button type="submit">Enviar </button>
            </form>
        </div>


</body>

</html>
```

En las prácticas siguientes veremos como utilzar el PUT y el DELETE (desde JS al igual que en la práctica anterior) de forma práctica y real, borrando registros de la BD y editando cuestiones de la BD.

---

### **7- Práctica**

1-Realiza la práctica guiada anterior e intenta que por cada usuario que se registre suba una foto del usuario dentro de la carpeta uploads.

![image-20251204132607438](./img/image-20251204132607438.png)

2-Crea dos formularios HTML (en peticiones_productos.html) **<u>que NO usarán javascript salvo que sea estrictamente necesario.</u>** Ambos deberán hacer una petición GET y un POST respectivamente. Usarán boton de tipo submit y la información no se recogerá dinámicamente si no que se hará directamente desde el html.

![image-20251204132623154](./img/image-20251204132623154.png)

- El POST del formulario html apuntará al script: productos_form.php. Lo que hará este script sera conectarse a la BD para registar un producto junto con su ID en una base de datos llamada productos y una breve descripcion. Deberás crear la tabla productos para ello.

- El GET recogerá la información de un producto en específico (quey -param), en este caso, no será un json si no que el script "info_productos.php" deberá devolver un echo con la información entera del producto.