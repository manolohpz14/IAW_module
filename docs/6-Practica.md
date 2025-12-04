

## Práctica Guiada 6. Conexiones a BD

Lo nuevo de esta práctica es que implementaremos los métodos DELETE y PUT para borrar info de la base de datos, al mismo tiempo usaremos cookies y haremos redirecciones.

En primer lugar querremos una página de incio llamada (inicioIAW.html) donde podamos acceder a una hipotética página web de nuestro módulo. Este html tendrá dos formularios. Uno para registrarnos y otro, para iniciar sesión.

![image-20251204132803044](/home/manolo/.config/Typora/typora-user-images/image-20251204132803044.png)

## 1-HTML de login_register.html (form anterior)

el html de los dos formularios (foto anterior), es este:

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

Observamos que cada formulario hace peticiones a scripts php distintos:

![image-20251204133333891](/home/manolo/.config/Typora/typora-user-images/image-20251204133333891.png)

- Visto esto, el de register_IAW.php es igual que en la práctica anterior, así que nos olvidamos de él ya que no hay nada nuevo.

- El script de login_IAW.php es un poco distinto, ya que si inicias sesión correctamente no te manda un json felicitandote, si no que te redirige de verdad a otro HTML donde puedes cambiar tu contraseña o borrar tu cuenta. Además, inserta dos cookies en tu navegador a las que nadie pordrá acceder nunca, solo el servidor de tu página web. Estas cookies serán la cookie de la contraseña en texto plano (recuerdo que en la BD la contraseña se haseha) y la otra cookie será el email de la persona que inicia sesión.

dejo por aquí solo el script login_IAW.php ya que el otro es igual que en la práctica anterior:

## 2-Script login_IAW.php

```php
<?php
header("Content-Type: application/json");

$host = getenv("DB_HOST");
$db = getenv("DB_NAME");
$user = getenv("DB_USER");
$pass = getenv("DB_PASS");

try {
    $dsn = "mysql:host=$host;dbname=$db;charset=utf8mb4";
    $pdo = new PDO($dsn, $user, $pass, [
        PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
        PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC,
        PDO::ATTR_EMULATE_PREPARES => false
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

    setcookie(
        "clave_plana",                  // Nombre
        $pass_input,                    // Valor (contraseña a pelo)
        [
            "expires" => time() + 3600,   // 1 hora
            "path" => "/",
            // "secure"   => true,         // Activar sólo en HTTPS
            "httponly" => true,            // NO accesible desde JavaScript
            "samesite" => "strict"         //La cookie solo se envia a paginar del mismo domino
        ]
    );

    setcookie(
        "email",                  // Nombre
        $email,                    // Valor (contraseña a pelo)
        [
            "expires" => time() + 3600,   // 1 hora
            "path" => "/",
            // "secure"   => true,         // Activar sólo en HTTPS
            "httponly" => true,            // NO accesible desde JavaScript
            "samesite" => "strict"         //La cookie solo se envia a paginar del mismo domino
        ]
    );

    // Redirigir a la página de inicio
    header("Location: inicioIAW.html");
    exit;

} catch (Throwable $e) {

    header("Content-Type: application/json");

    echo json_encode([
        "status" => "error",
        "message" => $e->getMessage()
    ]);
}

```

La parte importante es que con

```php
<?php
// Redirigir a la página de inicio
header("Location: inicioIAW.html")
```

Inserto un header en la respuesta del servidor al cliente (response). Este header permitirá al cliente hacer la redirección al nuevo HTML y se hace automáticamente así ya que el formulario de incio es de tipo submit luego sigue redirecciones (el siguiente fragmente de código es de la parte de HTML que posee `login_register.html` del punto anterior).

```html
<form action="Login.php" method="POST">
    <h2>Inicia sesión en la pagina web</h2>
    <input type="text" name="email" placeholder="email">
    <input type="password" name="password" placeholder="Contraseña">
    <button type="submit">Enviar</button>
</form>
```

Otra parte importante es que insertamos cookies:

```php
<?php
    setcookie(
        "clave_plana",                  // Nombre
        $pass_input,                    // Valor (contraseña a pelo)
        [
            "expires" => time() + 3600,   // 1 hora
            "path" => "/",
            // "secure"   => true,         // Activar sólo en HTTPS
            "httponly" => true,            // NO accesible desde JavaScript
            "samesite" => "strict"         //La cookie solo se envia a paginar del mismo domino
        ]
    );

    setcookie(
        "email",                  // Nombre
        $email,                    // Valor (contraseña a pelo)
        [
            "expires" => time() + 3600,   // 1 hora
            "path" => "/",
            // "secure"   => true,         // Activar sólo en HTTPS
            "httponly" => true,            // NO accesible desde JavaScript
            "samesite" => "strict"         //La cookie solo se envia a paginar del mismo domino
        ]
    );
```

donde httponly nos permite que la cookie no sea accesible desde javascript y solo viaje por http al servidor y con samesite strict nos aseguramos que no se envíe a otros servidores que no sean el nuestro.



## 3-HTML inicio_IAW.html

Si iniciamos sesión correctamente, entonces se nos redirige al HTML (inicio_IAW.HTML) y además se nos insertan las cookies anteriores. Mostramos captura de ambas cosas:

La imagen muestra el HTML que te aparece una vez inicias sesión

![image-20251204134707751](/home/manolo/.config/Typora/typora-user-images/image-20251204134707751.png)

Lo siguiente sería la captura de las cookies en el navegador:

![image-20251204134922756](/home/manolo/.config/Typora/typora-user-images/image-20251204134922756.png)

dejo aquí el código HTML de inicio_IAW.html

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

    <script>
        async function fetch_put() {
            let carta = document.querySelector(".carta_put");
            let div_put = document.querySelector(".resultado_put");
            let boton_put = document.querySelector("#button_put");

            let input_pass = document.querySelector('input[name="password"]');
            let input_pass2 = document.querySelector('input[name="password_repeated"]');


            if (div_put.innerHTML !== "") {
                div_put.innerHTML = "";
                boton_put.innerHTML = "Cambiar contraseña";
                input_pass.disabled = false;
                input_pass2.disabled = false;
                boton_put.disabled = false;

                return;
            }

            let password = input_pass.value;
            let password_repeated = input_pass2.value;

            try {

                input_pass.disabled = true;
                input_pass2.disabled = true;
                boton_put.disabled = true;

                const response = await fetch("change_password.php", {
                    method: "PUT",
                    headers: { 
                        "Content-Type": "application/json"
                    },
                    body: JSON.stringify({
                        "password": password,
                        "password_repeated": password_repeated
                    })
                });

                const data = await response.json();
                div_put.innerHTML = data.message;

                if (data.status === "success") {
                    boton_put.innerHTML = "Contraseña actualizada";
                } else {
                    boton_put.innerHTML = "Volver a intentar";
                    input_pass.disabled = false;
                    input_pass2.disabled = false;
                    boton_put.disabled = false;
                }

                carta.style.height = carta.scrollHeight + "px";

            } catch (error) {
                div_put.innerHTML = "Error inesperado";
                boton_put.innerHTML = "Volver a intentar";
                input_pass.disabled = false;
                input_pass2.disabled = false;
                boton_put.disabled = false;

                console.error(error);
            }
        }

        async function fetch_delete() {

            let div_delete = document.querySelector(".resultado_delete");
            let boton_delete = document.querySelector("#button_delete");

            if (div_delete.innerHTML !== "") {
                div_delete.innerHTML = "";
                boton_delete.innerHTML = "Borrar";
                return;
            }

            try {
                const response = await fetch("delete_user.php", {
                    method: "DELETE",
                    redirect: "follow"
                });

                if (response.redirected) {
                    alert("Usuario borrado");
                    window.location.href = response.url;
                    return;
                }

                // CASO 2: NO HAY REDIRECCIÓN → error JSON
                const data = await response.json();
                div_delete.innerHTML = data.message;
                boton_delete.innerHTML = "Volver a intentar borrado";

            } catch (error) {
                console.error("Error:", error);
                div_delete.innerHTML = "Error inesperado";
            }
        }

    </script>
</head>

<body>
    <h1 style="color:green">Sesión iniciada correctamente. Elige qué quieres hacer con el usuario</h1>
    <div class="contenedor">
        <div class="carta_delete carta">
            <form>
                <h2>Borra tu usuario de la base de datos</h2>
                <button id="button_delete" type="button" onclick="fetch_delete()">Enviar</button>
            </form>
            <div class="resultado_delete"></div>
        </div>
        
        <div class="carta_put carta">
            <form>
                <h2>Cambia la contraseña del usuario</h2>
                <input type="password" name="password" placeholder="Contraseña">
                <input type="password" name="password_repeated" placeholder="Repite la contraseña">
                <button id="button_put" type="button" onclick="fetch_put()">Enviar </button>
            </form>
            <div class="resultado_put"></div>
        </div>
    </div>

</body>

</html>
```

Además vemos también que cada formulario hace dos peticiones:

-El primero, hace la petición con javascript a **fetch_delete()** que a su vez, ejecuta el script **delete_user.php**

-El segundo, hace la petición con javascript a **fetch_put()** que a su vez, ejecuta el script **change_password.php**

Ambas funciones están en Inicio_IAW.html, que son las que hacen las peticiones a los scripts via JS y no via form HTML. Además, ninguna de estas peticiones envía body, veremos esto después.

Veamos cada uno de estos scripts para ver como manejan las solicitudes del cliente:

## 4-Script delete_user.php

El objetivo es borrar el usuario de la persona que ha iniciado sesión. Reconocerá quién ha iniciado sesión ya que la contraseña y el usuario estarán en las cookies. El código es el siguiente:

```php
<?php

// No enviar output antes de cookies o header()

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

    if (!isset($_COOKIE["email"])) {
        throw new Exception("No hay cookie de email.");
    }
    if (!isset($_COOKIE["clave_plana"])) {
        throw new Exception("No hay cookie de contraseña.");
    }

    $email_cookie = $_COOKIE["email"];
    $pass_cookie  = $_COOKIE["clave_plana"];

    // BUSCAR USUARIO EN BD
    // --------------------------------------------------------------------------

    $stmt = $pdo->prepare("SELECT id, email, password FROM usuarios WHERE email = ?");
    $stmt->execute([$email_cookie]);
    $userData = $stmt->fetch();

    if (!$userData) {
        throw new Exception("El usuario no existe en la BD.");
    }

    //VALIDAR CONTRASEÑA CON LA COOKIE
    // --------------------------------------------------------------------------

    if (!password_verify($pass_cookie, $userData["password"])) {
        throw new Exception("Contraseña incorrecta o cookie inválida.");
    }

    //BORRAR FOTOS USARIO

    $rutaCarpetaUsuario = __DIR__ . "/uploads/" . $email_cookie;

    // Función para borrar una carpeta completa
    function borrarCarpetaCompleta($ruta) {
        if (!is_dir($ruta)) return;

        $archivos = array_diff(scandir($ruta), ['.', '..']);
        foreach ($archivos as $archivo) {
            $rutaCompleta = $ruta . "/" . $archivo;
            if (is_dir($rutaCompleta)) {
                borrarCarpetaCompleta($rutaCompleta);
            } else {
                unlink($rutaCompleta); //comando para borrar un archivo
            }
        }
        rmdir($ruta);
    }

    borrarCarpetaCompleta($rutaCarpetaUsuario);

    //BORRAR USUARIO
    // --------------------------------------------------------------------------

    $stmt_del = $pdo->prepare("DELETE FROM usuarios WHERE id = ?");
    $stmt_del->execute([$userData["id"]]);

    //BORRAR COOKIES DE EMAIL Y CONTRASEÑA
    // --------------------------------------------------------------------------

    setcookie("email", "", [
        "expires"  => time() - 3600,
        "path"     => "/",
        "httponly" => true,
        "samesite" => "Strict"
    ]);

    setcookie("clave_plana", "", [
        "expires"  => time() - 3600,
        "path"     => "/",
        "httponly" => true,
        "samesite" => "Strict"
    ]);

    // REDIRIGIR A FORMULARIO DE LOGIN
    // --------------------------------------------------------------------------

    header("Location: login_register.html");
    exit;

} catch (Throwable $e) {

    // Si hay error → JSON
    header("Content-Type: application/json");

    echo json_encode([
        "status"  => "error",
        "message" => $e->getMessage()
    ]);
}

```

Una vez borrado el usuario, redirige a login_register.html y además, quita las cookies. Otra cosa importante es que el cliente debe aceptar esta redirección de forma manuan, ya que en este caso las peticiones no han sido via formulario de html si no vía fetch JavaScript. Esto se hace en este cacho de código de la función `fetch_put`del html `inicio_IAW.html`:

```php
try {
    const response = await fetch("delete_user.php", {
        method: "DELETE",
    });

    if (response.redirected) {
        alert("Usuario borrado");
        window.location.href = response.url;
        return;
    }

    const data = await response.json();
    div_delete.innerHTML = data.message;
    boton_delete.innerHTML = "Volver a intentar borrado";

}
```



## 5-Script change_password.php

El objetivo es cambiar la contraseña de la persona que ha iniciado sesión. Reconocerá quién ha iniciado sesión ya que la contraseña y el usuario estarán en las cookies. El código es el siguiente:

```php
<?php
header("Content-Type: application/json");

// Conexión a la BD
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


    // 1. VALIDAR COOKIES
    // ------------------------------------------------------------
    if (!isset($_COOKIE["email"])) {
        throw new Exception("No hay cookie de email.");
    }
    if (!isset($_COOKIE["clave_plana"])) {
        throw new Exception("No hay cookie de contraseña.");
    }

    $email_cookie = $_COOKIE["email"];
    $pass_cookie  = $_COOKIE["clave_plana"];


    // 2. LEER JSON DEL CLIENTE
    // ------------------------------------------------------------
    $json_raw = file_get_contents("php://input");
    $data = json_decode($json_raw, true);

    if (!$data) {
        throw new Exception("JSON inválido.");
    }

    $new_pass  = $data["password"] ?? null;
    $new_pass2 = $data["password_repeated"] ?? null;

    if (!$new_pass || !$new_pass2) {
        throw new Exception("Faltan contraseñas.");
    }

    // 3. COMPROBAR QUE LAS CONTRASEÑAS COINCIDEN
    // ------------------------------------------------------------
    if ($new_pass !== $new_pass2) {
        echo json_encode([
            "status" => "error",
            "message" => "Las contraseñas no coinciden."
        ]);
        exit;
    }

    // 4. VALIDAR USUARIO Y CONTRASEÑA ACTUAL
    // ------------------------------------------------------------
    $stmt = $pdo->prepare("SELECT id, email, password FROM usuarios WHERE email = ?");
    $stmt->execute([$email_cookie]);
    $userData = $stmt->fetch();

    if (!$userData) {
        throw new Exception("Usuario no existe.");
    }

    if (!password_verify($pass_cookie, $userData["password"])) {
        throw new Exception("La contraseña actual no es válida.");
    }

    // 5. ACTUALIZAR CONTRASEÑA EN LA BD
    // ------------------------------------------------------------
    $hashedNewPassword = password_hash($new_pass, PASSWORD_DEFAULT);

    $stmt_update = $pdo->prepare("UPDATE usuarios SET password = ? WHERE id = ?");
    $stmt_update->execute([$hashedNewPassword, $userData["id"]]);

 
    // 6. ACTUALIZAR COOKIE DE CONTRASEÑA
    // ------------------------------------------------------------
    setcookie("clave_plana", $new_pass, [
        "expires"  => time() + 3600,
        "path"     => "/",
        "httponly" => true,
        "samesite" => "Strict"
    ]);


    // 7. RESPUESTA ÉXITO
    // ------------------------------------------------------------
    echo json_encode([
        "status" => "success",
        "message" => "Contraseña actualizada correctamente."
    ]);

} catch (Throwable $e) {

    echo json_encode([
        "status"  => "error",
        "message" => $e->getMessage()
    ]);
}

```

Hace un control de errores para comprobar que coincidan, si no coincidiesen, mandaría al cliente un JSON diciendo que no coinciden, que se recoge en Inicio_IAW.html así:

```js
const data = await response.json();
div_put.innerHTML = data.message;

if (data.status === "success") { //Vemos si el código de respuesta es OK
    boton_put.innerHTML = "Contraseña actualizada";
} else {
    boton_put.innerHTML = "Volver a intentar";
    input_pass.disabled = false;
    input_pass2.disabled = false;
    boton_put.disabled = false;
}
```



## 6-Práctica

Copiando y entendiendo los scripts anteriores haz capturas del siguiente proceso:

 crea un usuario, logueate con él, cambia la contraseña, vuelve al inicio de sesión y comprueba que la contraseña funciona y por último borra el usuario.

¿Qué pasa con la carpeta la imagen del usuario una vez se elimina su cuenta?