## Práctica Guiada 5. Conexiones a BD

### 1-Breve introducción a la programación orientada a objetos en PHP

La Programación Orientada a Objetos (POO) es un paradigma de programación , es decir, una forma de organizar y estructurar el código, que se basa en el uso de objetos. Un objeto es una entidad que agrupa datos (propiedades o atributos) y comportamientos (métodos o funciones) que pertenecen a esa entidad. La idea principal es modelar partes del mundo real usando estructuras de código más cercanas a cómo pensamos las cosas.

Por ejemplo:

- Una persona tiene nombre, edad, DNI : son propiedades.
- Una persona camina(), habla(), saluda() : son métodos.

En POO combinamos ambas cosas dentro de una estructura llamada clase. A continuación vemos las definciones más básicar de la POO que veramos a lo largo de esta práctica.

**- Clase**
 Una clase es una <u>plantilla</u> que define cómo serán los objetos creados a partir de ella. Ejemplo: La clase Coche describe qué datos tiene un coche (propiedades) como la marca, el modelo o el año, y qué puede hacer (métodos) como arrancar y frenar.

**- Objeto**
 Un objeto es una instancia real creada a partir de una clase. Ejemplo: Un coche concreto, como un Seat Ibiza de 2018, es un objeto creado a partir de la clase Coche, es decir, una instancia.

<img src="./img/image-20251127105113973.png" alt="image-20251127105113973" style="zoom:50%;" />

**- Propiedades**
 Son variables que representan la información que guarda un objeto. Ejemplo: Un coche tiene propiedades como color='rojo' o kilómetros=120000 o velocidad=40km/h.

**- Métodos**
 Son funciones que representan acciones que un objeto puede realizar.  Ejemplo: Un coche puede ejecutar acciones (métodos) como arrancar() o acelerar(), que afectarán a propiedades como la velocidad o los kilómetros.

**- Propiedades estáticas**
 Son propiedades que pertenecen a la clase en general, no a cada objeto. Ejemplo (programado en lo siguiente): La clase Persona podría tener un contador de cuántas personas se han creado en total, independientemente de quiénes sean esas personas.



#### 1.1-Primer ejemplo de POO en PHP.

Nombre del script: primera_clase.php

En este script, vemos el concepto de: Clase, Objeto (Instancia de la clase), Propiedades de un objeto y propiedades estáticas de una clase. 

- La pantilla serán las personas. Cada persona tendrá dos propiedades, nombre y edad.
- Los objetos serán las personas que se crean a partir de la plantilla clase. Al crear una persona deberemos asignar un valor a las propiedades nombre y edad
- Además, la plantilla (la clase) tendrá una propiedad común a todas las personas, que será el número de personas creadas.

Os recomiendo ejecutar el siguiente código en el intérprete de php.

```php
<?php
class Persona
{
    // propiedades de la clase, que se definiran cuando la instanciemos
    public string $nombre;
    public int $edad;

    // Contador de instancias de objeto, cada vez que creeemos a una persona subiremos en 1
    public static int $contador = 0;

    public function __construct(string $nombre, int $edad)
    {
        $this->nombre = $nombre;
        $this->edad = $edad;
        self::$contador++;
    }
}

$persona1 = new Persona("Carlos", 30);
$persona2 = new Persona("Ana", 25);
$persona3 = new Persona("Luis", 40);

echo $persona1->nombre . ", " . $persona1->edad . "\n"; // Carlos, 30
echo $persona2->nombre . ", " . $persona2->edad . "\n"; // Ana, 25
echo $persona3->nombre . ", " . $persona3->edad . "\n"; // Luis, 40

echo Persona::$contador  . "\n"; // 3 → se crearon tres objetos

//Por desgracia, podemos cambiar el nombre a Paco, cosa que no deberíamos poder hacer.
$persona1->nombre="Paco";
echo $persona1->nombre . ", " . $persona1->edad . "\n"; // Carlos, 30

//Por desgracia, podemos cambiar el numero de personas, cosa que no deberíamos poder hacer porque tampoco tiene ningun sentido

Persona::$contador=28;
echo Persona::$contador .  "\n"; // 28 -> No tiene sentido porque tenemos sólo 3 personas`
```

Como podemos observar el código anterior tiene ciertos problemas

Se puede instanciar un objeto y posteriormente cambiar las propiedades del objeto sin piedad (como el nombre y la edad), incluso también podría cambiar la propiedad estática de la clase,  propiedad que como hemos comentado antes,  es compartida entre todos los objetos instanciados.

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
    // Ponemos la propiedad como estática.
    private static int $contador = 0;

    public function __construct(string $nombre, int $edad)
    {
        $this->nombre = $nombre;
        $this->edad = $edad;
        self::$contador++;
    }

    // Getter de contador
    public static function getContador(): int
    {
        return self::$contador;
    }
}


$persona1 = new Persona("Carlos", 30);
$persona2 = new Persona("Ana", 25);
$persona3 = new Persona("Luis", 40);

echo $persona1->nombre . ", " . $persona1->edad . "\n"; //La salida será: Carlos, 30
echo $persona2->nombre . ", " . $persona2->edad . "\n"; //La salida será: Ana, 25
echo $persona3->nombre . ", " . $persona3->edad . "\n"; //La salida será: Luis, 40

try {echo Persona::$contador . "\n";} //ya no podemos acceder ni editar así, hay que utilizar el getter
catch (Throwable $e) {echo $e->getMessage() . "\n";}
echo Persona::getContador() . "\n"; //Forma correcta de usar el getter

//Por desgracia, podemos cambiar el nombre a Paco, cosa que no deberíamos poder hacer.
$persona1->nombre="Paco";
echo $persona1->nombre . ", " . $persona1->edad . "\n"; // Carlos, 30

```

Como vemos al final del script siguen habiendo inconsistencias, como que se pueda cambiar el nombre de la persona (o incluso la edad). Esto se debe a que son propiedades públicas.

#### 1.3-Tercer ejemplo de POO en PHP. Getters y Setters. Encapsulamiento

En este último ejemplo, rizamos un poco más el rizo, lo que vemos ahora es encapsulación. Es decir, vamos a arreglar lo anterior. Para ello, no expondremos directamente al público las propiedades "edad" y "nombre" si no que estas se podrán editar u obtener únicamente a través de getters y setters.

**- Getters y Setters**
 Son métodos que permiten leer o modificar propiedades privadas de forma controlada. Ejemplo: Para saber la edad de una persona usamos getEdad(), y para cambiarla usamos setEdad(), que comprueba que sea un número válido.

```php
<?php
class Persona
{
    // Todas las propiedades son privadas
    private string $nombre;
    private int $edad;

    // Contador de instancias (estático y privado)
    private static int $contador = 0;

    public function __construct(string $nombre, int $edad)
    {
        $this->setNombre($nombre);
        $this->setEdad($edad);
        self::$contador++;
    }

    // Getter y setter de nombre
    public function getNombre(): string
    {
        return $this->nombre;
    }

    public function setNombre(string $nombre): void
    {
        if (empty($nombre)) {
            throw new Exception("El nombre no puede estar vacío.");
        }
        $this->nombre = $nombre;
    }

    // Getter y setter de edad
    public function getEdad(): int
    {
        return $this->edad;
    }

    public function setEdad(int $edad): void
    {
        if ($edad < 0) {
            throw new Exception("La edad no puede ser negativa.");
        }
        $this->edad = $edad;
    }

    // Método estático
    
    public static function getContador(): int
    {
        return self::$contador;
    }
}

$persona1 = new Persona("Carlos", 30);
$persona2 = new Persona("Ana", 25);
$persona3 = new Persona("Luis", 40);

// Mostrar datos usando saltos de línea (\n)
echo $persona1->getNombre() . ", " . $persona1->getEdad() . "\n";
```

#### Práctica PT1. Clase básica con propiedades públicas 

Crea un archivo llamado **coches_1.php**. En este archivo define una clase **Coche** con:

- Propiedades públicas:
  - **marca** (string)
  - **modelo** (string)
  - **velocidad** (int)
- Una propiedad estática pública llamada **contador**, que registre cuántos coches se han creado.
- Un constructor que reciba los valores de marca, modelo y velocidad inicial.

**Se pide:**

- Crear tres objetos tipo Coche, mostrar sus propiedades, mostrar el contador de coches, modificar “a mano” alguna propiedad pública (por ejemplo cambiar la marca o la velocidad) y modificar “a mano” la propiedad estática.

#### Práctica PT2. Clase básica con propiedades públicas 

Crea un archivo **coches_3.php**. En este archivo define una clase **Coche** con:

- Todas las propiedades serán privadas:
  - **marca (string)**
  - **modelo (string)**
  - **velocidad(int)**

- La propiedad estática **contador** será privada y tendrá un **getter** (getContador) para ver su valor.
- En el constructor, asigna marca y modelo usando **setters** (definidos más abajo) y no directamente en el constructor.
- La velocidad inicial debe ser siempre 0, independientemente de lo que se pase y se pondrá así en el constructor.
- Crea getters para ver los valores de las tres propiedades mencionadas

Reglas del setter de marca y modelo (setModelo, setMarca):

- No pueden estar vacíos y deben ser cadenas de texto.
- Además, un coche no puede cambair de marca y modelo luego deben ser setter privados y no se pueden acceder desde fuera.

Crear el metodo velocidad (addVelocidad) que suma o resta velocidad a un coche:

- La velocidad resultante no puede ser negativa y no puede superar los 250 km/h

**Se pide:**

- Crear tres objetos tipo Coche, mostrar sus propiedades con los getters, mostrar el contador de coches con su getter.

  Intenta modificar “a mano” alguna propiedad pública (por ejemplo cambiar la marca o la velocidad) ¿Qué pasa?

  Usar el méotodo addVelocidad para añadir 10km/h de velocidad.

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

### 2-Conexión a la BD en un script. Login de un usuario en la BBDD (Login.php)

Vamos a aprovechar el formulario de la pŕactica anterior (este):

![image-20251121105609040](./img/image-20251121105609040.png)

y vamos a hacer que dos formularios (dos POST's y un GET quitamos el resto) se conecten a scripts que hagan cosas interesantes. En primer lugar, el primer POST se conecta al primer script, que llamaremos (Login.php) que nos devuelve información del usuario (en el siguiente punto veremos como registar a un usuario), la primera parte del script contendrá esto:

```php
<?php
$host = getenv("DB_HOST");
$db   = getenv("DB_NAME");
$user = getenv("DB_USER");
$pass = getenv("DB_PASS");

$conn = new mysqli($host, $user, $pass, $db);

if ($conn->connect_error) {
    die("Error de conexión: " . $conn->connect_error);
}

// Recibir datos por un formulario (url-enconded)
$email = $_POST['email'] ?? null;
$pass_input = $_POST['password'] ?? null;

if (!$email || !$pass_input) {
    die("Faltan datos");
}
```

Hasta aquí, todo entendible. Para la conexión a la BD no queremos exponer que en el script se vea el valor de user ni el valor de pass, es por eso que se recogen desde variables de entorno (un fichero donde estas variables están definidas de forma que nadie pordrá verlas)

En concreto, estas variables las ponéis en:
/etc/apache2/envvars

y escribís:

![image-20251121180114532](./img/image-20251121180114532.png)

Seguimos rellenando el script con la parte de la consulta:

```php
$sql = "SELECT id, nombre, email, password FROM usuarios WHERE email = ?";
$stmt = $conn->prepare($sql);
$stmt->bind_param("s", $email);
$stmt->execute();
$result = $stmt->get_result();
```

¿Qué singifica?

```php
$sql = "SELECT id, nombre, email, password FROM usuarios WHERE email = ?";
```

El `?` es un hueco en donde luego vamos a poner el valor del email del usuario. No lo ponemos directamente en el string. Lo vamos a mandar aparte, de forma segura.

```php
$stmt = $conn->prepare($sql);
```

- `$conn`: es la conexión a MySQL (`new mysqli(...)`).
- `prepare()`: es un método que le dice a MySQL: Aquí tienes una consulta con huecos (`?`), prepárala porque luego te mandaré los valores.
- Devuelve un objeto de tipo `mysqli_stmt` (statement = sentencia preparada). Ese objeto lo guardamos en la variable `$stmt`.

Si fallara algo, $stmt  sería `false`. En código se comprueba:

```php
if (!$stmt) {
    die("Error al preparar: " . $conn->error);
}
```

Por último, falta dar valor a `?`: esto se hace así:

```bash
$stmt->bind_param("s", $email);
```

Este es el método que más confunde, así que vamos muy lento: Pega la variable `$email` al `?` que pusimos en la consulta. O sea: el `?` ahora significa “el valor de $email”. Nos falta todavía saber qué significa la "s".

 **¿Qué significa `"s"`?** : El primer parámetro "s" indica el tipo de dato del valor que vas a pasar:

- "s"  string (texto)
- "i" integer (entero / número)
- "d" double (número decimal)
- "b"  blob (datos binarios)

En este caso:  `"s"` = le estamos diciendo a MySQL: el hueco `?` va a recibir texto. En el ejemplo (el del POST) siguiente veremos que podemos pasar mas de una cosa y nó solo una al mismo tiempo. Por último, ejecutamos la consulta y obtenemos el resultado:

```php
$stmt->execute();
#No tienes que mandar otra vez la consulta ni los valores: ya están “dentro” del $stmt aplicando el método get_result.
$result = $stmt->get_result();

```

Si quisieramos mandar el resultado al cliente, se lo mnandaríamos mediante un array asociativo, usando;

> [!IMPORTANT]
>
> `fetch_assoc()` devuelve UN array asociativo POR LLAMADA, es decir, si el resultado tuviera más de una fila, tendríamos que hacer algo así:
>
> ```php
> while ($usuario = $result->fetch_assoc()) {
>     echo $usuario["nombre"] . "<br>";
> }
> ```
>
> De esta forma, en casa iteración obtengo una fila distinta, cosa que no es necesaria en este caso.

```bash
$userData = $result->fetch_assoc();
```

que hace lo siguiente, si mi tabla tiene:

| id   | nombre | email                                       |
| ---- | ------ | ------------------------------------------- |
| 5    | Juan   | [test@mail.com](mailto:test@mail.com)<br /> |

Convierte esto en un array asociativo que podremos manejar sin problemas en php;

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
$host = getenv("DB_HOST");
$db   = getenv("DB_NAME");
$user = getenv("DB_USER");
$pass = getenv("DB_PASS");

$conn = new mysqli($host, $user, $pass, $db);

if ($conn->connect_error) {
    die("Error de conexión: " . $conn->connect_error);
}

// Recibir datos
$email = $_POST['email'] ?? null;
$pass_input = $_POST['password'] ?? null;

if (!$email || !$pass_input) {
    die("Faltan datos");
}

// Consulta usando prepared statement
$sql = "SELECT id, nombre, email, password FROM usuarios WHERE email = ?";

$stmt = $conn->prepare($sql);
if (!$stmt) {
    die("Error al preparar: " . $conn->error);
}
$stmt->bind_param("s", $email);
$stmt->execute();
$result = $stmt->get_result();

if ($result->num_rows === 0) {
    die("Usuario no encontrado");
}

$userData = $result->fetch_assoc();

// Verificar contraseña hasehada, esta función es importantísima ya que mirará la contraseña hasheada de la bd
if (!password_verify($pass_input, $userData['password'])) {
    exit("Contraseña incorrecta");
}

// Si llega aquí, login válido.
unset($userData['password']);

echo json_encode([
    "status" => "success",
    "message" => "Login exitoso",
    "data" => $userData
]);

$stmt->close();
$conn->close();
?>

```

### 3-Conexión a la BBDD.POST. Registrar a un usuario para probar el loggin anterior. (Register.php)

Ya sabemos un poco sobre conexión a base de datos según lo visto en el punto anterior. Ahora vamos al revés, os doy el script entero de (Register.php) y lo que haremos será desmenuzar los puntos que no comprendamos.

```php
<?php
header("Content-Type: application/json");

ini_set('display_errors', 1);
ini_set('display_startup_errors', 1);
error_reporting(E_ALL);

$host = getenv("DB_HOST");
$db   = getenv("DB_NAME");
$user = getenv("DB_USER");
$pass = getenv("DB_PASS");

$conn = new mysqli($host, $user, $pass, $db);

header("Content-Type: application/json");

if ($conn->connect_error) {
    echo json_encode(["status" => "error", "message" => "Error de conexión"]);
    exit;
}

$nombre = $_POST['nombre'] ?? null;
$email = $_POST['email'] ?? null;
$pass_input = $_POST['password'] ?? null;

if (!$nombre || !$email || !$pass_input) {
    echo json_encode(["status" => "error", "message" => "Faltan datos"]);
    exit;
}

$hashedPassword = password_hash($pass_input, PASSWORD_DEFAULT);

$sql = "INSERT INTO usuarios (nombre, email, password) VALUES (?, ?, ?)";

try {
    $stmt = $conn->prepare($sql);
    $stmt->bind_param("sss", $nombre, $email, $hashedPassword);
    $stmt->execute();

    echo json_encode([
        "status" => "success",
        "message" => "Usuario registrado correctamente",
        "user_id" => $stmt->insert_id
    ]);

} catch (Exception $e) {
    echo json_encode([
        "status" => "error",
        "message" => "Error al registrar usuario",
        "mysql_error" => $conn->error
    ]);
}

if (isset($stmt)) {
    $stmt->close();
}

$conn->close();
?>

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

###### Otras cosas importantes

En el (`INSERT`) que acabamos de usar, hacemos un "if" con el execute`if ($stmt->execute())` . Esto no lo hacíamos en el punto 2 de esta práctica cuando veíamos el loggin.

Esto tiene sentido en este caso y no en el del punto anterior porque `execute()` te dice si la consulta se ejecutó correctamente o falló (true o false). En este caso, si no se insertó nada, falla y devuelve false, por tanto, debemos devolver un error.


Sin embargo, En el login anterior, se hacía una `SELECT` y no un `INSERT`, y en este caso, `execute()` devuelve true siempre que la consulta haya terminado correctamente, a pesar de que haya devuelto 0 registros, esto pasa porque la consulta es válida aunque NO DEVUELVA RESULTADOS.  Por eso se usa después:

```php
if ($result->num_rows === 0)
```

y no el if directamente sobre el método execute.



### 4-Conexión a la BBDD. GET que devuelve la info de todos los usuarios. (All_users.php)

En este caso, no hace falta explciar tanto ya que hemos visto muchas cosas, este formulario es el ejemplo vivo de un get en el que no se envía ninguna información. A continuación os dejo el código. Recordar que en un formulario html que use get, los parámetros se envían por query-param:

```php
<?php
header("Content-Type: application/json");

$host = getenv("DB_HOST");
$db   = getenv("DB_NAME");
$user = getenv("DB_USER");
$pass = getenv("DB_PASS");

$conn = new mysqli($host, $user, $pass, $db);

if ($conn->connect_error) {
    die(json_encode(["status" => "error", "message" => "Error de conexión: " . $conn->connect_error]));
}


if ($_SERVER['REQUEST_METHOD'] !== 'GET') {
    http_response_code(405);
    die(json_encode(["status" => "error", "message" => "Método no permitido"]));
}

// Consulta
$sql = "SELECT id, nombre, email FROM usuarios";

$stmt = $conn->prepare($sql);
if (!$stmt) {
    die(json_encode(["status" => "error", "message" => "Error al preparar consulta: " . $conn->error]));
}

$stmt->execute();
$result = $stmt->get_result();

$usuarios = [];

while ($row = $result->fetch_assoc()) {
    $usuarios[] = $row;
}

echo json_encode([
    "status" => "success",
    "message" => "Usuarios obtenidos correctamente",
    "data" => $usuarios
]);

$stmt->close();
$conn->close();
?>

```

Ojo cuidado, una cosa importante es que en este coso tenemos que recorrer fetch_assoc() según lo que explicamos ya en el bloque important del punto 2.



### 5- Esquema del HTML

A modo esquematico, el body html hará las peticiones así (el put y el delete siguen haciendo lo que debían hacer en la práctica anterior utilizando los fetch en JS). En primer lugar, el HTML  (El primer POST y el segundo POST) harán las peticiones así:

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

En las prácticas siguientes veremos como utilzar el PUT y el DELETE de forma práctica y real, borrando registros de la BD y editando cuestiones de la BD.

### **6- Práctica**

1-Realiza la práctica guiada anterior e intenta que por cada usuario que se registre suba una foto del usuario dentro de la carpeta uploads.

2-Crea dos formularios HTML (en peticiones_productos.html) **<u>que NO usarán javascript.</u>** Ambos deberán hacer una petición GET y un POST respectivamente. Usarán boton de tipo submit y la información no se recogerá dinámicamente si no que se hará directamente desde el html.

- El POST del formulario html apuntará al script: productos_form.php. Lo que hará este script sera conectarse a la BD para registar un producto junto con su ID en una base de datos llamada productos y una breve descripcion. Deberás crear la tabla productos para ello.

- El GET recogerá la información de un producto en específico (quey -param), en este caso, no será un json si no que el script "info_productos.php" deberá devolver un echo con la información entera del producto.