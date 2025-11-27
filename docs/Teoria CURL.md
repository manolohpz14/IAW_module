# -Teoría de `curl` (Previo práctica)

### ¿Qué es `curl`?

`curl` =Client URL. Es una herramienta de línea de comandos para transferir datos con URLs. Soporta múltiples protocolos (HTTP, HTTPS, FTP, SFTP, SCP, LDAP, etc.), pero normalmente lo usamos con HTTP/HTTPS.

### Sintaxis básica

```bash
curl [opciones] [URL]
#Ejemplo simple:
curl https://jsonplaceholder.typicode.com/posts/1
```

## Flags (opciones) más comunes en HTTP

### 1- Método HTTP

`-X <METHOD>` . Especifica el método (`GET`, `POST`, `PUT`, `DELETE`, etc.) Por defecto, si no ponemos nada usa GET.

```bash
curl -X GET http://localhost:3000/ping
curl -X DELETE http://localhost:3000/delete/1
```

### 2- Encabezados

`-H "<Header>: <valor>"` . añade encabezados HTTP. Nos centramos en el encabezado "Content-Type"

```bash
curl -H "Content-Type: application/json" -H "Authorization: Bearer token" http://localhost:3000/json
```

### 3- Cuerpo de la petición (body)

Solo nos centraremos en 4 tipos de formatos (Content-Type del Header). 

- `application/x-www-form-urlencoded`

- `application/json`

- `text/plain`**, **

- `multipart/form-data`.

Ahora bien, ¿cómo se envían este tipo de formatos a través de un CURL?

- `-d "campo1=valor&campo2=valor"` → envía datos en **`application/x-www-form-urlencoded`.** De hecho, si no pones -X POST, se sigue haciendo POST por defecto.

- `-d @archivo.json` → envía el contenido de un archivo como body en formato  **`application/json`**. Está guay por si tienes el contenido del Json directamente en un archivo y no quieres escribirlo en la petición.

- `--data-raw "texto crudo"` → envía datos tal cual (útil para texto plano sin necesidad de que estén en un archivo) como por ejemplo **`text/plain`**. Aunque también se puede enviar JSON si se pone en el header :**`application/json`** sin necesidad de escribir la información en un archivo.

- `-F "campo=@archivo.txt"` → envía un archivo en **`multipart/form-data`.** Útil para enviar binarios con información (justo lo que hacen los formularios de register, enviar texto junto con la foto d perfil del usuario)

1-Ejemplos: `application/x-www-form-urlencoded`

```bash
# Enviar formulario POST URL-en (lo q hace por defecto los HTML)
curl -X POST "https://httpbin.org/post" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "username=ana&password=secreto"
```

2-Ejemplos: `archivo.json`. La ruta de `archivo.json` debe ser **relativa a donde ejecutas el comando** o absoluta.

```bash
# el archivo.json de ejmpl contiene {"name":"Ana","role":"admin"}
curl -X POST "https://httpbin.org/post" \
  -H "Content-Type: application/json" \
  -d @archivo.json
```

3-Ejemplos: `--data-raw "texto crudo"`   ¿Por qué usar `--data-raw` en lugar de `-d`?

- `-d` también sirve, pero interpreta algunas cosas (por ejemplo `@archivo` lo trata como archivo).

- `--data-raw` asegura que el string se mande tal cual (sin interpretar).

  Si usas JSON con `--data-raw`, sí debes poner `-H "Content-Type: application/json"` porque el servidor necesita saber que lo que le mandas es JSON.

```bash
curl -X POST "https://httpbin.org/post" \
  -H "Content-Type: text/plain" \
  --data-raw "Este es un mensaje en texto plano\nSegunda línea"
  
 # Enviar JSON inline sin fichero (mínimo)
curl -X POST "https://httpbin.org/post" \
  -H "Content-Type: application/json" \
  --data-raw '{"title":"Hola","body":"Mundo"}'
```

4-Ejemplos:`-F "campo=@archivo.txt"` → envía un archivo en `multipart/form-data`. `-F "campo=valor"` → envía también campos normales de texto.

- file=@/ruta/a/mapa.png` → adjunta el archivo (ruta relativa o absoluta, igual que con `@archivo.json`)

```bash
curl -X POST http://localhost:3000/map \
  -F "file=@/ruta/a/mapa.png" \
  -F "mission=recuperar_muro" \
  -F "commander=Erwin"
```

Ojo: Si usas `-d` sin especificar `-X`, curl automáticamente usa `POST` y así te lo ahorras.



### 4- Envío de datos dentro de la propia URL.

Los query params siempre van en la URL, después del `?`. Es decir, nunca viajan en el cuerpo de la petición.  Para hacer un envío de query-params a través de un get, se hace de la siguiente forma:

```bash
curl --get "https://api.ejemplo.com/buscar" \
  -d "q=hola mundo" \
  -d "categoria=libros usados" \
  -d "autor=Manolito"

```

Ojo, No uses `-X GET` cuando envías query params con `-d`. Usa `--get` . Esto es un error conceptual, porque:

- `-d` (o `--data`) se utiliza para enviar cuerpo (POST/PUT)
- `GET` no tiene cuerpo

Poniendo `--get` o incluso `G`, obligamos a que -d se convierta en un query-param. En el caso de que se quiera enviar caractéres codificamos en UTF-8 conviene usar --data-urlencode.

```bash
curl -G "https://api.ejemplo.com/buscar" \
  --data-urlencode "q=hola mundo" \
  --data-urlencode "categoria=libros usados" \
  --data-urlencode "autor=El churumbel"
```



### 5- Mostrar respuesta y otras opciones de curl

- Por defecto imprime la respuesta en stdout.
- `-i` . muestra también encabezados de la respuesta.
- `-s`. modo silencioso (no muestra barra de progreso ni errores).
- `-v` . verbose (muestra detalles de la petición y respuesta).
- `-o archivo.txt` . guarda la respuesta en un archivo.
- `-O` . guarda con el mismo nombre que el archivo remoto.



