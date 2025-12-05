# Práctica

Esta práctica guiada consiste en ejecutar distintos bloques de código y responder a las preguntas que se plantean a lo largo del proceso. Para resolverlas, podéis apoyaros en búsquedas en internet o en herramientas de IA. La idea es que os creeis un proyecto con información que os iré pasando en cada bloque de texto. Para ello, deberás crear una carpeta y seguir las instrucciones aquí especificadas.

El objetivo es que, a través de estas preguntas y de la ejecución de ejemplos, entendáis mejor cómo montar un servidor en PHP y, al mismo tiempo, adquiráis nociones básicas de Docker y Docker Compose: uso de volúmenes, bind mounts y redes bridge.

A nivel personal, el objetivo es que vayáis creando vuestro propio proyecto a la vez que leéis y comprendéis la siguiente documentación técnica. Como avanzábamos antes, la primera parte de Docker Compose trata sobre la instalación. No tenéis que saber cómo funciona docker-compose desde un punto de vista demasiado técnico pero sí deberéis saber que existe Docker Hub, de donde salen las imágenes y cuáles suelen ser las claves más comunes en los YAML que permiten la creación de contenedores a partir de docker compose. En cuanto a Apache, veremos una configuración básica con las opciones más usuales.

## 1 Instalacion de Apache descentralizada con Docker

En esta parte de la práctica vais a montar un entorno similar a XAMPP pero usando Docker y Docker Compose. En lugar de instalar manualmente Apache, MySQL y PHP en vuestro ordenador (como hace XAMPP), levantaréis contenedores que se comunican entre sí para daros el mismo resultado. La estrcutura del proyecto que os debeis crear es la siguiente:

```makefile
project/
│── docker-compose.yml
│── Dockerfile
│── www/
│   ├── index.php
│   └── .htaccess
│   └── productos.php
└── apache/
    └── 000-default.conf

```

En los puntos que veremos a continuación, os daré el código necesario para la ejecución.

### 1.1 docker-compose.yml:

```yaml
services:
  apache-php:
    image: php:8.2-apache
    container_name: apache-php
    ports:
      - "8080:80"
    volumes:
      - ./www:/var/www/html
    depends_on:
      - db
    restart: always
    build:
      context: .
      dockerfile: Dockerfile

  db:
    image: mysql:8.0
    container_name: mysql-db
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: rootpass
      MYSQL_DATABASE: testdb
      MYSQL_USER: testuser
      MYSQL_PASSWORD: testpass
    ports:
      - "3306:3306"
    volumes:
      - db_data:/var/lib/mysql

  phpmyadmin:
    image: phpmyadmin/phpmyadmin
    container_name: phpmyadmin
    restart: always
    ports:
      - "8081:80"
    environment:
      PMA_HOST: db
      MYSQL_ROOT_PASSWORD: rootpass
    depends_on:
      - db

volumes:
  db_data:

```

1. **apache-php**: Imagen base: `php:8.2-apache`. Es el servicio que actúa como servidor web y intérprete de PHP. Sirve las páginas desde la carpeta `./www` (en tu máquina) que se monta dentro del contenedor en `/var/www/html`. Tiene extensiones de PHP para conectarse a MySQL (`pdo_mysql`, `mysqli`).

2. **db**: Imagen base: `mysql:8.0`. Contenedor que levanta una base de datos MySQL. Contiene algunas variables de entorno que s epueden ver en el repo de la imagen en dockerhub (tal y como hemos visto en clase sólo utilzaremos las siguiente variables de entorno).
   
   - Variables de entorno:
     - `MYSQL_ROOT_PASSWORD` : contraseña del usuario administrador.
     - `MYSQL_DATABASE` : base creada automáticamente al iniciar.
     - `MYSQL_USER` y `MYSQL_PASSWORD`: usuario adicional para que se conecte tu aplicación.
     
     Ojo, los datos de este servicio se guardan en un volumen (`db_data`), de manera que no se pierden al reiniciar o borrar el contenedor. Podeis investigar los comandos que tienen docker para ver qué volúmenes tenéis en vuestro ordenador y cómo visualizarlos y borrarlos, pero es opcional. Solo teneis que entender que en el momento que se asocia un volumen a una carpeta de un contenedor, los datos persisten.
   
3. **phpmyadmin**
   - Imagen base: `phpmyadmin/phpmyadmin`.
   - Es un servicio extra para tener una interfaz web de administración de MySQL. En nuestro caso, se accede desde el navegador en `http://localhost:8081`. No lo veremos por ahora pero ofrece una interfaz grafica más o menos amigable.
   

---

Explicados los tres servicios, la instalación funciona de la siguiente manera. Con el código anterior:

 - Levantas 3 contenedores. Cada uno tiene el número estandar de puertos y cada uno ejecuta el programa correspondiente en el puerto usualmente destinado a su ejecución, por ejemplo, apache corre en el puerto 80 de su propio contenedor pero se mapea con el 8080 de mi host.

 - De igual forma, el resto de contenedores correran los programas específicos en los puertos correspondientes.

**[1-PREGUNTA]: ¿En qué puertos internos de los contenedores corren los programas anteriores y a qué puertos del host se mapean?**

 - Ahora bien, los programas se pueden comunicar entre sí gracias a que docker incluye a cada servicio en una interfaz red bridge que permite la comunicación entre ellos.

---

**[2-PREGUNTA]: Imagina que yo instalase los programas independientemente cada uno en mi host, sin usar docker. ¿Qué puertos de qué maquinas utilzarían por defecto? ¿Mediante que interfaz de red se comunicaría unos con otros?**

**[3-PREGUNTA]: Quiero que expliques que significa esto:**

```yaml
services:
  apache-php:
   image: php:8.2-apache
   volumes:
      - ./www:/var/www/html
   build:
      context: .
      dockerfile: Dockerfile
      
  db:      
    volumes:
      - db_data:/var/lib/mysql
```

Todo en docker funciona a través de una "imagen previa".  Una imagen en Docker es como una plantilla o fotografía inmutable que contiene todo lo necesario para ejecutar un programa, es decir, a partir de esa plantilla se puede ejecutar practicamente el servicio que quieras.  Habitualmente, esa plantilla se suele crear a través de un Dockerfile, que es un archivo que contiene las instrucciones de creación de la plantilla. Aunque no es siempre así, el caso más habitual es coger la imagen de Docker Hub, aplicacion web que funciona como repositorio de imagenes para los servicios más populares (como apache). De hecho, incluso en el archivo del Dockerfile, se suele empezar desde una imagen previa que se encontrará en DockerHub.

**[4-PREGUNTA]: Más abajo, encontrarás el Dockerfile que se usa en esta práctica, ¿qué imagen de DockerHub está utilizando?**

   En ejemplo anterior es un caso muy raro, porque parece ser que el servicio se instala a partir de la imagen "php:8.2-apache". Esto no es cierto, porque abajo tenemos un "build", que nos indique que la imagen se construye a partir del docker file (llamado "Dockerfile"), que se encuentra en nuestro directorio actual. En el caso de que tanto image como build estén sobre un mismo "services", image será el nombre que asocia Docker internamente a la imagen.

---

## 2 El Dockerfile

Lo más importante es que, desde el Dockerfile, cargamos módulos de apache con a2enmod, que utilziaremos más adelante:

![image-20251017151111330](./img/image-20251017151111330.png)

En particular, el dockerfile usado en la instalación es el siguiente.

![image-20251204111145513](./img/image-20251204111145513.png)

Y esto se hace así en el Dockerfile, donde construimos nuestra imagen de Docker.

```dockerfile
FROM php:8.2-apache

# Instalamos extensiones de PHP para MySQL. docker-php-ext-install es un script que viene incluido en las imágenes oficiales de PHP (como php:8.2-apache).
RUN docker-php-ext-install pdo pdo_mysql mysqli


#Es un comando que activa módulos de Apache. rewrite Es un módulo de Apache que permite cambiar la forma en la que se interpretan las URLs. .htaccess es un archivo donde escribes las reglas que mod_rewrite ejecuta.
RUN a2enmod rewrite headers expires


```

Antes hemos comentado que los Dockerfile pueden crear por si mismos una imagen que después se ejecute como contenedor (gracias a los comandos FROM, LABEL, RUN, COPY, ADD, WORKDIR, EXPOSE, ENV, ARG, CMD, ENTRYPOINT, VOLUME) De hecho, si tenemos un Dockerfile con código en nuestro ordenador, ejecutando la siguiente linea de código, podríamos crear una imagen propia.

```bash
docker build -t nombre_imagen:tag -f Dockerfile .
```

- **`docker build`** : es el comando para construir una imagen.
- **`-t nombre_imagen:tag`** : asigna un nombre y una etiqueta (tag) a la imagen.
- **`-f Dockerfile`** : indica el archivo Dockerfile a usar. No es obligatorio si el archivo se llama exactamente `Dockerfile`.
- **`.` (punto final)** : ruta del *contexto de build*, normalmente la carpeta actual donde está el Dockerfile y los archivos que quieres copiar dentro de la imagen.

En el siguiente ejemplo, crearíamos una imagen llamada miapp:1.0

```bash
docker build -t miapp:1.0 .
```

Volviendo al Dockerfile de nuestro proyecto, al principio tenemos: FROM php:8.2-apache. Si esta imagen la hubiera creado yo anteriormente a través de un Dockerfile, docker la sacaría de mi host. En caso de no exister, la saca de Dockerhub. Ese es el flujo normal. En el Dockerfile anterior, utilizamos algunos módulos interesantes para nuestro archivo de configuración (siguiente punto) que cargamos gracias a la funcionalidad docker-php-ext-install de apache. Os hago la siguiente pregunta de la que deberéis busca información.

**[5-PREGUNTA]:**

- **¿Qué módulos cargamos?**
- **¿Para qué srive cada uno?**
- **¿Qué módulo se utilizaría (no está) para capturar y filtrar número de peticiones? (rate limit)**. Si no sabes qué es el rate limit, deberás indicarlo aquí también

---

## 3 apache/000-default.conf: (explicamos cada uno de ellos)

El objetivo de lo siguiente es hablar del archivo de configuración inicial de apache y de cada una de sus peculiaridades. Al igual que con la parte anterior de la práctica, os iré haciendo preguntas al respecto de lo que vaya saliendo, poco a poco.

![image-20251017151010293](./img/image-20251017151010293.png)

---

### 3.1 El código del 000-default.conf

```makefile
<VirtualHost *:80>
    ServerAdmin webmaster@localhost
    DocumentRoot /var/www/html

    <Directory /var/www/html>
        # Opciones de directorio
        Options -Indexes +FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>

    # Seguridad con cabeceras
    Header always set X-Content-Type-Options "nosniff"
    Header always set Referrer-Policy "strict-origin-when-cross-origin"
    Header always set Content-Security-Policy "default-src 'self'; object-src 'none'; base-uri 'self'"


    # Autenticación para la carpeta /admin. Ahora mismo no tenemos ni siquiera la carpeta creada y menos el fichero con
    #la contraseña, sería interesante (aunque no lo pido en la práctica, que probéis a hacerlo por vosotros mismos.)
    <Directory /var/www/html/admin>
        AuthType Basic
        AuthName "Zona restringida"
        AuthUserFile /etc/apache2/.htpasswd
        Require valid-user
    </Directory>

    # Control de caché para estáticos (mejora rendimiento)
    <FilesMatch "\.(jpg|jpeg|png|gif|css|js|ico|woff2?)$">
        ExpiresActive On
        ExpiresDefault "access plus 1 month"
    </FilesMatch>

    # Restringir carpeta sensible solo a una IP (permite solo esa IP)
    <Directory /var/www/html/dev-tools>
        Require ip 192.168.1.100
    </Directory>

    # Logs
    ErrorLog ${APACHE_LOG_DIR}/app-error.log
    CustomLog ${APACHE_LOG_DIR}/app-access.log combined
</VirtualHost>

```

---

### 3.2 Analizando el 000-default.conf

Vamos a ver todas las opciones del archivo anterior

---

#### 3.2.1 Bloque de sitio virtual host

Vemos qué significa esto:

```
<VirtualHost *:80>
    ServerAdmin webmaster@localhost
    DocumentRoot /var/www/html
```

- Empieza con la etiqueta `\<VirtualHost *:80>` y termina con su cierre  `\</VirtualHost>`. Con eso Apache define un  "bloque de sitio” que escucha en el puerto 80 de *todas* las interfaces (`*`).  Dentro del contenedor, Apache escucha en el 80; fuera, tú lo ves en `http://localhost:8080` porque Docker hace el mapeo `8080:80`. Si algún día tienes más "sitios", puedes crear más bloques `VirtualHost` (incluso uno por dominio) y diferenciarlos con `ServerName`.

**[6-PREGUNTA]: ¿Qué pasa si reutilizo la etiqueta VirtualHost? ¿Podría servir "dos sitios" con reglas distintas?.**

- `ServerAdmin webmaster@localhost` es solo un correo de contacto que Apache muestra en algunas páginas de error y mete en los logs. No afecta al funcionamiento; puedes poner tu email real o ignorarlo.

- `DocumentRoot /var/www/html` indica la carpeta pública desde la que Apache sirve archivos. En tu setup, esa ruta está montada desde tu máquina con `./www:/var/www/html`, así que lo que guardes en `./www` es lo que ve Apache y lo que se sirve al navegador (Es decir cuanto tu pongas en el navegador https://midonio/inicio.html) Ese incio.html que se visualizará saldrá de la carpeta /var/www/html.  

---

#### 3.2.2 Bloque Directory /var/www/html

	<Directory /var/www/html>
	    # Opciones de directorio
	    Options -Indexes +FollowSymLinks
	    AllowOverride All
	    Require all granted
	</Directory>

 `\<Directory /var/www/html>` aplica reglas específicas a esa carpeta:`Options -Indexes +FollowSymLinks` controla funcionalidades dentro del directorio.

<u>Opción 1:</u> `-Indexes` desactiva el listado de directorios. Es más seguro y profesional.

**[7-PREGUNTA]: Esta explicación puede que no quede muy clara, especifica qué significa con un ejemplo. Lanza tu servidor con esta opción y sin esta y comprueba por ti mismo (adjuntando captura de pantalla) que efectivamente sucede así.**

<u>Opción 2:</u> `+FollowSymLinks` permite a Apache seguir enlaces simbólicos. Útil si apuntas subcarpetas a otros lugares. Si quieres ser aún más estricto, existe `SymLinksIfOwnerMatch` (solo sigue el symlink si el propietario coincide), pero `FollowSymLinks` es lo habitual.

**[8-PREGUNTA]: Pon ejemplos de qué significa esto definiendo distintos propietarios: Para guiarte un poco, piensa que la carpeta /var/www/privado con propietario manolo  se crea un enlace con /var/www/html/privado. Si alguien accede por red, ¿podría ver el contenido de /var/www/html/privado? Para ello investiga con que usuario se indenficaría dicha persona al entrar a la web e intentar la redirección.**

`AllowOverride` es otra directiva de Apache que en este caso controla qué directivas están permitidas dentro de un `.htaccess` (archivo que veremos más tarde). Le estás diciendo a Apache en este caso: “Dentro de esta carpeta, haz caso a los archivos `.htaccess` que haya”. Significa que Apache permitirá archivos `.htaccess` en esa carpeta (en este caso:`/var/www/html`) y en cualquiera de sus subcarpetas (ojo, lo de cualquiera de sus subcarpetas es importante y más tarde veremos porqué, ya que afecta al funcionamiento de un servidor en producción y provoca que, habitualmente, no se cree el archivo .htacess.  En conclusión: es muy común dejarlo en "All" durante desarrollo y como hemos dicho, en producción, por rendimiento, muchos mueven esas reglas al propio VirtualHost y ponen `AllowOverride None`. Lo veremos más tarde.

`Require all granted` es la regla de acceso (desde Apache 2.4). Significa “permite acceso a cualquiera”. Si no la pones (o niegas el acceso), podrías ver errores 403 (los códigos de errores los he comentado ya mucho en clase pero los veremos cuando generemos nuestros propios endpoints). Es la forma moderna de los antiguos `Allow from all`. Ejemplo: solo permitir a la IP `192.168.1.100` sería de la forma: Require ip 192.168.1.100.

**[9-PREGUNTA]: ¿Cómo se podría negar el acceso a una sola IP a nuestro servidor Apache?**

---

#### 3.2.3 Logs de apache

```php
    ErrorLog ${APACHE_LOG_DIR}/app-error.log
    CustomLog ${APACHE_LOG_DIR}/app-access.log combined
```

${APACHE_LOG_DIR} y  ${APACHE_LOG_DIR} son variables de entorno de apache (la misma en este caso) y se guarda en: `/etc/apache2/envvars` lugar físico donde se encuentran las ENV

- ErrorLog ${APACHE_LOG_DIR}/app-error.log

  **[10-PREGUNTA]: ¿Qué tipo de errores guarda el log app-error.log?**

  **[11-PREGUNTA]: ¿Qué significa ${APACHE_LOG_DIR} y donde se guardan dichas variables?**

- CustomLog ${APACHE_LOG_DIR}/app-access.log combined

  **[12-PREGUNTA]: Durante las prácticas anteriores, has hecho peticiones http a un servidor. ¿Qué es lo que se vería en este caso en app-access.log y que relacion tiene con la práctica anterior?**

**<u>Importante</u>**:

Dentro del contenedor, esos archivos viven en `/var/log/apache2/`. Para verlos puedes hacer `docker exec -it apache-php bash` y luego `tail -f /var/log/apache2/app-error.log`. Si prefieres tenerlos en tu host, puedes montar un volumen para `logs/` y así inspeccionarlos con tu editor. Otra opción avanzada es enviar logs a stdout/stderr para leerlos con `docker logs`, pero con ficheros sueles tener más control y lo recomiendo sobremanera. Los comandos mencionados a ejecutar dentro del contenedor son los siguientes:

```bash
docker exec -it apache-php bash
cd /var/log/apache2
tail -f app-access.log
```

**[13-PREGUNTA]: Ya contestaste que era lo que se guardaba en app-access.log, una vez visto esto, y realizado al menos una petición al servidor, metete al contenedor y comprueba qué es lo que se guarda mediante una captura de pantalla**

---

#### 3.2.4 Restricciones a ciertos directorios

```php
<Directory /var/www/html/admin>
    AuthType Basic
    AuthName "Zona restringida"
    AuthUserFile /etc/apache2/.htpasswd //lugar del server con la contraseña de la carpeta hasheada
    Require valid-user
</Directory>
```

`<Directory /var/www/html/admin>` Tenemos una regla de autenticación sobre este directorio, aunque si os fijáis y tal y como pongo en el comentario encima de la regla, el directorio no existe, aunque no afecta a Apache. Lo que has visto en ese bloque de configuración es un ejemplo de autenticación HTTP básica en Apache. Sirve para proteger con usuario y contraseña un directorio concreto, en este caso `/var/www/html/admin`.

1-<u>`AuthType Basic`</u> indica que se usará el método de autenticación más sencillo, llamado *Basic*. Este funciona enviando el usuario y la contraseña codificados en base64 (debes saber qué es ya que muchas veces los servidores envía binarios en base64) en cada petición. Por eso, en la práctica, siempre se recomienda usarlo junto con HTTPS para que las credenciales viajen cifradas. 

**[14-PREGUNTA]  Auth Type sirve para proteger recursos. El estandar a día de hoy ha cambiado a JWT. Busca información al respecto sobre qué es. ¿Todo el mundo debe tener derecho a acceder a cualquier endpont?**

2-<u>`AuthName "Zona restringida"`</u> define el texto que verá el usuario cuando el navegador muestre la ventana emergente pidiéndole credenciales. No afecta a la seguridad, es simplemente informativo, pero ayuda a diferenciar distintas áreas protegidas de un mismo servidor.

3- <u>`AuthUserFile /etc/apache2/.htpasswd`</u> indica dónde está guardado el archivo que contiene los usuarios y contraseñas autorizados. E nuestro proyecto no lo tenemos definido porque, de hecho la carpeta /admin no existe. Ese archivo `.htpasswd` NO debe estar dentro de la carpeta pública del servidor web, porque si no, cualquiera podría descargarlo. En él se almacenan las credenciales en formato cifrado (normalmente con algoritmos como MD5, SHA o bcrypt).

4-<u>`Require valid-user`</u> define quién puede acceder. En este caso, “valid-user” significa que cualquiera de los usuarios definidos en el archivo `.htpasswd` podrá entrar. Pero hay otras variantes: se puede especificar `Require user juan` para permitir solo a un usuario concreto, o `Require user juan pedro` para permitir únicamente a esos dos. También existe la posibilidad de trabajar con grupos de usuarios, usando `Require group admins`, donde los grupos se definen en un archivo adicional llamado `.htgroup`.

---

#### 3.2.5 Cacheado de archivos dentro del bloque /var/www/htmlo:

```makefile
<FilesMatch "\.(jpg|jpeg|png|gif|css|js|ico|woff2?)$">
    ExpiresActive On
    ExpiresDefault "access plus 1 month"
</FilesMatch>
```

Esto significa: “Aplica las siguientes reglas a todos los archivos cuyo nombre termine en `.jpg`, `.jpeg`, `.png`, `.gif`, `.css`, `.js`, `.ico` o `.woff/woff2` (fuentes web)”. Es decir, cualquier recurso estático habitual de una web (con estático nos referimos a archivos que sualmente no van a cambiar y que se van a mandar tal cual). 

Dentro de ese bloque, `ExpiresActive On` activa el módulo de caducidad de caché. Y `ExpiresDefault "access plus 1 month"` establece que el navegador DEL CLIENTE puede mantener esos archivos en caché durante un mes a partir del momento en que los descarga (por eso se llama access + 1 año). Ojo, el navegador del cliente sería el que cachearía dicha información, evitando así incluso que la petición llegue al servidor.

Es importante saber que todo lo que pongamos como "Expire" en apache, se generan las cabeceras http Expires y Cache-Control en la respuesta del servidor. Algo así cuando se accede a un css por primera vez con la regla: `ExpiresByType text/css "access plus 1 month"`

```makefile
HTTP/1.1 200 OK
Content-Type: text/css
Cache-Control: max-age=2592000
Expires: Tue, 05 Jan 2026 12:00:00 GMT
Content-Length: 4521
```

De hecho, en este punto utilizaremos dos métodos para cachear:

```makefile
-ExpiresByType, hace uso del módulo de apache "mod_expires". Toca tanto la cabecera Cache-Control como la Expires.

-Header set Cache-Control, hace uso del módulo de apache "mod_headers". En este caso solo tocaríamos la cabecera "Cache_Control" y no la cabecera "Expires". No es malo ya que la cabecera "Expires" está en desuso.
```

De hecho, esto:

```makefile
<FilesMatch "\.(jpg|jpeg|png|gif|css|js|ico|woff2?)$">
    ExpiresActive On
    ExpiresDefault "access plus 1 month"
</FilesMatch>
```

con "Header set" sería así:

```makefile
<FilesMatch "\.(jpg|jpeg|png|gif|css|js|ico|woff2?)$">
    Header set Cache-Control "public, max-age=2592000"
</FilesMatch>

```

Como vemos, afecta directamente a la cabecera HTTP, generando esta salida:

```makefile
HTTP/1.1 200 OK
Content-Type: text/css
Cache-Control: max-age=2592000
Content-Length: 4521
```

Con todo esto, si un usuario entra varias veces a tu web, su navegador no volverá a pedir al servidor la imagen del logo, los estilos CSS o el JavaScript en cada carga, sino que reutilizará los que ya descargó hasta que pase un mes. Veamos qué mas reglas podemos definir:

-Para **imágenes y vídeos grandes**, podrías poner más tiempo (Esto no viene en nuestro archivo de configuración): Fijate que la directiva es ExpiresByType 

```makefile
ExpiresByType image/jpeg "access plus 1 year"
ExpiresByType image/png "access plus 1 year"
ExpiresByType video/mp4 "access plus 1 year"
```

-Para **archivos CSS y JS**, quizá prefieras menos tiempo, como una semana, porque podrías actualizarlos con frecuencia durante el desarrollo (Esto no viene en nuestro archivo de configuración pero se podría poniendo):

```makefile
ExpiresByType text/css "access plus 1 week"
ExpiresByType application/javascript "access plus 1 week"
```

-Para **documentos sensibles o que cambian mucho** (como  un endpoint PHP), lo normal es directamente desactivar la caché con el OFF (Esto no viene en nuestro archivo de configuración): 

```makefile
<FilesMatch "\.(php)$">
    ExpiresActive Off
    Header set Cache-Control "no-store, no-cache, must-revalidate"
</FilesMatch>
```

"no store" significa que no quieres que nadie cachee la información. "no-cache" significa que si alguien a cacheado, como el navegador, preguntar antes al servidor qué hacer con él. "must-revalidate" significa que si la copia local está expirada o marcada como no válida, el navegador está obligado a revalidarla con el servidor antes de usarla. También tendríamos la opción de hacer esto, que es mucho más común con los archivos php:

```makefile
<FilesMatch "\.(php)$">
    Header set Cache-Control "public, no-cache, must-revalidate"
</FilesMatch>
```

De forma que el servidor deja al navegador cachear pero este siempre pregunta antes de usar su cache, es decir: si el archivo cambia, lo pide al servidor. Si el archivo no ha cambiado, lo coge de la caché.  El ejemplo anterior generaría la siguiente response.

```makefile
HTTP/1.1 200 OK
Date: Thu, 05 Dec 2025 11:32:10 GMT
Server: Apache/2.4.58 (Ubuntu)
Content-Type: text/html; charset=UTF-8
Cache-Control: public, no-cache, must-revalidate
Pragma: no-cache
Expires: 0
Content-Length: 5120

```

El navegador que recibe la respuesta será capaz de cachearla según las opciones propuestas.

**[15-PREGUNTA]** **¿Cual es la diferencia entre Expires y Cache-Control? Busca las distintas opciones de Cache-Control y qué singifican.**

---

#### 3.2.6 Seguridad con cabecera en el deafult.conf

Sigamos viendo otras partes, como las cabeceras http, que ya hemos visto que pueden ser muy importantes en la interaccion del cliente con el servidor. Ya sabemos que cuando un navegador (cliente) habla con un servidor web, lo hace usando el protocolo HTTP. Cada petición y cada respuesta viaja en forma de texto, con varias líneas de información llamadas cabeceras (headers). 

Estas cabeceras son como *metadatos*: pequeños mensajes que acompañan a la petición o a la respuesta para dar más contexto sobre qué se envía, cómo debe interpretarse y cómo debe comportarse el navegador o el servidor. En la práctica anterior nos centramos muchísimo en la cabecera "Content-Type" o en la cabercera de "Cache", pero lo normal es que haya más cabeceras que se envíen por defecto, por ejemplo, lo siguiente muestras las cabeceras http de una petición (REQUEST):

```makefile
GET / HTTP/1.1
Host: midominio.com
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64)
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: es-ES,es;q=0.9
Referer: https://google.com
Cookie: sessionid=abc123 #Las trabajaremos mucho cuando llegue el momento. Es importante protejerlas

```

Lo siguiente muestras las cabeceras de la respuesta a la petición (RESPONSE)

```makefile
HTTP/1.1 200 OK
Date: Wed, 09 Oct 2025 18:30:00 GMT
Server: Apache/2.4.54 (Debian)
Content-Type: text/html; charset=UTF-8
Content-Length: 4523
X-Content-Type-Options: nosniff
Content-Security-Policy: default-src 'self'; object-src 'none'; base-uri 'self'
```

En nuestro codigo de configuración de Apache (en concreto la parte con el comentario # Seguridad con cabeceras) hemos colocado algunas lineas que mucho tienen que ver con las cabeceras.. Las líneas de nuestra configuración de Apache añaden cabeceras HTTP a las respuestas que el servidor envía al navegador. Estas, al igual que la parte del caché vista anteriormente, definirán distintos casos de actuación, aunque en vez de explicaroslas, quiero que busquéis información al respecto y que pongáis ejemplos en la medida de lo posible.

-1)`X-Content-Type-Options "nosniff"` 

**[16-PREGUNTA]: ¿Para qué sirve y qué tiene que ver con los Content-Type estudiados hasta ahora?**

-2)`Content-Security-Policy: default-src 'self'; object-src 'none'; base-uri 'self'`

**[17-PREGUNTA]: Aquí os ayudo un poco más ya que pienso que es más complejo:**

La cabecera `Content-Security-Policy` (CSP) permite al servidor indicar al navegador desde qué orígenes puede cargar recursos como scripts, imágenes o estilos. De esta forma, si en el HTML hay etiquetas que intentan cargar contenido externo no permitido, el navegador lo bloqueará automáticamente. Haciendo uso de la explicación anterior, detalla cómo funciona esta protección con algunos ejemplos además, responde a lo siguiente: qué significa que `default-src 'self'` solo deje cargar recursos del mismo dominio, qué consigue `object-src 'none'` al prohibir objetos embebidos, y cómo `base-uri 'self'` limita el uso de la etiqueta `<base>` en el documento. Muestra ejemplos de código HTML que serían permitidos y otros que serían bloqueados.

-3)`Referrer-Policy "strict-origin-when-cross-origin"`

**[18-PREGUNTA]: ¿Qué es referer en la cabecera y que hace este tipo de política?**

---

## 4 Archivo `.htaccess` (uso de reglas RewriteCond y RewriteRule)

El archivo **`.htaccess`** es un archivo de configuración especial que Apache lee cada vez que recibe una petición dentro de una carpeta concreta (mirar AllowOverride explicado en el archivo de configuración). Su nombre viene de *hypertext access* y fue creado originalmente para manejar reglas de acceso, aunque hoy en día se usa para muchas otras cosas.  La idea es sencilla: si en tu servidor tienes definido un directorio raíz (por ejemplo `/var/www/html`), puedes colocar un `.htaccess` dentro de esa carpeta o en alguna subcarpeta. Como nuestro directorio raíz es `var/www/html` Apache, cuando sirve un archivo o procesa una URL, busca si en esa ruta de la siguiente forma :

- Imagina que la Carpeta raíz definida es, como en nuestro caso: `/var/www/html`
- El cliente hace la petición: `http://localhost:8080/blog/posts/123`
- En el servidor, la ruta real: `/var/www/html/blog/posts/123`

Si AllowOverride es "ON" como en nuestro caso, Apache revisa de la siguiente forma para ver:

1. `/var/www/html/blog/posts/.htaccess`
2. `/var/www/html/blog/.htaccess`
3. `/var/www/html/.htaccess`

No hace falta convercerse para darse cuenta de que ha nivel de rendimiento es una castaña tremenda, por eso, para desarrollo, es coherente, pero no a nivel de producción, donde se pone `AllowOverride none`...

Veamos ahora qué es lo que tenemos en este archivo `.htaccess`

```bash
RewriteEngine On

# Si la URL no apunta a un archivo real...
RewriteCond %{REQUEST_FILENAME} !-f
# ... y no apunta a un directorio real
RewriteCond %{REQUEST_FILENAME} !-d
# Redirigimos todo a index.php
RewriteRule ^ index.php [L]

```

Vamos a estudiar las condiciones de este .htaccess, que como es dicho, podrían estar directamente sobre el archivo de configuración de Apache y no necesariamente en el .htaccess

---

###  4.1 La primera RewriteCond

```bash
RewriteCond %{REQUEST_FILENAME} !-f
```

`%{REQUEST_FILENAME}`  es la ruta física del archivo que corresponde a la URL pedida.`! -f`  significa "NO es un archivo normal existente".

solo se aplicará la regla si el archivo real no existe

Ejemplo:

- URL: `/style.css` , sí existe `/var/www/html/style.css` : la condición NO se cumple.

- URL: `/api/usuarios` , no existe `/var/www/html/api/usuarios` como archivo : la condición se cumple.

---


### 4.2 La segunda RewriteCond

```bash
RewriteCond %{REQUEST_FILENAME} !-d
```

Solo se aplicará la regla si la ruta pedida no es un directorio real.

Ejemplos:

- Se ejecuta la URL: `/uploads/` , sí existe `/var/www/html/uploads/` como carpeta: la condición NO se cumple.
- se ejecuta la URL: `/api/usuarios` , no existe `/var/www/html/api/usuarios` como carpeta : la condición se cumple.

¿Por qué se tienen que cumplir las dos? Porque si existe un archivo o un directorio real, nuestra configuración decide que Apache debe servirlo directamente. Este tipo de configuración suele ser común y estándar y tenéis que saber cómo funciona. En conclusión, no queremos que una petición de `style.css` acabe en `index.php` (en caso de que sí exista style.css) ni que la carpeta `uploads/` sea redirigida. Solo cuando:

- No existe dicho archivo,
- No existe dicha carpeta,
   entonces aplicamos la regla final:

Hemos visto dos ejemplos de `RewriteCond` pero, en general, las RewriteCond siguen la siguiente estructura. Estudiemos su estructura y veamos más ejemplos.

---

#### 4.3 Estudio de las Condiciones (RewriteCond) en general, con más ejemplos

Sintaxis general:

```bash
RewriteCond TESTSTRING CONDITION
```

- **TESTSTRING** : es lo que se va a evaluar (ej: `%{REQUEST_FILENAME}`, `%{HTTP_HOST}`, `%{QUERY_STRING}`, etc.).

**[19-PREGUNTA] ¿QUE OTRAS VARIABLES DE ENTORNO SE GENERAN EN APACHE CUANDO SE RECIBE UNA PETICIÓN HTTP?**

- **CONDITION** : es la comparación (ej: `!-f`, regex, strings, etc.). Al final de estos apuntes hay un anexo con regex.

   Veamos muchos ejemplo, no hace faltas que los sepáis de memoria pero sí que los entendáis.

   <u>EJEMPLOS</u>

- `RewriteCond %{REQUEST_FILENAME} !-f` : se cumple si no existe archivo.

- `RewriteCond %{REQUEST_FILENAME} -d` :  se cumple si sí existe directorio.

- `RewriteCond %{HTTP_HOST} ^www\.midominio\.com$` : se cumple si el host pedido es exactamente `www.midominio.com`.

- `RewriteCond %{HTTP_HOST} !^www\.`
   Se cumple si el host no empieza con www.

- `RewriteCond %{HTTPS} on`
   Se cumple si la conexión es HTTPS.

- `RewriteCond %{HTTPS} off`
   Se cumple si la conexión es HTTP (no cifrada).

- `RewriteCond %{QUERY_STRING} id=([0-9]+)`
   Se cumple si la URL tiene `?id=algo` en la query string.

- `RewriteCond %{QUERY_STRING} !id=`
   Se cumple si la query string no contiene `id=`.

   **TODO DE QUERY_STRING ESTO OS DEBE SONAR**: En la primera práctica vimos que mi servidor montado con NodeJS no diferenciabala lo que hubiese en los queryparams, le daba exactamente igual que hubiera más o menos campos.
   
- `RewriteCond %{REMOTE_ADDR} ^192\.168\.1\.100$`
   Se cumple si el cliente viene de la IP `192.168.1.100`.

- `RewriteCond %{REMOTE_ADDR} !^192\.168\.`
   Se cumple si el cliente **no está en la subred 192.168.*.*`.

  Ojo, Puedes poner varias `RewriteCond`: todas deben cumplirse para que se ejecute la regla que viene después. Por último, nos queda desrbir la última linea del htaccess.

---

### 4.4 Primera condición RewriteRule

```makefile
RewriteRule ^ index.php [L]
```

- **`^` en el patrón**: Como ya sabéis, en expresiones regulares, `^` significa “inicio de la cadena”. Aquí el patrón es `^`, lo que equivale a decir “cualquier cosa que empiece en el principio”, o sea, todas las URLs. Normalmente se combina con las condiciones de`RewriteCond` ya explicadas para filtrar casos, como ya haces con `!-f` y `!-d`.

- **`index.php` en el destino** Significa: “reescribe la URL para que Apache sirva `index.php`”. Esto no cambia lo que el usuario ve en el navegador, simplemente Apache decide que la petición `/productos/42` en realidad la atienda el archivo `index.php` y no por un archivo o carpeta que estuviera en `var/www/html`.

- **`[L]` (flag “Last”)** Indica que esta debe ser la última regla evaluada si se cumple. Una vez que Apache aplica esta regla, deja de mirar otras reglas de reescritura en ese contexto. Es como un `break` en un bucle de programación.

Al igual que con las `RewriteCond` vemos la sintáxsis general de las `RewriteRule`

---

### 4.5 Estudio de las RewriteRule en general

```bash
RewriteRule patrón destino [flags]
```

- **patrón** : es una expresión regular (ver Anexo) que Apache aplica sobre la URL solicitada (normalmente sobre el `REQUEST_URI` sin el dominio inicial).

**[20-PREGUNTA] ¿Que es request_URI?**

- **destino** : adónde se debe redirigir o reescribir la petición. Puede ser un archivo local (ej: `index.php`) o incluso una URL completa (`https://otro.com/`).

- **[flags]** : opciones que modifican el comportamiento (como `L`, `R`, etc.).

**[21-PREGUNTA]: Busca información sobre otro flag que se pudiera añadir e indica qué significaría**.

---

## 5. Algunos Scripts en apache y su ejecución

Veamos algunos scripts, que podemos colocar el la carpeta www para que se puedan ejecutar. Tenéis que tener claro que en el siguiente tema veremos lenguaje php desde el principio para que podáis entender estos end

### 5.1 index.php con conexión a MySQL y dos endpoints simples

Ahora un ejemplo un poco más útil, que prueba la conexión a la base de datos con las credenciales que definiste en las variables de entorno de `docker-compose.yml` (No tienen porqué ser las mismas que yo puse).

**[22-PREGUNTA] Esta pregunta es sólo para ver si estáis siguiendo la lectura ya que lo que pregunto a continuación ya ha sido explicado ¿En qué casos php y bajo qué peticiones del cliente ejecutará index.php APACHE?**

```php
<?php
header("Content-Type: application/json");

// Parámetros de conexión (los mismos que en docker-compose.yml)
$host = "db";           // nombre del servicio de MySQL en docker-compose
$user = "testuser";
$pass = "testpass";
$dbname = "testdb";

// Conectar a MySQL
$conn = new mysqli($host, $user, $pass, $dbname);

// Verificar conexión
if ($conn->connect_error) {
    http_response_code(500);
    echo json_encode(["error" => "Error de conexión a la BD: " . $conn->connect_error]);
    exit();
}

// Obtener la ruta solicitada
$uri = parse_url($_SERVER['REQUEST_URI'], PHP_URL_PATH);

// ENDPOINT: /api/usuarios
if ($uri === '/api/usuarios') {
    $sql = "CREATE TABLE IF NOT EXISTS usuarios (
        id INT AUTO_INCREMENT PRIMARY KEY,
        nombre VARCHAR(50) NOT NULL
    )";
    $conn->query($sql);

    // Insertar un usuario de prueba si la tabla está vacía
    $conn->query("INSERT INTO usuarios (nombre)
                  SELECT 'Ana'
                  WHERE NOT EXISTS (SELECT 1 FROM usuarios)");

    $result = $conn->query("SELECT * FROM usuarios");

// Recorremos todos los resultados de la consulta SQL ($result)
// fetch_assoc() devuelve cada fila como un array asociativo
//   - Las claves son los nombres de las columnas de la tabla (ej: "id", "nombre")
//   - Los valores son los datos de esa fila
    $usuarios = [];
    while ($row = $result->fetch_assoc()) {
        $usuarios[] = $row;
    }
// Al final, $usuarios será un array de arrays asociativos
    echo json_encode($usuarios);
}
// ENDPOINT: /api/ping (solo prueba de vida)
elseif ($uri === '/api/ping') {
    echo json_encode(["msg" => "pong"]);
}
else {
    http_response_code(404);
    echo json_encode(["error" => "Endpoint no encontrado"]);
}

$conn->close();

```

Además, este archivo tiene dos endpoints definidos. Debes de contestar a las siguientes dos preguntas:

**[23- PREGUNTA]:**

- **¿Cuáles son esos dos endpoints?**
- **¿Cuál es el flujo que hace Apache desde la petición del cliente hasta que devuelve la respuesta del endpoint? (Mirar .htaccess)**
- **¿Tienen verbos http definidos estos endpoints? ¿Responderían ante cualquier tipo de verbo http?**
- **¿Qué tipo de "Content-Type" envían ambos al cliente?**
- **¿Si tuvieras que darle un verbo http, que verbo le darías a cada endpoint anterior? Justifica tu respuesta**

---

### **5.2 productos.php**

```php
<?php
header("Content-Type: application/json");

// Asegurar que sea POST
if ($_SERVER['REQUEST_METHOD'] !== 'POST') {
    http_response_code(405); // Method Not Allowed
    echo json_encode(["error" => "Solo se admite POST"]);
    exit();
}

// Asegurar que Content-Type sea application/json
if (stripos($_SERVER["CONTENT_TYPE"], "application/json") === false) {
    http_response_code(400); // Bad Request
    echo json_encode(["error" => "Content-Type debe ser application/json"]);
    exit();
}

// Leer el cuerpo de la petición
$input = file_get_contents("php://input");
$data = json_decode($input, true);

// Validar JSON
if (json_last_error() !== JSON_ERROR_NONE) {
    http_response_code(400);
    echo json_encode(["error" => "JSON inválido"]);
    exit();
}

// Verificar que tenga el campo "nombre"
if (empty($data['nombre'])) {
    http_response_code(400);
    echo json_encode(["error" => "Falta el campo 'nombre'"]);
    exit();
}

// Conectar a la BD
$host = "db";
$user = "testuser";
$pass = "testpass";
$dbname = "testdb";

$conn = new mysqli($host, $user, $pass, $dbname);
if ($conn->connect_error) {
    http_response_code(500);
    echo json_encode(["error" => "Error de conexión: " . $conn->connect_error]);
    exit();
}

// Crear tabla si no existe
$conn->query("CREATE TABLE IF NOT EXISTS productos (
    id INT AUTO_INCREMENT PRIMARY KEY,
    nombre VARCHAR(100) NOT NULL
)");

// Insertar el nuevo producto
$nombre = $conn->real_escape_string($data['nombre']);
if ($conn->query("INSERT INTO productos (nombre) VALUES ('$nombre')")) {
    http_response_code(201); // Created
    echo json_encode([
        "success" => true,
        "id" => $conn->insert_id,
        "nombre" => $data['nombre']
    ]);
} else {
    http_response_code(500);
    echo json_encode(["error" => "Error al insertar producto"]);
}

$conn->close();

```



**[24-PREGUNTA]: Tal y como esta configurado apache, el archivo php anterior representa un endpoint por si mismo.**

- **¿Cómo se hace la petición a ese archivo? ¿Qué tipo de verbo http sería?** 

- **¿Qué tipo de formato de datos accepta la petición?** **¿Qué tipo de formato devuelve?**

- **Prueba a a hacer una petición con POSTMAN y otra con CURL y adjunta capturas de la respuesta.**



**[25-PREGUNTA] En esta práctica habéis configurado manualmente vuestro servidor Apache con `000-default.conf` y un `.htaccess` para redirigir (algunas) lURLs a `index.php`. Todo el control de rutas, autenticación y caché lo habéis hecho directamente en el servidor web.**

- **¿Qué cambia cuando usamos un framework como Laravel?**

- **¿Dónde se definen las rutas en ese caso: en el servidor Apache o dentro del framework?**

- **¿Qué ventajas y desventajas tiene cada enfoque (controlarlo en Apache vs. dejarlo al framework)?**

- **¿Por qué creéis que en proyectos grandes se recomienda que Apache solo apunte a `public/` y que el framework gestione las rutas y la lógica?**



**[26-PREGUNTA] A lo largo de la práctica has visto que se ha utilizado la extensión mysqli para hacer querys.**

- **¿Qué desventaja tiene esto en un servidor Web real?**
- **¿Qué es un ORM y que relación tiene con lo anterior?**
- **¿Qué ventajas ofrecen los ORM que nos proporcionan algunos frameworks como Larabel?**

---

## 6. Anexo (Cosas básis de regexp con algunos ejemplos)

- **Caracteres literales** : Un string  tal cual. Ej: `hola` encuentra “hola”.
- **Metacaracteres**:
  - `.` : cualquier carácter.
  - `^` : inicio de la cadena.
  - `$` : fin de la cadena.
  - `*` : cero o más repeticiones.
  - `+` : una o más repeticiones.
  - `?` : opcional (cero o una).
  - `[]` : conjunto de caracteres. Ej: `[0-9]` = cualquier dígito.
  - `|` : alternancia (OR). Ej: `perro|gato`.
  - `()` : agrupar.
  - `{n}` : exactamente n veces.
  - `{n,}` : al menos n veces (sin límite superior).
  - `{n,m}` : entre n y m veces.

- Ejemplos típicos (si entiendes estos ejemplos, vas bien con las regexp, si no, repásatelas por favor)

  - `^[0-9]+$` : solo números enteros.

  - `^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}$` : email básico.

  - `\.(jpg|jpeg|png)$` : archivos que terminan en `.jpg`, `.jpeg` o `.png`.

  - `^https?://`: URLs que empiezan por `http://` o `https://`. (piensa que hace)

  - `^[0-9]+([.,][0-9]+)?$`: (piensa que hace)

  - `^(0[1-9]|[12][0-9]|3[01])/(0[1-9]|1[0-2])/[0-9]{4}$`: (piensa que hace)





-------------------------------------------------------------------------------------------------------------------------







