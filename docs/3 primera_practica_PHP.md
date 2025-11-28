# Programación en PHP Usando Apache (Primera práctica)

Este documento está pensado para que avancéis con calma, son cosas muy sencillas y explciadas muy paso a paso. Cada bloque introduce una idea, muestra un ejemplo breve y termina con práctica guiada. No hay prerequisitos.

Aunque creáis que no, soy razonable :) y entiendo que docker os puede dar problemas, la práctica, está pensada para hacerla en tiempo de clase y si docker os da problema, llamadme sin miedo, por favor. En principio, los siguientes comandos de docker os deben solventar cualquier tipo de situación, los dejo por aquí y los explico.

### Preámbulo 1. Cómo ejecutar el código php con docker

Cada vez que abráis vuestro proyecto, os recomiendo ejecutar:

```bash
sudo docker compose down
sudo docker compose up --build
```

De esta forma, evitaréis problemas.

Si habéis elegido docker, vamos ahora con los errores más comunes que os podéis encontrar. Uno de los errores más comunes es el siguiente

```bash
Error response from daemon: Conflict. The container name "/<nombre_del_contenedor>" is already in use by container "<id_del_contenedor>". You have to remove (or rename) that container to be able to reuse that name.
```

Cuando ejecutáis el docker compose up, se creen tantos contenedores como servicios tenéis definidos sobre dicho archivo. En nuestro caso, se crean tres contenedores, uno de mysql, otro de apache y otro de phpmyadmin. Si alguna vez os sale ese error, tenéis dos opciones:

1- Matar ese contenedor: primero lo paras (por nombre del servicio o por id del contenedor si sabes mirarlo) y después lo matas:

```bash
sudo docker stop container_name|container_id #lo paro
sudo docker rm container_name|container_id #lo borro
```

2- Si estás en el mismo proyecto desde el cual creaste los contenedores que quieres matar, bastaría con:

```bash
sudo docker compose down -v
```

Esto cierra todos los contenedores creados en el docker-compose, ojo, debes tener definido un docker compose, si no, no irá

3- Por último, para ver todos los contenedores que está corriendo en tu sistema, podrías hacer:

```bash
sudo docker ps
```

4- Si nada de lo anterior funciona, buscar el PID del sistema que corre este contenedor y matarlo

```bash
sudo kill -9 PID
```

Esto sólo lo comento porque puede ser útil, no porque os vaya a solucionar nada.

#### Resumen ejecutar el código php con docker

1. Inicia tu servidor local de Docker (Apache según práctica 2).
2. Coloca tu scripts en una carpeta publica que pueda ser consumible (www) de la práctica anterior.
3. Es decir, no toques nada de lo que ya hay. cada vez, que hagas un cambio, NO HACE FALTA ejecutar: `docker compose up --build`  solamente guarda los cambios en el script y después accede a `http://localhost:8080/archivo.php" o al script que quieras ejecutar-

---

### Preámbulo 2. Cómo ejecutar el código php instalando apache

Si no queréis usar docker, podéis instalar apache en ubuntu 24.04, para ello, ejecutad lo siguiente:

```bash
sudo apt update
sudo apt install -y apache2 php8.3 php8.3-mysql #instalamos 3 paquetes, apache y lo necesario para correr php junto con mysql (por ahora no usaremos mysql)
sudo a2enmod rewrite headers expires #Activamos los módulos de apache, al igual que con docker, para que puedan usarse en nuestro archivo de configuración.
sudo systemctl restart apache2 #reiniciamos el servicio
```

Una vez hecho esto ejecutad los siguientes comando des copiado (Ojo con las ruta relativas y absolutas): En vuestro proyecto, tenéis el archivo apache/000-default.conf, de configuración de apache, pegadlo en:

```bash
sudo cp "/home/manolo/Escritorio/PHP inicio/apache/000-default.conf" /etc/apache2/sites-available/000-default.conf
```

Recordad también pegar el .htacces en la carpeta '/var/www/html'.

Una vez habéis copiado la config, dad permisos de lectura escritura y ejecución a '/var/www/html' y abrid dicha carpeta en vscode. Con esto ya podéis desarrollar los scripts de esta práctica

#### Resumen ejecutar el código php con apache en ubuntu

1. Inicia tu servidor local de Apache (systemctl start apache2).
2. Visita el script que quieras ejecutar:`http://localhost:8080/archivo.php" 

#### Opcional

Si no queréis trabajar directamente sobre dicha carpeta, podéis realizar enlaces simbólicos. Los siguientes comandos cumplen el cometido que se quiere.

- Pegad vuestra carpeta www dentro de /var/www/html, para ello:

  ```bash
  sudo cp -r "/home/manolo/Escritorio/PHP inicio/www"/. /var/www/html/
  ```

- Ya tenéis apache corriendo en el puerto 80!. si accedéis a http://localhost ejecutaréis directamente el index.php. Por comodidad, os recomiendo hacer enlaces simbólicos para desarrollar mas rápdido desde vscode:

  ```bash
  sudo rm -rf /var/www/html #recursive y force
  sudo ln -sf "/home/manolo/Escritorio/PHP inicio/www" /var/www/html #simbolic y force, la opción -f (force) borra el destino si ya existe (sea un archivo o un enlace, esdecir, sea lo que sea),y lo reemplaza por el nuevo enlace simbólico.
  sudo ln -sf "/home/manolo/Escritorio/PHP inicio/apache/000-default.conf" /etc/apache2/sites-available/000-default.conf
  
  ```

  Error común. Si tenéis enlaces simbólico podéis tener problemas con los permisos, recomiendo cambiarlos de la siguiente forma

  ```bash
  sudo chown -R $USER:www-data "/home/manolo/Escritorio/PHP inicio/www"
  sudo chmod -R 775 "/home/manolo/Escritorio/PHP inicio/www"
  
  sudo systemctl reload apache2 
  ```


---

### Preámbulo 3.Cómo ejecutar el código php desde intérprete

 Esto sólo os servirá en algunos puntos de la práctica y no lo recomiendo, pero es conveniente que lo sepáis.

```bash
sudo apt update
sudo apt install php
```

y posteriormente, ejecuta tu script:

```bash
php script.php
```

Así pruebas la ejecución.

----

## 1. “Hola, mundo”

```php
<?php
echo "Hola, mundo desde PHP";
```
Guarda esto en el archivo como `hola_mundo.php` según lo comentado antes

### Práctica (haz captura de lo siguiente y visítalo para ver que funciona) Archivo: hola_mundo.php
1. Cambia el mensaje por tu nombre.

2. Agrega una segunda línea adicional con `echo` que muestre la fecha actual con el comando de abajo y añade "\n" al final de "Hola, mundo desde PHP", de la siguiente forma (¿Pasa algo si no pones el punto y coma?)

   ```php
   echo date("Y-m-d");
   ```

   Nota: `date` es una estructura ya integrada en php que veremos más tarde, en este caso devuelve la fecha actual según el formato especificado. Recarga en el navegador y verifica los cambios o ejecuta con el interprete de php.
   
3. Haz lo mismo pero al final de "Hola, mundo desde PHP", añade `<br>`, qué pasa?



## 2. PHP + HTML en el mismo archivo

Puedes mezclar HTML con PHP para generar páginas dinámicas.

```php
<!DOCTYPE html>
<html>
<head>
  <meta charset="utf-8">
  <title>Mi primera página PHP</title>
</head>
<body>
  <h1>Bienvenido</h1>
  <p>
    <?php
    echo "Hoy es " . date("d/m/Y"); #El punto sirve para concatenar cadenas
    ?>
  </p>
</body>
</html>
```

### Práctica. Archivo: first_html.php
Cambia el `<h1>` por tu nombre. Añade un párrafo donde pone php que diga: “Esta página fue generada por PHP”. Fijate como se inserta el código php dentro del html



## 3. Comentarios

```php
<?php
// Comentario de una línea
# Otra forma de comentar
/*
 Comentario de varias líneas
*/
echo "lo anteror eran comentarios de tres tipos distnitos"; // fin de línea
```

### Práctica. firstcomment.php
Crea `sintaxis.php` y escribe tres líneas `echo` con distintos comentarios.



## 4. Variables y tipos básicos

Las variables empiezan con `$`. PHP infiere el tipo según el valor.

```php
<?php
$nombre = "Lucía";   // string
$edad   = 21;        // integer
$altura = 1.65;      // float
$activo = true;      // boolean
$colores = ["rojo", "verde", "azul"];
$variable = null;

echo gettype($nombre);
echo "<br>";
echo gettype($edad);  
echo "<br>";
echo gettype($altura); 
echo "<br>";
echo gettype($activo); 
echo "<br>";
echo gettype($colores); 
echo "<br>";
echo gettype($variable);

echo "Me llamo $nombre y tengo $edad años."; //interpolación de variables en PHP,
```
Nota: las variables distinguen mayúsculas y minúsculas (`$edad` ≠ `$Edad`).

### Práctica. variables.php
Crea `variables.php` y declara: `$nombre`, `$ciudad`, `$anioNacimiento`. Muestra una frase completa usando las tres variables, en vez de concatenar como en el ejemplo anterior, vas a hacer concatenación con `.` Ejemplo:

```php
echo "Soy " . $profesion . " desde " . $anio . ".";
```

En PHP, el operador `.` une (concatena) dos o más textos o variables en una sola cadena. En python, por ejemplo, es el "+"



## 5. Entrada y salida sencillas

En páginas web, la “entrada” suele venir de formularios ejecutados en el cliente. Por ahora, haremos una entrada mínima con query string (`GET`). Aquí usamos el operador: `??` que solo comprueba si existe y no es null.

```php
<?php
$nombre = $_GET["nombre"] ?? "invitado";
echo "Hola, " . htmlspecialchars($nombre);
```
Visita: `http://localhost/mi-primer-proyecto/saludo.php?nombre=Ana`

> `htmlspecialchars` evita que se ejecute HTML que el usuario pueda enviar. Es una estructura de control que conviene implementar para prevenoir ataques.

### Práctica (saludo.php)
Agrega un segundo parámetro `?ciudad=Madrid` y muéstralo. Si `ciudad` no está, muestra “(ciudad no especificada)”. 

Además prueba a mandar por el query-param `<h1>PACO</h1>`, ¿se ejecuta el html?



## 6. Operadores básicos

Operadores aritméticos, comparación estricta y lógicos.

```php
<?php
$a = 10;
$b = 3;

echo $a + $b . "<br>";
echo $a - $b . "<br>";
echo $a * $b . "<br>";
echo $a / $b . "<br>";
echo $a % $b . "<br>";

//-------una comparación estricta
$unoComoTexto = "1";
var_dump(1 === $unoComoTexto); // avanzo false
echo "<br>";
var_dump(1 == $unoComoTexto); //((¿que será))

//------ operadores lógicos
$edad = 20;
$permisoPadres = false;
echo "<br>";
var_dump($edad >= 18 || $permisoPadres); //Operador OR en AND es &&
```
`var_dump` se usa para inspeccionar valores y tipos.  Sirve para mostrar el tipo de dato y su valor exacto. No hace falta que lo pongas echo delante.

### Práctica. (operadores.php)
Crea `operadores.php`: pide `a` y `b` por `GET` param, para ello, cuando los recibas los params, serán string, para ello, pásalos a float:

```php
$a = (float)$a;
$b = (float)$b;
```

 y muestra todas las operaciones al igual que en el ejemplo. 

Añade una línea que diga si `a` es mayor o divisible entre `b` usando var_dump por delante.

Añade otra que diga si `a` es menor que `b` o es divisible entre `b` también usando var_dump por delante de la comparativa.



## 7. Condicionales

```php
<?php
    

$nota = $_GET["nota"];

//-----Ejemplo de negación !, la función is_numeric es importante
if (!is_numeric($nota)) {
    echo "<p>Error: el valor proporcionado no es un número válido.</p>";
    echo "<p>Ejemplo correcto:nota=7</p>";
    exit;
}

else {
    $nota = (int) ($_GET["nota"] ?? 0); //LO paso a INT antes
}
//-----Aquí vemos por primera vez un &&
if ($nota >= 9 && $nota <= 10) {
  echo "Sobresaliente";
} elseif ($nota >= 5 && $nota <= 10) {
  echo "Aprobado";
} else {
  echo "Insuficiente";
}
```
Prueba con: `?nota=4`, `?nota=7`, `?nota=10` (querystrings)

Arregla el código para que cuando sea >10 no ponga insuficiente.

### Práctica. (Condicionales.php)
Crea `Condicionales.php` que reciba `n` por querystring y diga si es positivo, negativo o cero. Añade un mensaje extra si `n` es par o impar y otro mensaje que diga si es mayor que 5 y menor que 10. Para ello, usa el resto del punto anterior.



## 8. Bucles (while, for, foreach)

```php
<?php
//---------while
$i = 1;
while ($i <= 3) {
  echo "while: $i<br>";
  $i++;
}

//----------for
for ($j = 1; $j <= 3; $j++) {
  echo "for: $j<br>";
}

//----------- foreach
$frutas = ["manzana", "pera", "uva"];
foreach ($frutas as $fruta) {
  echo "fruta: $fruta<br>";
}

//------------Array asociativo
$persona = [
    "nombre" => "Lucía",
    "edad" => 21,
    "ciudad" => "Madrid"
];

echo $persona["nombre"]; // Lucía

foreach ($persona as $clave => $valor) {
    echo "$clave: $valor<br>";
}

```

### Práctica (bluches.php)
- Crea que muestre la tabla de multiplicar de `n` (por `GET`).
- Crea que recorra un array de 5 libros y los imprima usando etiquetas `<li>` de html.
- Busca o prueba si los arrays pueden tener mas de un tipo distintos dentro



## 9. Métodos de arrays y arrays asociativos

```php
//------------Crear y acceder
<?php
$frutas = ["manzana", "pera", "uva"];
echo $frutas[0]; // manzana

$persona = [
  "nombre" => "Lucía",
  "edad" => 21,
  "ciudad" => "Madrid"
];
echo $persona["edad"]; // 21


//--------Añadir
$frutas[] = "naranja"; // añade al final naranja
$persona["profesion"] = "Estudiante"; // añade nueva clave

//-----------eliminar
unset($frutas[1]);      // elimina el segundo elemento, los indices empiezan por 0
$ultima = array_pop($frutas); //printea lo que devuelve
unset($persona["edad"]); // elimina una clave del array asociativo

//--------contar
echo count($frutas);  // número total de elementos
echo count($persona);  // número total de elementos

//---------array de keys y valores
$claves = array_keys($persona);
$valores = array_values($persona);
//------------Ojo usamos print_r para estructuras mas complejas no echo
print_r($claves);  // ["nombre", "ciudad", "profesion"]
print_r($valores); // ["Lucía", "Madrid", "Estudiante"]




```

### Práctica (metodos.php)

1. Crea un array `$colores = ["rojo", "verde", "azul"];`

2. Añade `"amarillo"` al final

3. Elimina `"verde"` usando `unset()`.

4. Elimina azul con un pop y guardalo en la variable color

5. Añade al final `rojo``

6. Recorre el array con `foreach` e imprime cada color en una lista HTML de `li`

   ---

1. Crea un array `$coche` con las claves: `"marca"`, `"modelo"`, `"año"`, `"color"`.

   y añade lkos valores que quieras

2. Añade una nueva clave `"precio"`.

3. Muestra las claves con `array_keys()` y los valores con `array_values()`.

4. Recorre el array original con `foreach` mostrando `"clave: valor"`.



## 10. Funciones

```php
<?php
//-----------------función tipada
function saludar(string $nombre): string {
  return "Hola, $nombre";
}

function multiplicar_2($n) {
    return 2 * $n;
}

echo saludar("Marcos") . "<br>";

//--------------Funciones flecha
// Versión corta (arrow function)
$doble = fn($n) => $n * 2; // asignada correctamente

echo $doble(3) . "<br>"; // imprime 6

//--------------maps y filters con funciones flecha
$numeros = [1, 2, 3, 4, 5];

$dobles = array_map(fn($n) => $n * 2, $numeros);
$pares = array_filter($numeros, fn($n) => $n % 2 == 0);

print_r($dobles); // [2, 4, 6, 8, 10]
print_r($pares);  // [2, 4]

//--------------maps y filters sin funciones flecha
$dobles = array_map("multiplicar_2", $numeros);

$pares = array_filter($numeros, function($n) {
    return $n % 2 == 0;
});

print_r($dobles);
print_r($pares);


```

### Práctica (funciones.php)
- Escribe una función `esPar(int $n): bool` y pruébala con varios valores.
- crea una funcion que tome dos int y los sume, pruebala introducciendo dos float: ¿qué sucede?
- ¿Es obligatorio tipar las funciones? 
- Escribe `areaTriangulo(float $base, float $altura): float` que devuelva el área.
- Crea un array `$edades = [15, 22, 30, 17, 19, 45];`

  Usa `array_filter()` con una función flecha para obtener solo las mayores o iguales de 18.

  Muestra el resultado con `print_r($mayores);`



