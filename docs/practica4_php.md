# Práctica 4: PHP (Continuación de lo anterior)

Esta práctica continúa con el aprendizaje de PHP centrándose en pequeños detalle intermedias necesarios para el desarrollo web que veremos, en especial en el punto 8 veremos como trabajar sobre los disntitos datos que envia el cliente. Al igual que en la práctica anterior, cada bloque incluye explicación, ejemplos y práctica que deberás resolver.

Ojo, para resolver esta práctica debes haber resuelto la anterior porque hay conceptos previos que debes saber, por ejemplo, en el apartado 1, deberás saber cómo recorrer un array asociativo...

---

## 1. Valores *truthy* y *falsy* (importantísimo)

En PHP, ciertos valores se consideran falsos incluso si no son literalmente `false`. Esto tiene que ver mucho con los dos tipos de comparaciones que tenemos: `==`y `===`.

Esto es muy importante al usar condicionales y seguro que en algún punto os trae problemas si no lo hacéis correctamente. El siguiente código muestra para cada uno de los elementos del array si son truly o falsy.

```php
<?php
$valores = [0, 0.0, "", "0", null, false, [], [1], "hola"];
foreach ($valores as $v) {
    if ($v) { //if($v==true)
        echo var_export($v, true) . " es TRUTHY<br>"; //al poner true devuelve como un string. var_export es silimar a print_r($v, true), que de igual forma, con el true al final, devuelve un strning
    } else {
        echo var_export($v, true) . " es FALSY<br>";
    }
}
```
Debéis saber que, si ejecutáis el código anterior:

- Falsy: `0`, `"0"`, `""`, `[]`, `null`, `false`  
- Truthy: cualquier otro valor.

Veamos donde puede causar esto problemas en un código, aunque primero vamos a repasar como afecta los truthy y falsy en operadores ternarios. En la práctica anterior visteis como usar `??`:

```php
<?php
$nombre = $_GET["nombre"] ?? "invitado";
```

que significa:  usa valor si no es null ni está sin definir (variable no declarada todavía), si no usa el de la derecha. Pero muchas veces veréis esto en otros códigos.

```php
<?php
$nombre = $_GET["nombre"] ?: 'inivitado';
$nombre = $_GET["nombre"] ? "trhuthy": "falsy"
```

que significa algo completamente distinto:

Si $GET["nombre"] es truthy : `$_GET["nombre"]`
Si $GET["nombre"] es falsy : `'inivitado`

Ahora, vamos a ver un ejemplo de código que pone de manifiesto la importancia de manejar bien valores truthy y falsy:

```php
<?php
$amigos = $_GET['numero_amigos'] ?? null; 

if ($amigos) {
    echo "El usuario está activo y tiene {$amigos} amigos";
} else {
    echo "El usuario NO tiene definido el número de amigos";
}

```

Si imaginamos que el valor de `$_GET['numero_amigos']` es el string `"0"`, y en PHP `"0"` es falsy.  Así que para este valor, el código entra en el else aunque el parámetro está "presente", es decir, valga "0". En general, tenéis que tener mucho cuidado con la forma de comparar, lo anterior en la parte del if es equivalente a usar:

```php
<?php
if ($amigos==true)
```

Por lo visto anteriormente "0" es un valor falsy en php, es decir:`"0"==false` pero `"0"===false`no es cierto. De hecho se puede ver con este bloque de código:

```php
<?php
var_dump("0" == false);   // bool(true). Comparación laxa (convierte tipos)
var_dump("0" === false);  // bool(false). Comparación estricta
```

### Práctica (truthy_falsy.php)
Crea `truthy_falsy.php` . Este script recorre todas las claves del array asocitativo `$_GET` (mirar el punto de los bucles de la práctica anterior para saber como recorrer un array asociativo) y printea si sus valores son trulys o falsys (con los operadores ternarios `?`o `??`según te convenga y sin usar if), para ello, prueba a hacer la petición con los siguiente queryparams: 

```php
<?php
["sin_animo_de_lucro","", "0", "hola", " ", "i_love_iaw"]
```



## 2. Bucle: `do...while`, Uso de:`break`, `continue`

PHP ofrece un bucle que no vimos en la práctica pasada, el `do while`, además, los bucles poseen las claúsulas `break` y `continue`.

```php
<?php
$j = 1;
do {
  echo "do...while: $j<br>";
  $j++;
} while ($j <= 3);

$frutas = ["manzana", "pera", "uva", "plátano"];
foreach ($frutas as $f) {
  if ($f === "pera") continue; // salta
  if ($f === "uva") break;    // rompe
  echo "Fruta: $f<br>";
}
```
`continue` salta a la siguiente iteración, `break` rompe el bucle.

### Práctica (otros_bucles.php)
1. Crea un script que <u>imprima los números del 1 hasta infinito,</u> pero salte los múltiplos de 3 y termine al llegar a 8.  
   - Hazlo con un do while con la condición true en el while, usa un break cundo el número valga 8 y un continue cuando sea divisible entre 3, recuerda que el resto de números debe imprimirlos.
   - Después, haz lo mismo con un bucle for con un break cuando llegue a 8 y un continue cuando sea divisible entre 3, recuerda que el resto de números debe imprimirlos.



## 3. Funciones nombradas, anónimas, flecha. Uso de callbacks en funciones de orden superior como: array_filter, array_map y array_reduce (nuevo!!)

Antes de explicar este punto, que puede ser un poco confuso, me gustaría comentar algunas cosas potencialmente importante de **array_filter:**

PHP no distingue entre “arrays normales” y “asociativos” como otros lenguajes (por ejemplo, JavaScript o Python). Internamente, todo array es un mapa clave–valor, es decir:

```php
<?php
$nums = [10, 20, 30];
```

es en realidad:

```php
<?php
$nums = [
    0 => 10,
    1 => 20,
    2 => 30
];

```

Esto puede suponer problemas a la hora de realziar un array filter:

```php
<?php
$nums = [-3, -2, -1, 0, 1, 2, 3, 4];

function esPar($n) {
    return $n % 2 === 0;
}

$soloPares = array_filter($nums, 'esPar');

print_r($soloPares);

```

cuya salida es:

```php
<?php
Array
(
    [1] => -2
    [3] => 0
    [5] => 2
    [7] => 4
)

```

Si quisieramos tener los indices normales de un array, es decir, empezando en 0, deberíamos hacer lo siguiente:

```php
<?php
$soloPares = array_values(array_filter($nums, 'esPar'));
print_r($soloPares)

```

si guardas los valores, por defecto se indexará en 0.

```php
Array
(
    [0] => -2
    [1] => 0
    [2] => 2
    [3] => 4
)
```

Dicho esto sobre array_filter, vamos a ver la diferencia entre funciones nombradas, anónimas, flecha y el uso de callbacks. empezamos por los tipos de funciones que tenemos:

```php
<?php

//------------1) TIPOS DE FUNCIONES: nombradas o definidas, anónimas y flecha------------
// Función definida (con nombre)
function esPar2(int $n): bool {
    return $n % 2 === 0;
}

function duplicar(int $n): int {
    return $n * 2;
}

// Función anónima (closure), almacenada en variable, para poder ser usada más tarde
$esPositivo = function (int $n): bool {
    return $n > 0;
};

// Función flecha (arrow function), también la guardamos en variable para poder ser usada más tarde
$triplicar = fn ($n) => $n * 3;

//Definimos un array para utilizarlo
$nums = [-2, -1, 0, 1, 2, 3, 4];

```

Uso de array_filter con los tres tipos de funciones pasadas como "callbacks"

```php
<?php
//2) vamos a ver como usar array_filter con los tres tipos de funciones definidad

// a) Con función definida (named)
$soloPares = array_values(array_filter($nums, 'esPar')); // ['-2','0','2','4'] pero manteniendo claves originales

// b) Con función anónima
$soloPositivos = array_values(array_filter($nums, function (int $n): bool {
    return $n > 0;
})); // [1,2,3,4]

// c) Con función f$soloPares = array_values(array_filter($nums, 'esPar2')); // ['-2','0','2','4'] pero manteniendo claves originales
echo "<br>";
print_r($soloPares);

// b) Con función anónima
$soloPositivos = array_values(array_filter($nums, $esPositivo)); // [1,2,3,4]
echo "<br>";
print_r($soloPositivos);

// c) Con función flecha (p.ej. mayores que 1)
$mayoresQueUno = array_values(array_filter($nums, fn (int $n): bool => $n > 1)); // [2,3,4]
echo "<br>";
print_r($mayoresQueUno);
lecha (p.ej. mayores que 1)
$mayoresQueUno = array_values(array_filter($nums, fn (int $n): bool => $n > 1)); // [2,3,4]
```

Uso de array_map con los tres tipos de funciones pasadas como "callbacks"

```php
<?php
//3) vamos a ver como usar array_map con los tres tipos

// a) Con función definida
$duplicados = array_map('duplicar', $nums); // [-4,-2,0,2,4,6,8]
echo "<br>";
print_r($duplicados);

// b) Con función anónima
$alCuadrado = array_map(function (int $n): int {
    return $n * $n;
}, $nums); // [4,1,0,1,4,9,16]
echo "<br>";
print_r($alCuadrado);

// c) Con función flecha
$triplicados = array_map($triplicar, $nums); // [-6,-3,0,3,6,9,12]
echo "<br>";
print_r($triplicados);
```

Uso de array_reduce con los tres tipos de funciones pasadas como "callbacks"

```php
<?php
//3) vamos a ver como usar array_reduce con los tres tipos

$nums = [-2, -1, 0, 1, 2, 3, 4];

// a) Con función definida
function sumarSiPar(int $acc, int $n): int {
    return esPar($n) ? $acc + $n : $acc; //ojo, debe ser truly para que acumule!!
}
echo "<br>";
$sumaDePares1 = array_reduce($nums, 'sumarSiPar', 0); //El último parámetro es el alcumulador e indica por donde comenzamos
echo "Suma de pares (función nombrada): {$sumaDePares1}\n"; // 4

// b) Con función anónima (tipada en este caso)
$sumaDePares2 = array_reduce($nums, function (int $acc, int $n): int {
    return ($n % 2 === 0) ? $acc + $n : $acc;
}, 0);
echo "<br>";
echo "Suma de pares (función anónima): {$sumaDePares2}\n"; // 4

// c) array_reduce con callback (acumulador)
$sumaDePares3 = array_reduce(
    $nums,
    fn(int $acc, int $n): int => ($n % 2 === 0) ? $acc + $n : $acc,
    0
);
echo "<br>";
echo "Suma de pares (arrow function): {$sumaDePares3}\n";


```

Ojo: tanto `array_map()` como `array_filter()` funcionan perfectamente con arrays asociativos,
 pero por defecto usan solo los valores, ignorando las claves (a menos que se indique lo contrario).

```php
<?php
$precios = ["pan" => 1, "leche" => 2, "queso" => 3];

$iva = array_map(fn($p) => $p * 1.21, $precios);
echo "<br>";
print_r($iva);
/**
    Array
(
    [0] => 1.21
    [1] => 2.42
    [2] => 3.63
)
    */

```

Para mantenerlas, habría que utilizar `array_combine`

```php
<?php
$iva2 = array_combine(array_keys($precios), $iva);
echo "<br>";
print_r($iva2);
/**
    Array
(
    [pan] => 1.21
    [leche] => 2.42
    [queso] => 3.63
)
    */

```



### Práctica (mis_callbacks.php)

```php
<?php
$nums = [-5, -2, -1, 0, 1, 2, 3, 4, 5];
```

Dado el array anterior:

- Usa `array_filter` para quedarte solo con los números positivos e impares y reindixa
- Usa `array_map` para elevarlos al cuadrado
- Usa `array_reduce` para obtener la suma total de esos cuadrados.
- Muestra cada resultado (`print_r` o `echo` según el caso).

El array_filter lo harás con una función anónima, el array_map con una funcion nombrada definida previamente y el array_reduce con una función flecha.



## 4. Métodos útiles para Arrays. Desectructuración de Arrays (tanto normales como asocitativos)

Además de `array_map` y `array_filter`, vistos en la práctica y en el punto anterior existen funciones muy prácticas para el manejo de arrays. Además, veremos el concepto de "desectructurar un array", importantísimo.

```php
<?php
//------ordenar arrays------------------
$nums = [5, 2, 9, 1];
echo "<br>";
print_r($nums);
sort($nums); // ordena ascendente
echo "<br>";
print_r($nums);
rsort($nums); // descendente
echo "<br>";
print_r($nums);

//----haciendo slices--------------------

$parte = array_slice($nums, 1, 3, true); //cogemos tres elementos a partir del indicie 1 incluyendolo
echo "<br>";
print_r($parte);


$parte = array_slice($nums, 3, true); //cogemos todos los elementos a partir del índice 3
echo "<br>";
print_r($parte);


/*
Array
(
    [1] => 20
    [2] => 30
    [3] => 40
)

*/


$nums = [10, 20, 30, 40, 50];
$ultimos = array_slice($nums, -2); // últimos 2 elementos
echo "<br>";
print_r($ultimos);
/*
Array
(
    [0] => 40
    [1] => 50
)
*/

$ultimo = end($nums); // devuelve 50
echo "<br>";
echo $ultimo;


//----mergear arrays-----------------------------
$mix = [1, 2, 3];
$otros = ["a", "b"];
$combinado = array_merge($mix, $otros); // une arrays
echo "<br>";
print_r($combinado);


//------destructuración, defino tres variables de forma rapidísima
$valores = ["x", "y", "z"];
list($a, $b, $c) = $valores; 
echo "<br>";
echo $a; // x

//-----------------Buscar y verificar en arrays
$colores = ["rojo", "verde", "azul"];
if (in_array("verde", $colores)) echo "Verde está";
echo "<br>";
echo array_search("azul", $colores); // devuelve índice. Solo devuelve la primera clave

```
Para arrays-asociativos:

```php
<?php
//--------------------slices
$animales = [
    "a" => "perro",
    "b" => "gato",
    "c" => "pez",
    "d" => "loro"
];

$trozo = array_slice($animales, 1, 2, true);
echo "<br>";
print_r($trozo);
/*
Array de salida
(
    [b] => gato
    [c] => pez
)
*/

$ultimo = end($animales);
echo "<br>";
echo $ultimo; // loro

$ultimos = array_slice($animales, -2, null, true);
echo "<br>";
print_r($ultimos);
/*
Array de salida
(
    [c] => pez
    [d] => loro
)
*/
```

También ordenamos

```php
<?php
$personas = ["juan" => 30, "ana" => 25, "pedro" => 35];
sort($personas); // pierde las claves
echo "<br>";
print_r($personas);
```

resultado

```php
Array
(
    [0] => 25
    [1] => 30
    [2] => 35
)
```

El merge funciona sin problemas también para arrays asociativos

```php
<?php
$personas = ["juan" => 30, "ana" => 25, "pedro" => 35];
sort($personas); // pierde las claves
echo "<br>";
print_r($personas);


$a = ["x" => 1, "y" => 2];
$b = ["z" => 3];
echo "<br>";
print_r(array_merge($a, $b));

/*

Array
(
    [x] => 1
    [y] => 2
    [z] => 3
)
*/
```

desectructuración de arrays_asocitavos:

```php
<?php
$a = ["x" => 1, "y" => 2];
$b = ["z" => 3];
echo "<br>";
print_r(array_merge($a, $b));
```

in_array y array_search funcionan igual también

```php
<?php
$colores = ["rojo" => "#f00", "verde" => "#0f0"];
echo in_array("#0f0", $colores);
echo array_search("#0f0", $colores); 
```

### Práctica (mas_metodos_arrays.php)

Crea un array con los números `[9, 1, 5, 2, 7]`.

- Ordénalos en orden ascendente.
- Muestra el primer (menor) y el último (mayor) valor del array.
- Luego ordénalos en descendente y muestra el nuevo orden.



Crea dos arrays:

- `$frutas = ["manzana", "pera", "kiwi"];`
- `$bebidas = ["agua", "zumo"];`
- Combínalos con `array_merge()` en un único array llamado `$lista`,
   y muestra su contenido con `print_r()`.



Dado el array $colores = ["rojo", "verde", "azul"];

- Desestructura los tres valores en variables individuales `$a`, `$b`, `$c` usando `list()`,
   y muestra una frase como: `Los colores primarios son rojo, verde y azul`



Crea el array: `$animales = ["perro", "gato", "pez", "loro"]`

- Verifica si `"gato"` está dentro del array con `in_array()`.

- Busca el índice o posición de `"loro"` con `array_search()`.



Crea el siguiente array asociativo con nombres y edades  `$personas = [
    "juan" => 30,
    "ana" => 25,
    "pedro" => 35,
    "marta" => 28
];`

- Muestra todas las personas y sus edades.
- Ordena el array por edad ascendente manteniendo las claves si usar la función predefinida que no veremos `asort`
-  Muestra el índice del más joven y el más mayor. Te recomiendo buscar en el array de valores el número más alto y después buscar el indice. Ambo está hecho en ejemplos en esta práctica.
-  Verifica si existe la persona `"ana"` con en el array de keys.



## 5. Métodos importantes para Strings

```php
<?php
$texto = "  Hola Mundo desde PHP  ";
echo strlen($texto); // longitud
echo "<br>";
echo strtolower($texto);
echo "<br>";
echo strtoupper($texto);
echo "<br>";
echo trim($texto); // elimina espacios
echo "<br>";
echo str_replace("PHP", "el servidor", $texto);
echo "<br>";
echo strpos($texto, "Mundo"); // posición
echo "<br>";
echo substr($texto, 2, 4); // subcadena
echo "<br>";

//Aceder yb recorrer strings
$texto = "Hola Mundo";

echo $texto[0]; // H
echo "<br>";
echo $texto[1]; // o
echo "<br>";
echo $texto[5]; // M
echo "<br>";

$texto = "Hola";

for ($i = 0; $i < strlen($texto); $i++) {
    echo "<br>";
    echo $texto[$i] . "\n";
}

$texto = "Hola";
$texto[0] = "M";
echo "<br>";
echo $texto; // Mola
```
También puedes dividir y unir cadenas:

```php
<?php
$frase = "uno,dos,tres";
$partes = explode(",", $frase); // ["uno","dos","tres"]
echo implode(" - ", $partes); // uno - dos - tres
```

### Práctica (metodos_strings.php)
1. Crea `strings.php` y crea una funcion llamada prueba que sobre una cadena prueba e imprime todos los métodos anteriores.  
2. Añade un `$_GET["texto"]` después de  y realiza: longitud, conversión a mayúsculas, y búsqueda de una palabra concreta ("comida"). Esto se hará realizando un explode y un array_search normalizando cada elemento del array pasandolo a mayúsculas y haciendole un trim.

## 6. Manejo de errores: `try`, `catch`, `finally`

Esto es fundamental de muchos lenguajes y permite manejar excepciones sin detener el script.

Cuando el código dentro de un bloque `try` puede generar un error, se encierra dentro el codigo dentro de `try { ... }`.
 Si dentro de ese bloque ocurre una excepción, PHP interrumpe la ejecución normal del `try` y salta directamente al bloque `catch`, donde se puede capturar y manejar ese error de manera segura.El bloque `finally` se ejecuta siempre, ocurra o no una excepción.

**También vemos como: throw new Exception("No se puede dividir entre 0");**  que significa que el programa está generando (lanzando) una excepción de forma intencional cuando ocurre una situación que consideramos un error, en este caso intentar dividir entre cero.

```php
<?php
try {
    $divisor = $_GET["divisor"] ?? 0;
    if ($divisor == 0) throw new Exception("No se puede dividir entre 0");
    echo 10 / $divisor;
} catch (Exception $e) {
    echo "Error: " . $e->getMessage(); //ojo e->getMessage() es la ejecución de un objeto.
} finally {
    echo "<br>Bloque finally ejecutado siempre.";
}
```

### Práctica (mi_ip.php)
Crea una funcion en PHP que:

1. Reciba un parámetro `ip` por la URL (`?ip=192.168.1.10`).
2. Separe la IP en 4 bloques usando `explode(".")`.
3. Verifique que haya exactamente 4 partes.
4. Compruebe que cada bloque sea numérico (utiliza is_digit) y esté entre 0 y 255.
5. Si falla alguna validación, lance `throw new Exception("mensaje de error")` y recoja la excepción con un catch y dejando claro que la ip, no es correcta.
6. En conclusión, usa: `try`, `catch` y `finally` para que el script no se detenga nunca.

### Atención: Exception vs Error y uso de Throwable.

Fijémonos el en siguiete código:

```php
<?php
try {
 $divisor = $_GET["divisor"] ?? 0;

 if ($divisor == 0) {
     throw new Exception("No se puede dividir entre 0");
 }

 $resultado = 10 + ["esto genera un error"];
 echo 10 / $divisor;

} catch (Exception $e) {
 echo "Excepción: " . $e->getMessage();

} catch (Error $e) {
 echo "Error del motor PHP: " . $e->getMessage();

}

finally {
 echo "<br>Bloque finally ejecutado siempre.";
}
?>
```

En PHP, una Exception representa errores *controlables* dentro de la lógica de la aplicación: condiciones que tú esperas y decides manejar, como validar datos o evitar divisiones por cero. En cambio, un Error representa fallos del *motor de PHP*, cosas que no se pueden hacer a nivel de lenguaje, como sumar un número con un array o llamar a una función inexistente. Las excepciones las lanzas tú para controlar el flujo; los errores los genera PHP cuando detecta algo imposible o inválido en tiempo de ejecución.

## 7. Parámetros de ruta (continuación de Query Params). Peticiones desde clientes HTML y fetch con JavaScript (inline)

<u>Lo que vas a ver la forma básica y manual de manejar rutas en PHP usando Apache, sin frameworks.</u>

Hasta ahora usamos query params (`?nombre=Juan`).  
También podemos pasar parámetros en la ruta, por ejemplo:  
`http://localhost/usuario/juan`

Para que esto funcione, el servidor (Apache) debe tener habilitado `mod_rewrite` y un archivo `.htaccess` que redirija todas las peticiones al archivo principal (`index.php` ) tal y como hicimos en la práctica 2.

```php
<?php
// index.php
$path = $_SERVER["REQUEST_URI"]; //url solicitada
$partes = explode("/", trim($path, "/")); // ["usuario", "ana"] pensadlo, ambas funciones vistas antes
$ultimo = end($partes); // "ana"

$metodo = $_SERVER["REQUEST_METHOD"]; // GET, POST, PUT, DELETE...

echo "<h2>Ruta accedida: " . htmlspecialchars($path) . "</h2>";
echo "<p>Último segmento: <strong>" . htmlspecialchars($ultimo) . "</strong></p>";
echo "<p>Método HTTP: <strong>$metodo</strong></p>";

// Ejemplo interesante: comportamiento distinto según método que se pida
if ($metodo === "GET") {
    echo "<p>Mostrando información del usuario <strong>$ultimo</strong>.</p>";
} elseif ($metodo === "POST") {
    echo "<p>Creando un nuevo recurso llamado <strong>$ultimo</strong>.</p>";
} else {
    echo "<p>Método no manejado.</p>";
}
?>
```

### Práctica (index.php)
Ejecuta el código anterior con GET POST PUT y DELETE mediante postman y haz capturas de pantalla, prueba a enviar información a ver si la recibe correctamente. Importa que envíes información o da igual la info que envíes (por ejemplo en un post) .

Después de hacer las peticiones con postman, elabora un formulario llamdo "formulario.html" que mediante 4 formularios html's (dentro de formulario.html) haga las peticiones al script anterior con GET POST PUT Y DELETE, (resuelto en el siguiente script.) Qué problema encuentras con PUT Y DELETE? (Pues que no deja como action en el HTML)

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
            height: 13rem;
            border: 1px solid rgb(1, 1, 1);
            text-align: center;
            margin-top: 1rem;
            border-radius: 1rem;
            padding: 1rem;
            background-color: rgb(222, 255, 252);
            transition: height 0.5s ease-in-out;
            overflow: hidden;
        }

        form {
            display: flex;
            flex-direction: column;

        }
        input {
            margin: 0px auto;
            margin-top: 0.5rem;
            max-width: 10rem;

        }
        button {
            margin: 0px auto;
            margin-top: 0.5rem;
        }
    </style>

    <script>
        async function fetch_put() {
            let carta= document.getElementById("carta_put")
            let div_put = document.getElementById("resultado_put")
            let boton_put = document.getElementById("button_put")
            if (div_put.innerHTML != "") {
                div_put.innerHTML = ""
                boton_put.innerHTML = "Enviar PUT"
                carta.style.height= "15rem"
            }
            else {
                let put_value = document.getElementById("put-msg").value
                try {
                    const response = await fetch("usuario/ana", {
                        method: "PUT",
                        headers: { "Content-Type": "application/json" },
                        body: JSON.stringify({ "mensaje": put_value })
                    })
                    const respuesta_text = await response.text()
                    div_put.innerHTML=respuesta_text
                    boton_put.innerHTML = "Borrar salida PUT"
                    carta.style.height= carta.scrollHeight + "px"   
                }

                catch (error) {
                    div_put.innerHTML=error
                    boton_put.innerHTML = "Borrar salida PUT"
                }
            }
        }


        async function fetch_delete() {
            let carta= document.getElementById("carta_delete")
            let div_delete = document.getElementById("resultado_delete")
            let boton_delete = document.getElementById("button_delete")
            if (div_delete.innerHTML != "") {
                div_delete.innerHTML = ""
                boton_delete.innerHTML = "Enviar DELETE"
                carta.style.height= "15rem"
            }
            else {
                let delete_value = document.getElementById("delete-msg").value
                try {
                    const response = await fetch("usuario/ana", {
                        method: "DELETE",
                        headers: { "Content-Type": "application/json" },
                        body: JSON.stringify({ "mensaje": delete_value })
                    })
                    const respuesta_text = await response.text()
                    div_delete.innerHTML=respuesta_text
                    boton_delete.innerHTML = "Borrar salida DELETE"
                    carta.style.height= carta.scrollHeight + "px"  
                }
                catch (error) {
                    div_delete.innerHTML=error
                    boton_delete.innerHTML = "Borrar salida PUT"
                }
            }
        }

    </script>

</head>

<body>
    <div class="contenedor">

        <div class="carta">
            <form action="/usuario/ana" method="GET">
                <h2>Esto sería un GET</h2>
                <input type="text" name="mensaje" placeholder="Pon tus datos">
                <button type="submit">Enviar</button>
            </form>
        </div>

        <div class="carta">
            <form action="ejemplo_post.php" method="POST" enctype="multipart/form-data">
                <h2>Esto sería un POST</h2>
                <input type="text" name="usuario" placeholder="Pon tus datos">
                <input type="text" name="clave" placeholder="Pon tus datos">
                <input type="file" name="foto">
                <button type="submit">Enviar </button>
            </form>
        </div>

        <div class="carta" id="carta_put">
            <form>
                <h2>Esto sería un PUT</h2>
                <input id="put-msg" type="text" placeholder="Pon tus datos">
                <button id="button_put" type="button" onclick="fetch_put()">Enviar PUT </button>
            </form>
            <div id="resultado_put"></div>
        </div>

        <div class="carta" id="carta_delete">
            <form>
                <h2>Esto sería un DELETE</h2>
                <input id="delete-msg" type="text" placeholder="Pon tus datos">
                <button id="button_delete" type="button" onclick="fetch_delete()">Enviar DELETE </button>
            </form>
            <div id="resultado_delete"></div>
        </div>
    </div>
</body>
</html>
```



##  8. Preparar una ruta para recoger información que envía el cliente. Manejar el Content-Type para recoger los datos que envía el cliente. (Opcional)

Una vez hemos visto los conceptos más básicos de php, lo que vemos ahora es cómo configurar el servidor para poder recibir distintos tipos de datos.

### 8.1 Recoger contenido de un Form Data (x-www-form-urlencoded o multipart/form-data para archivos)

Cuando los datos se envían mediante un formulario HTML tradicional, se usan los tipos de contenido `application/x-www-form-urlencoded` o `multipart/form-data` (si se incluyen archivos). Por ejemplo, un formulario podría ser:

```html
<form method="POST" action="api.php">
  <input name="usuario" value="pepe">
  <input name="clave" value="1234">
  <button type="submit">Enviar</button>
</form>

```

En el servidor PHP, en el `script api.php`se pueden leer los datos de la siguiente forma:

```php
$usuario = $_POST['usuario'];
$clave = $_POST['clave'];
echo "usuario $usuario con clave $clave logueado correctamente" //simulo que hago un loggin con los datos, como todavía no tenemos bases de datos no tiene mucho sentido, pero ya sabemos como recoger la info que envía el cliente.
```

Cuando el formulario HTML del cliente se envía con una foto (ejemplo siguiente), PHP crea automáticamente un arreglo especial llamado `$_FILES` con la información del archivo. Por ejemplo, imagina que el cliente envía lo siguiente:

```html
<form method="POST" action="api.php" enctype="multipart/form-data">
  <input type="text" name="usuario" value="pepe">
  <input type="file" name="foto">
  <button type="submit">Subir archivo</button>
</form>

```

Si en el servidor (script `subir.php` en este caso) ejecutas lo siguiente

```php
<?php
print_r($_POST);
print_r($_FILES);

```

Verás algo así:

```test
Array
(
    [usuario] => pepe
)
Array
(
    [foto] => Array
        (
            [name] => selfie.jpg
            [type] => image/jpeg
            [tmp_name] => /tmp/php8F3.tmp
            [error] => 0
            [size] => 123456
        )
)

```

Donde `$_FILES` también es un array, pero contiene otro array dentro (porque puede haber varios archivos aunque aquí sólo hay 1). En este ejemplo, el campo del formulario html se llamaba `foto`, por eso aparece una sección `[foto]`. Dentro de esa sección hay otra lista con la información del archivo que se crea de forma predeterminada:

- `name`: el nombre original del archivo (`selfie.jpg`)
- `type`: el tipo de archivo (`image/jpeg`)
- `tmp_name`: la ubicación temporal donde PHP lo guarda automáticamente
- `error`: código de error (0 significa que no hubo error)
- `size`: el tamaño en bytes (por ejemplo 123456)

Podríamos guardar el archivo enviado en el html en una carpeta del servidor de la siguiente forma añadiendo un poco más de código al script `subir.php`.

```php
<?php
print_r($_POST);
print_r($_FILES);
$archivo_temporal = $_FILES['foto']['tmp_name'];
$nombre_final = 'uploads/' . $_FILES['foto']['name']; //ruta relativa luego carpeta uploads de var/www/html (ninguna seguridad y cualquier cliente puede acceder)
move_uploaded_file($archivo_temporal, $nombre_final);
```

### 8.2 Recoger application/json o text/raw

Cuando una aplicación o un cliente envía datos en formato JSON , se usa el tipo de contenido `application/json`. Este tipo de envío es común en APIs modernas o peticiones AJAX. Por ejemplo, desde JavaScript se puede enviar algo así, tal y como hicimos en el punto 7.

```javascript
fetch("api.php", {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({ usuario: "pepe", clave: "1234" })
});
```

En el servidor PHP no se usa `$_POST` en este caso, porque PHP no interpreta automáticamente el JSON. En su lugar, se debe leer el cuerpo crudo de la petición con `php://input` y luego decodificarlo con `json_decode`:

```php
<?php
$raw = file_get_contents("php://input");
$data = json_decode($raw, true); #con true devuelve un array asociativo, ponedlo.
$usuario = $data['usuario'];
$clave = $data['clave'];
```

Si el usuario enviase text/raw, el contenido se recogería así:

```php
<?php
$raw = file_get_contents("php://input");
```

es decir, no haría falta decodificarlo a JSON (ya que se envía texto, nada más)

### 8.3 Ejemplo combinado de lo anterior (emjemplo_final_practica4.php)

El siguiente código estará también en index.php ya que está elaborado para detectar incluso rutas que no existen. Este script presenta muchas formas de trabajar con las peticiones que llegan por el body, ya sean GET por query-param, POST por formularios o JSON's, que envía el cliente, archivos que envía el cleinte o incluso texto plano.

```php
<?php
// ------------------------------------------------------------
// CONFIGURACIÓN BÁSICA
// ------------------------------------------------------------

// Siempre estableceremos la codificación de salida ya que en este caso, enviaremos texto plano al cliente, para que el navegador sepa exactamente lo que se envía.
header("Content-Type: text/plain; charset=utf-8");

// ------------------------------------------------------------
// VARIABLES DE ENTORNO DEL SERVIDOR
// ------------------------------------------------------------
$path = $_SERVER["REQUEST_URI"];
$metodo = $_SERVER["REQUEST_METHOD"];
$tipoContenido = $_SERVER["CONTENT_TYPE"] ?? 'no especificado';

// Variables para la respuesta que enviaremos en el texto de la respuesta
$data = [];
$resultado = "";

// ------------------------------------------------------------
// LÓGICA PRINCIPAL DE PROCESAMIENTO
// ------------------------------------------------------------
try {
    if ($metodo === "GET") {
        // ----------------- PETICIÓN GET, recoger query-params -----------------
        $data = $_GET;
        $resultado = "Datos recibidos por método GET.";

    } elseif (in_array($metodo, ["POST", "PUT", "DELETE"])) {

        // ----------------- recoger application/json del cliente -----------------
        if (str_contains($tipoContenido, "application/json")) {
            $raw = file_get_contents("php://input");
            $data = json_decode($raw, true);
#Ojo aqui a las funcines json_last_error() yjson_last_error_msg(), muy importantes.
            if (json_last_error() !== JSON_ERROR_NONE) {
                throw new Exception("JSON inválido: " . json_last_error_msg()); #Es una función interna de  PHP que te dice si la última llamada a json_decode() tuvo algún error.
            }

            $resultado = "Datos recibidos en formato JSON.";
        }

        // ----------------- recoger application/x-www-form-urlencoded del cliente -----------------
        elseif (str_contains($tipoContenido, "application/x-www-form-urlencoded")) {
            $data = $_POST;
            if (empty($data)){ #empty devuelve true si $data es falsy o no existe
                throw new Exception("No se recibieron datos del formulario.");
            }
            $resultado = "Datos recibidos desde un formulario (x-www-form-urlencoded).";
        }

        // ----------------- recoger multipart/form-data (con archivos) -----------------
        elseif (str_contains($tipoContenido, "multipart/form-data")) {
            $data = $_POST;

            if (empty($_FILES)) {
                throw new Exception("No se recibió ningún archivo.");
            }

            foreach ($_FILES as $nombre => $archivo) {
                if ($archivo['error'] !== 0) {
                    throw new Exception("Error al subir'{$archivo['name']}'. Código: {$archivo['error']}");
                }

                $destinoCarpeta = 'uploads/';
                if (!is_dir($destinoCarpeta)) { //¿que hace is_dir?
                    throw new Exception(" 'uploads/' no existe. Créala con permisos de escritura.");
                }

                $dest = $destinoCarpeta . basename($archivo['name']); //¿que hace basename?

                if (!move_uploaded_file($archivo['tmp_name'], $dest)) {
                    throw new Exception("No se pudo mover el archivo a la carpeta de destino.");
                }

                $data['archivo_subido'] = "Archivo '{$archivo['name']}' guardado correctamente en '$dest'.";
            }

            $resultado = "Datos recibidos desde un formulario con archivo (multipart/form-data).";
        }

        // ----------------- recoger text/plain del cliente -----------------
        elseif (str_contains($tipoContenido, "text/plain")) {
            $raw = file_get_contents("php://input");
            if (trim($raw) === "") {
                throw new Exception("El cuerpo del texto está vacío.");
            }
            $data["texto"] = $raw;
            $resultado = "Texto plano recibido (text/plain).";
        }

        // ----------------- Tipo desconocido -----------------
        else {
            throw new Exception("Formato de contenido no admitido: $tipoContenido");
        }
    } else {
        throw new Exception("Método HTTP no soportado por este ejemplo.");
    }
    
} catch (Exception $e) {
    $resultado = "Error: " . htmlspecialchars($e->getMessage());
}
echo $resultado . " " . $data;
```

### Práctica (index.php)

Todavía no tienes que saber como programar esto, así que ejecuta el código anterior con GET POST PUT y DELETE mediante postman y haz capturas de pantalla, prueba a enviar información a ver si la recibe correctamente con los tres tipos de datos distintos: urlencoded, queryparams y fotos con form/data (crea la carpeta uploads en www/var/html con los permisos adecuados). 

