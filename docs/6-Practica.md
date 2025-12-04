### 3-Conexión a la BD en un script. Login de un usuario en la BBDD (Login.php)

Vamos a aprovechar el formulario de la pŕactica anterior (este):

![image-20251121105609040](/home/manolo/Escritorio/IAW/materia/docs/img/image-20251121105609040.png)

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

![image-20251121180114532](/home/manolo/Escritorio/IAW/materia/docs/img/image-20251121180114532.png)

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

### 1- Esquema del HTML de la práctica anterior

La práctica anterior terminó con tres formularios HTML (Código del Punto 6)

Estos tres formularios hacían peticiones a tres scripts de php (uno con un verbo POST, para registrar usuario, otro con un verbo POST para hacer un login del usuario y otro con un GET para recoger en un array todos los usuarios de la base de datos)

A lo largo de esta práctica, vamos a dividir esos HTML's en dos páginas distintas. En la primera página, tendremos el típico (inicio de sesión/ register)  de cualquier página web. Algo más o menos así:

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
            flex-direction: column;
            align-items: center;
            justify-content: center;
            height: 95vh;
        }

        h1 {
            text-align: center;
        }


        .contenedor {
            display: flex;
            justify-content: center;
            align-items: center;
            margin-top: 10vh;
        }

        .carta {
            border: 1px solid rgb(1, 1, 1);
            text-align: center;
            margin-top: 1rem;
            border-radius: 1rem;
            padding: 1rem;
            background-color: rgb(222, 255, 252);
            max-width: 320px;
            margin-right: 1rem;
        }

        form {
            margin-top: 0.5rem;
            display: flex;
            flex-direction: column;
            text-align: left;

        }

        input {
            max-width: 2, 5rem;
            margin: 0px auto;
            margin-top: 0.5rem;
        }

        button {
            max-width: 2, 5rem;
            margin: 0px auto;
            margin-top: 0.5rem;
        }
    </style>

</head>


<body>
    <h1>Bienvenido al formulario de Inicio de la pagina WEB de IAW</h1>
        <div class="contenedor">
            <div class="carta">
                <form action="Login.php" method="POST">
                    <h2>Inicia sesión en la pagina web</h2>
                    <input type="text" name="email" placeholder="email">
                    <input type="password" name="password" placeholder="Contraseña">
                    <button type="submit">Enviar</button>
                </form>
            </div>

            <div class="carta">
                <form action="Register.php" method="POST" enctype="multipart/form-data">
                    <h2>Registrate e inserta una imagen</h2>
                    <input type="text" name="nombre" placeholder="Nombre">
                    <input type="text" name="email" placeholder="Email">
                    <input type="password" name="password" placeholder="Contraseña">
                    <input type="file" name="imagen">
                    <button type="submit">Enviar </button>
                </form>
            </div>
        </div>

</body>

</html>
```

Aprovecho para recordar que esos dos formularios HTML usan por defecto url-encoded (y que no es el estándar de las API's REST). Si quisiéramos que estos formularios enviaran un JSON, necesitaríamos hacer un fetch, al igual que en el punto 8 de la práctica 4. De todas formas, lo dejamos así ya que funciona a la perfección.







Cuando iniciemos sesión con un usuario (y únicamente en ese caso). El usuario será redigido a otro html, donde podrá hacer tres cosas.

-Borrar su usuario (DELETE)

-Cambiar su contraseña (PUT)

-Ver a todos los usuarios registrados (GET)

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

2-Crea dos formularios HTML (en peticiones_productos.html) **<u>que NO usarán javascript salvo que sea estrictamente necesario.</u>** Ambos deberán hacer una petición GET y un POST respectivamente. Usarán boton de tipo submit y la información no se recogerá dinámicamente si no que se hará directamente desde el html.

- El POST del formulario html apuntará al script: productos_form.php. Lo que hará este script sera conectarse a la BD para registar un producto junto con su ID en una base de datos llamada productos y una breve descripcion. Deberás crear la tabla productos para ello.

- El GET recogerá la información de un producto en específico (quey -param), en este caso, no será un json si no que el script "info_productos.php" deberá devolver un echo con la información entera del producto.