### 2 Práctica

Esta práctica guiada consiste en ejecutar distintos bloques de código y responder a las preguntas que se plantean a lo largo del proceso. Para resolverlas, podéis apoyaros en búsquedas en internet o en herramientas de IA. La idea es que os creeis un proyecto con información que os iré pasando en cada bloque de texto. Para ello, deberás crear una carpeta y seguir las instrucciones aquí especificadas.

El objetivo es que, a través de estas preguntas y de la ejecución de ejemplos, entendáis mejor cómo montar un servidor en PHP y, al mismo tiempo, adquiráis nociones básicas de Docker y Docker Compose: uso de volúmenes, bind mounts y redes bridge.

------

A nivel personal, el objetivo es que vayáis creando vuestro propio proyecto a la vez que leéis y comprendéis la siguiente documentación técnica. Como avanzábamos antes, la primera parte de Docker Compose trata sobre la instalación. No tenéis que saber cómo funciona docker-compose desde un punto de vista demasiado técnico pero sí deberéis saber que existe Docker Hub, de donde salen las imágenes y cuáles suelen ser las claves más comunes en los YAML que permiten la creación de contenedores a partir de docker compose. En cuanto a Apache, veremos una configuración básica con las opciones más usuales.

### 2.1 Instlacion de Apache descentralizada con Docker

En esta parte de la práctica vais a montar un entorno similar a XAMPP pero usando Docker y Docker Compose. En lugar de instalar manualmente Apache, MySQL y PHP en vuestro ordenador (como hace XAMPP), levantaréis contenedores que se comunican entre sí para daros el mismo resultado. La estrcutura del proyecto que os debeis crear es la siguiente:

```text
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

#### <u>- docker-compose.yml:</u>

```yaml
version: "3.8"

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
    # Instalamos extensiones de PHP necesarias para MySQL
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
     - `MYSQL_ROOT_PASSWORD` → contraseña del usuario administrador.
     - `MYSQL_DATABASE` → base creada automáticamente al iniciar.
     - `MYSQL_USER` y `MYSQL_PASSWORD` → usuario adicional para que se conecte tu aplicación.
     
     Ojo, los datos de este servicio se guardan en un volumen (`db_data`), de manera que no se pierden al reiniciar o borrar el contenedor. Podeis investigar los comandos que tienen docker para ver qué volúmenes tenéis en vuestro ordenador y cómo visualizarlos y borrarlos, pero es opcional. Solo teneis que entender que en el momento que se asocia un volumen a una carpeta de un contenedor, los datos persisten.
   
3. **phpmyadmin**
   - Imagen base: `phpmyadmin/phpmyadmin`.
   - Es un servicio extra para tener una interfaz web de administración de MySQL. En nuestro caso, se accede desde el navegador en `http://localhost:8081`. No lo veremos por ahora pero ofrece una interfaz grafica más o menos amigable.

Esto funciona de la siguiente manera. Con el código anterior:

 - Levantas 3 contenedores. Cada uno tiene el número estandar de puertos y cada uno ejecuta el programa correspondiente en el puerto usualmente destinado a su ejecución, por ejemplo, apache corre en el puerto 80 de su propio contenedor pero se mapea con el 8080 de mi host.

 - De igual forma, el resto de contenedores correran los programas específicos en los puertos correspondientes.

   **[PREGUNTA]: ¿En qué puertos internos de los contenedores corren los programas anteriores y a qué puertos del host se mapean?**
   SOL: Sólo basta con mirar la configuración de docker:

   Apache-PHP: corre en el puerto interno 80 del contenedor y se mapea al puerto 8080

   MySQL: corre en el puerto interno 3306 y se mapea al 3306

   phpMyAdmin: corre en el puerto interno 80 y se mapea al 8081

   Ahora bien, los programas se pueden comunicar entre sí gracias a que docker incluye a cada servicio en una interfaz red bridge que permite la comunicación entre ellos. FIN SOL

   **[PREGUNTA]: Imagina que yo instalase los programas independientemente cada uno en mi host, sin usar docker. ¿Qué puertos de qué maquinas utilzarían por defecto? ¿Mediante que interfaz de red se comunicaría unos con otros?**

   SOL: Apache: usa por defecto el puerto 80 o 443 si lo configurasemos correctamente para https, esto se ve en el contenedor o haciendo una pequeña búsqueda por internet.

   MySQL: usa por defecto el puerto 3306, esto se ve igualmente en el contenedor o haciendo una pequeña búsqueda por internet

   phpMyAdmin: se ejecuta como aplicación web servida por Apache en el 80 .

   Evidentemente si todos estuvieran bajo un mismo host, La comunicación entre ellos sería a través de localhost. Esto causaría problemas ente phpMyAdmin y Apache, luego deberíamos hacer que phpMyAdmin corra en otro puerto. FIN SOL

   

   **[Pregunta]: Quiero que expliques que significa esto:**

   SOL: Usamos dos serivicios apache-php y db. Uno es de apache, otro de la base de datos. EN el caso de apache, hacemos un enlace simbólico (en docker llamado bind mount) entre mi carpeta www y la carpeta /var/www/html del contenedor donde está instalado apache, esto se hace para que cuando suba un script en mi proyecto a www se cree en el directorio correcto de apache en mi contendor donde apache sube los archivo estáticos. FIN SOL

   

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

   Todo en docker funciona a través de una "imagen previa".  Una imagen en Docker es como una plantilla o fotografía inmutable que contiene todo lo necesario para ejecutar un programa, es decir, a partir de esa plantilla se puede ejecutar practicamente el servicio que quieras. 

   Habitualmente, esa plantilla se suele crear a través de un Dockerfile, que es un archivo que contiene las instrucciones de creación de la plantilla. Aunque no es siempre así, el caso más habitual es coger la imagen de Docker Hub, aplicacion web que funciona como repositorio de imagenes para los servicios más populares (como apache). De hecho, incluso en el archivo del Dockerfile, se suele empezar desde una imagen previa que se encontrará en DockerHub.

   **[PREGUNTA]: Más abajo, encontrarás el Dockerfile que se usa en esta práctica, ¿qué imagen de DockerHub está utilizando?**

   SOL: Está usando una imagen descargada de dockerhub (php:8.2-apache) a la que mediante un Dockerfile, se le han añadido dependencias necesarias para la configuración de módulos en apache. Es decir, es una imagen de dockerhub un poco toqueteada. FIN SOL

   En ejemplo anterior es un caso muy raro, porque parece ser que el servicio se instala a partir de la imagen "php:8.2-apache". Esto no es cierto, porque abajo tenemos un "build", que nos indique que la imagen se construye a partir del docker file (llamado "Dockerfile"), que se encuentra en nuestro directorio actual. En el caso de que tanto image como build estén sobre un mismo "services", image será el nombre que asocia Docker internamente a la imagen.

#### <u>- Dockerfile</u>



Lo más importante es que cargamos módulos de apache con a2enmod, que utilziaremos más adelante:

![image-20251017151111330](/home/manolo/.config/Typora/typora-user-images/image-20251017151111330.png)

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

**[PREGUNTA]: **

- **¿Qué módulos cargamos? **

  SOL: `rewrite`, `expires`, `headers`. El resto son extensiones de PHP (ojo, no módulos) para conectarse a bases de datos MySQL. Realemente sólo nos bastaba con `mysqli`. FIN SOL

- **¿Para qué srive cada uno? **

  SOL:

  - rewrite: nos permite cambiar la forma en que apache responde a una petición a una URL. El ejemplo claro es que yo tengo un script llamado ejemplo.php. Para que el cliente lo ejecute deberia hacer algo parecido a:

    http://localhost:8080/ejemplo.php (ya que tenemos mapeados los puertos). Sin embargo, no te gusta que el cliente sepa que está ejecutando un script o simplemente quieres hacer que el flujo sea mas simple. De esta forma, se busca que cuando el cliente busque: http://localhost:8080/ejemplo en realidad ejecute: http://localhost:8080/ejemplo.php. Esto se haría con un htaccess sobre /var/www/html que ponga lo siguiente:

    `RewriteRule ^ejemplo$ ejemplo.php [L]`

  - expires: Gestiona el control de caché de los recursos estáticos. En nuestra configuración de apache, se usa aqui:

    ```text
    <FilesMatch "\.(jpg|jpeg|png|gif|css|js|ico|woff2?)$">
        ExpiresActive On
        ExpiresDefault "access plus 1 month"
    </FilesMatch>
    ```

    Este bloque de configuración lo explicamos más tarde

  - headers: Permite modificar cabeceras HTTP de las respuestas. Esto se puede hacer directamente desde php en un script sin utilizar el modulo headers, por ejemplo, está en la práctica 4 en el punto 8, en el que indicaremos la cabecera del content-type que daremos al cliente. Este mismo ejemplo en apache, sería así:

    ```conf
    <FilesMatch "^a.*\.php$">
        Header set Content-Type "application/json"
    </FilesMatch>
    ```

    Más tarde veremos otras configuraciones de cabeceras puestas como estas:

    ```
    Header always set X-Content-Type-Options "nosniff"
    Header always set Referrer-Policy "strict-origin-when-cross-origin"
    Header always set Content-Security-Policy "default-src 'self'; object-src 'none'; base-uri 'self'"
    ```

    FIN SOL

- **¿Qué módulo se utilizaría (no está) para capturar y filtrar número de peticiones? (rate limit)**. Si no sabes qué es el rate limit, deberás indicarlo aquí también

  SOL: mod_ratelimit, y permite limitar el ancho de banda asignado a cada cliente o tipo de recurso. Vemos un ejemplo:
  ```text
  <IfModule mod_ratelimit.c>
      <FilesMatch "\.(mp4|zip|iso)$">
          SetOutputFilter RATE_LIMIT
          SetEnv rate-limit 200
      </FilesMatch>
  </IfModule>
  ```

  SOL: Ojo, la sentencia <IfModule> es una condicional específica de Apache. Sirve para: “Ejecuta este bloque solo si el módulo indicado está cargado”. El ".c" de  "mod_ratelimit.c" no es más que notación para hacer referencia al módulo cargado en lenguaje c, pero se puede poner sólo "IfModule mod_ratelimit".

  

  La directiva "SetOutputFilter RATE_LIMIT" activa el filtro del módulo "mod_ratelimit", que controla la velocidad de envío de datos al cliente para evitar que una descarga o transmisión sature el ancho de banda. 

  Esto se hace con "SetEnv rate-limit 200 "que crea una variable de entorno llamada "rate-limit" con  valor de "200 KB/s" por conexión. Estas variables de entorno consultarse desde scripts en lenguaje php en la carpeta www/var/html (prácticas 3 y 4) mediante $_SERVER['rate-limit']; FIN SOL.

#### <u>- apache/000-default.conf: (explicamos cada uno de ellos)</u>

El objetivo de lo siguiente es hablar del archivo de configuración inicial de apache y de cada una de sus peculiaridades. Al igual que con la parte anterior de la práctica, os iré haciendo preguntas al respecto de lo que vaya saliendo, poco a poco.



![image-20251017151010293](/home/manolo/.config/Typora/typora-user-images/image-20251017151010293.png)

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

- Empieza con la etiqueta `\<VirtualHost *:80>` y termina con su cierre  `\</VirtualHost>`. Con eso Apache define un  "bloque de sitio” que escucha en el puerto 80 de *todas* las interfaces (`*`).  Dentro del contenedor, Apache escucha en el 80; fuera, tú lo ves en `http://localhost:8080` porque Docker hace el mapeo `8080:80`. Si algún día tienes más "sitios", puedes crear más bloques `VirtualHost` (incluso uno por dominio) y diferenciarlos con `ServerName`.

     **[PREGUNTA]: ¿Qué pasa si reutilizo la etiqueta VirtualHost? ¿Podría servir "dos sitios" con reglas distintas?.**

     SOL. Sí, sin problema pero deben tener distintos puertos, no hay más. En cada dominio, puedes tener configuraciones distintas, como es lógico. FIN SOL.

     

- `ServerAdmin webmaster@localhost` es solo un correo de contacto que Apache muestra en algunas páginas de error y mete en los logs. No afecta al funcionamiento; puedes poner tu email real o ignorarlo.

  

- `DocumentRoot /var/www/html` indica la carpeta pública desde la que Apache sirve archivos. En tu setup, esa ruta está montada desde tu máquina con `./www:/var/www/html`, así que lo que guardes en `./www` es lo que ve Apache y lo que se sirve al navegador (Esto es justo lo que vimos ayer en clase metiéndonos dentro del contenedor).  El bloque `\<Directory /var/www/html>` aplica reglas específicas a esa carpeta:

  

- `Options -Indexes +FollowSymLinks` controla funcionalidades dentro del directorio.

  - `-Indexes` desactiva el listado de directorios. 

    **[PREGUNTA]: Esta explicación puede que no quede muy clara, especifica qué significa con un ejemplo. Lanza tu servidor con esta opción y sin esta y comprueba por ti mismo (adjuntando captura de pantalla) que efectivamente sucede así.**

    Bastaba con investigar un poco. Si quito la opción, en el servidor me queda esto:
    ![image-20251101152215031](/home/manolo/.config/Typora/typora-user-images/image-20251101152215031.png)

    Ahora hay que reiniciar el servidor. Para ello, podéis hacer `systemctl restart apache` o si tenéis docker: `docker compose down` y `docker compose up --build`

    Una vez esto, creáis una carpeta en `var/www/html` yo le he puesto el nombre `otros`. En la carpeta, dejáis un script, por ejemplo `bucles.php`.

    Si en el navegador escribís: `http://locahost/otros` (por favor, poned el puerto que utilicéis si estáis con docker) os aparecerá algo así:

    ![image-20251101152658477](/home/manolo/.config/Typora/typora-user-images/image-20251101152658477.png)

    Esto no es NUNCA conveniente en un servidor apache, es por eso que se pone la opción -Indexes (para que no de info de lo que hay en el servidor). Poner +Indexes es lo mismo que quitar directamente todo el texto "- Indexes", tal y como está en la captura de arriba.

    Si volvéis a poner -Indexes en la configuración:

    ![image-20251101152922949](/home/manolo/.config/Typora/typora-user-images/image-20251101152922949.png)

    Entonces al visitar:  `http://locahost/otros` veréis esto:
    ![image-20251101153024875](/home/manolo/.config/Typora/typora-user-images/image-20251101153024875.png)

    que es justo lo que se espera de un servidor.

    

  - `+FollowSymLinks` permite a Apache seguir enlaces simbólicos. Útil si apuntas subcarpetas a otros lugares. Si quieres ser aún más estricto, existe `SymLinksIfOwnerMatch` (solo sigue el symlink si el propietario coincide), pero `FollowSymLinks` es lo habitual.

    **[PREGUNTA]: Pon ejemplos de qué significa esto definiendo distintos propietarios: Para guiarte un poco, piensa que la carpeta /var/www/privado con propietario manolo  se crea un enlace con /var/www/html/privado. Si alguien accede por red, ¿podría ver el contenido de /var/www/html/privado? Para ello investiga con que usuario se indenficaría dicha persona al entrar a la web e intentar la redirección.**

    SOL. No os lo voy a preguntar. Sólo valoraré que lo hayáis escrito correctamente y lo hayáis explicado y ejemplificado. FIN SOL.
    
    

- `AllowOverride` es otra directiva de Apache que en este caso controla qué directivas están permitidas dentro de un `.htaccess` (archivo que veremos más tarde). Le estás diciendo a Apache en este caso: “Dentro de esta carpeta, haz caso a los archivos `.htaccess` que haya”. Significa que Apache permitirá archivos `.htaccess` en esa carpeta (en este caso:`/var/www/html`) y en cualquiera de sus subcarpetas (ojo, lo de cualquiera de sus subcarpetas es importante y más tarde veremos porqué, ya que afecta al funcionamiento de un servidor en producción y provoca que, habitualmente, no se cree el archivo .htacess. 

     En conclusión: es muy común dejarlo en "All" durante desarrollo y como hemos dicho, en producción, por rendimiento, muchos mueven esas reglas al propio VirtualHost y ponen `AllowOverride None`. Lo veremos más tarde.

     

- `Require all granted` es la regla de acceso (desde Apache 2.4). Significa “permite acceso a cualquiera”. Si no la pones (o niegas el acceso), podrías ver errores 403 (los códigos de errores los he comentado ya mucho en clase pero los veremos cuando generemos nuestros propios endpoints). Ejemplo: solo permitir a la IP `192.168.1.100` sería de la forma: `Require ip 192.168.1.100. `

- `/etc/apache2/envvars` lugar físico donde se encuentran las ENV, podéis echarle un vistazo, es interesante. Lo vimos en clase, simplemente, os metéis al contenedor y le hacéis un cat al fichero.

  **[PREGUNTA]: ¿Cómo se podría negar el acceso a una sola IP a nuestro servidor Apache?**

  ```text
  <Directory /var/www/html>
      Require all granted
      Require not ip 192.168.1.105
  </Directory>
  ```

  SOL. Esto lo hice en clase, desplegando el servidor con un balanceador de carga entre mi raspberry pi5 y mi portatil que estaban ambas bajo una misma interfaz de red y vimos cómo se negaba el acceso a mi raspberry pi5 sin ningún tipo de problema. FIN SOL.

  

- `ErrorLog ${APACHE_LOG_DIR}/app-error.log` 

  **[PREGUNTA]: ¿Qué tipo de errores guarda el log app-error.log?**

  SOL: Registra errores internos del servidor Apache como: Fallos espécificos de sintaxis en configuraciones y problemas de permisos a la hora de acceder a recursos. En clase vimos este ejemplo:

  > [!IMPORTANT]
  >
  > Dentro del contenedor, esos archivos viven en `/var/log/apache2/`. Para verlos puedes hacer `docker exec -it apache-php bash` (esto entra en una terminal del contenedor donde está apache instalado) y luego `tail -f /var/log/apache2/app-error.log`. (sirve para ver el log de errores). En este caso se observana el error de la ip de mi raspberry intentando acceder a un recurso no válido.

  

  **[PREGUNTA]: ¿Qué significa ${APACHE_LOG_DIR} y donde se guardan dichas variables?**

  SOL: ${APACHE_LOG_DIR} es una variable de entorno definida en `/etc/apache2/envvars`. De hecho, más arriba se comenta que este fichero posee las variables de entorno de apache. FIN SOL.

  
  
- `CustomLog ${APACHE_LOG_DIR}/app-access.log combined` 

  **[PREGUNTA]: Durante las prácticas anteriores, has hecho peticiones http a un servidor. ¿Qué es lo que se vería en este caso en app-access.log y que relacion tiene con la práctica anterior? **

  SOL: En este log, se registran todas las peticiones HTTP que recibe Apache: método, URL, código de respuesta, tamaño...

  En clase, no llegamos a acceder a este log, pero se haría de igual forma que con el otro log al que yo sí os enseñe cómo se accedía:

  > [!IMPORTANT]
  >
  > Dentro del contenedor, esos archivos viven en `/var/log/apache2/`. Para verlos puedes hacer `docker exec -it apache-php bash` (esto entra en una terminal del contenedor donde está apache instalado) y luego `tail -f /var/log/apache2/app-access.log`. (sirve para ver el log de peticiones).

  El combined no es más que un formato que define la forma y el contenido de lo que se añade en el log.

  FIN SOL

  

- `<Directory /var/www/html/admin>` Tenemos una regla de autenticación sobre este directorio, aunque si os fijáis y tal y como pongo en el comentario encima de la regla, el directorio no existe, aunque no afecta a Apache.

  Lo que has visto en ese bloque de configuración es un ejemplo de autenticación HTTP básica en Apache. Sirve para proteger con usuario y contraseña un directorio concreto, en este caso `/var/www/html/admin`.

  - La directiva `AuthType Basic` indica que se usará el método de autenticación más sencillo, llamado *Basic*. Este funciona enviando el usuario y la contraseña codificados en base64 (debes saber qué es ya que muchas veces los servidores envía binarios en base64) en cada petición. Por eso, en la práctica, siempre se recomienda usarlo junto con HTTPS para que las credenciales viajen cifradas. 

    **[PREGUNTA]  Auth Type sirve para proteger recursos. El estandar a día de hoy ha cambiado a JWT. Busca información al respecto sobre qué es. ¿Todo el mundo debe tener derecho a acceder a cualquier endpont?**

    JWT es un estandar para proteger recursos en programación web, permite verificar la identidad del usuario creando un webtoken cada vez que el usuario inicia sesión en la página. Este webtoken lo usaremos para (de alguna forma), proteger algunas zonas de nuestra aplicación que no queremos que todo el mundo tenga acceso, por ejemplo, un script.php que solo debe ejecutar un admin, o un script.php que te mande los mensajes que te han mandado otros usuarios con los que has establecido una relación de amistad dentro de la pagina web. Evidentemente, si no estás registrado no debes poder ejecutar ese script.

  - `AuthName "Zona restringida"` define el texto que verá el usuario cuando el navegador muestre la ventana emergente pidiéndole credenciales. No afecta a la seguridad, es simplemente informativo, pero ayuda a diferenciar distintas áreas protegidas de un mismo servidor.

  - La línea `AuthUserFile /etc/apache2/.htpasswd` indica dónde está guardado el archivo que contiene los usuarios y contraseñas autorizados. E nuestro proyecto no lo tenemos definido porque, de hecho la carpeta /admin no existe. Ese archivo `.htpasswd` NO debe estar dentro de la carpeta pública del servidor web, porque si no, cualquiera podría descargarlo. En él se almacenan las credenciales en formato cifrado (normalmente con algoritmos como MD5, SHA o bcrypt).

  - Finalmente, `Require valid-user` define quién puede acceder. En este caso, “valid-user” significa que cualquiera de los usuarios definidos en el archivo `.htpasswd` podrá entrar. Pero hay otras variantes: se puede especificar `Require user juan` para permitir solo a un usuario concreto, o `Require user juan pedro` para permitir únicamente a esos dos. También existe la posibilidad de trabajar con grupos de usuarios, usando `Require group admins`, donde los grupos se definen en un archivo adicional llamado `.htgroup`.

    

-  Vamos con el siguiente cacho de código, que es muy interesante y muy práctico:

      ```text
      <FilesMatch "\.(jpg|jpeg|png|gif|css|js|ico|woff2?)$">
          ExpiresActive On
          ExpiresDefault "access plus 1 month"
  </FilesMatch>
  ```

  Esto significa: “Aplica las siguientes reglas a todos los archivos cuyo nombre termine en `.jpg`, `.jpeg`, `.png`, `.gif`, `.css`, `.js`, `.ico` o `.woff/woff2` (fuentes web)”. Es decir, cualquier recurso estático habitual de una web (con estático nos referimos a archivos que sualmente no van a cambiar y que se van a mandar tal cual). Dentro de ese bloque, `ExpiresActive On` activa el módulo de caducidad de caché. Y `ExpiresDefault "access plus 1 month"` establece que el navegador puede mantener esos archivos en caché durante un mes a partir del momento en que los descarga. Ojo, el navegador del cliente sería el que cachearía dicha información, evitando así incluso que la petición llegue al servidor. La cuestión aquí es saber cómo se le dice al navegador que cachee dicha petición, eso lo veremos ahora y avanzo que será gracias a las cabeceras http de la respuesta del servidor!

  Con todo esto, si un usuario entra varias veces a tu web, su navegador no volverá a pedir al servidor la imagen del logo, los estilos CSS o el JavaScript en cada carga, sino que reutilizará los que ya descargó hasta que pase un mes. Como es lógico, el resultado es que la web se siente mucho más rápida y consumes menos ancho de banda en el servidor. Lo interesante es que puedes personalizar estas reglas para diferentes tipos de archivo. Por ejemplo:

  - Para **imágenes y vídeos grandes**, podrías poner más tiempo (Esto no viene en nuestro archivo de configuración): Fijate que la directiva es ExpiresByType 

  ```text
  ExpiresByType image/jpeg "access plus 1 year"
  ExpiresByType image/png "access plus 1 year"
  ExpiresByType video/mp4 "access plus 1 year"
  
  ```
  
  - Para **archivos CSS y JS**, quizá prefieras menos tiempo, como una semana, porque podrías actualizarlos con frecuencia durante el desarrollo (Esto no viene en nuestro archivo de configuración):

  ```text
  Para archivos CSS y JS, quizá prefieras menos tiempo, como una semana, porque podrías actualizarlos con frecuencia durante el desarrollo
  ```
  
  - Para **documentos sensibles o que cambian mucho** (como  un endpoint PHP), lo normal es directamente desactivar la caché (Esto no viene en nuestro archivo de configuración): 

  ```text
  <FilesMatch "\.(php)$">
      ExpiresActive Off
      Header set Cache-Control "no-store, no-cache, must-revalidate"
  </FilesMatch>
  
  ```
  
  Al final lo que hace todo esto visto en los ejemplos anteriores es insertar una cabecera http en la respuesta, algo como esto:

  ```makefile
  HTTP/1.1 200 OK
  Content-Type: image/png
  Content-Length: 10542
  Expires: Wed, 09 Nov 2025 10:49:27 GMT
  Cache-Control: max-age=2592000
  ```
  
  El navegador que recibe la respuesta será capaz de cachearla según las opciones propuestas.
  
  **[PREGUNTA]** **¿Cual es la diferencia entre Expires y Cache-Control? Busca las distintas opciones de Cache-Control y qué singifican.**
  
  SOL: **Expires:** define la fecha en la que el recurso dejará de ser cacheado. Se utilizaba en HTTP 1.0 y suele estar en desuso en navegadores actuales. Se sigue poniendo autmáticamente para compatibilidad.
  
  **Cache-Control:** es la versión actual del expires, con muchas más opciones y más versatil, dejo las opciones más comunes que trabajremos. Aquí valoraré que lo que hayáis puesto, sea correcto.
  
  - `no-store`: no lo cachees bajo ninguna circunstancia
  
  - `max-age=3600`: válido en cache duante una hora (tiempo en segundos)
  
  - `must-revalidate`: Puedes usar el archivo desde la caché hasta que expire, pero cuando expire, debes verificar con el servidor si sigue siendo válido.
  
  - `no-cache`: Puedes guardar el archivo, pero no lo uses nunca directamente sin antes preguntarle al servidor si sigue siendo válido. veremos como hacer esto, cuando toque.
  
    FIN SOL.
  
  
  
  > [!IMPORTANT]
  > Dentro del contenedor, esos archivos viven en `/var/log/apache2/`. Para verlos puedes hacer `docker exec -it apache-php bash` y luego `tail -f /var/log/apache2/app-error.log`. Si prefieres tenerlos en tu host, puedes montar un volumen para `logs/` y así inspeccionarlos con tu editor. Otra opción avanzada es enviar logs a stdout/stderr para leerlos con `docker logs`, pero con ficheros sueles tener más control y lo recomiendo sobremanera. Los comandos mencionados a ejecutar dentro del contenedor son los siguientes:
  >
  > ```bash
  > docker exec -it apache-php bash
  > cd /var/log/apache2
  > tail -f app-access.log
  > 
  > ```
  >
  > **[PREGUNTA]: Ya contesta que era lo que se guardaba en app-access.log, una vez visto esto, y realizado al menos una petición al servidor, metete al contenedor y comprueba qué es lo que se guarda mediante una captura de pantalla**
  >
  > ![image-20251031132301475](/home/manolo/.config/Typora/typora-user-images/image-20251031132301475.png)

Con una captura tal que así, me bastaba. FIN SOL

- Sigamos viendo otras partes, como las cabeceras http, que ya hemos visto que pueden ser muy importantes en la interaccion del cliente con el servidor. Ya sabemos que cuando un navegador (cliente) habla con un servidor web, lo hace usando el protocolo HTTP. Cada petición y cada respuesta viaja en forma de texto, con varias líneas de información llamadas cabeceras (headers). 

  Estas cabeceras son como *metadatos*: pequeños mensajes que acompañan a la petición o a la respuesta para dar más contexto sobre qué se envía, cómo debe interpretarse y cómo debe comportarse el navegador o el servidor. En la práctica anterior nos centramos muchísimo en la cabecera "Content-Type", pero lo normal es que haya más cabeceras que se envíen por defecto, por ejemplo, lo siguiente muestras las cabeceras http de una petición (REQUEST):
  
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

  - `X-Content-Type-Options "nosniff"` 

    **[PREGUNTA]: ¿Para qué sirve y qué tiene que ver con los Content-Type estudiados hasta ahora?**

    SOL: Evita que el navegador “infiera” el tipo de contenido si el servidor no se indica (en el apartado 8 de la práctica 4, veremos como el servidor desde un script puede indicar el Content-Type) Con esta opción, el navegador solo acepta el tipo declarado en Content-Type. Ojo, esto no se suele poner, y no lo pondremos por defecto. En la práctica 1 ya estudiamos los Content-Types y en este caso no es el Content-Type que se envía del cliente al servidor, si no al revés. FIN SOL.

  - Content-Security-Policy: default-src 'self'; object-src 'none'; base-uri 'self'
  
    **[PREGUNTA]: Aquí os ayudo un poco más ya que pienso que es más complejo: **
  
    **La cabecera `Content-Security-Policy` (CSP) permite al servidor indicar al navegador desde qué orígenes puede cargar recursos como scripts, imágenes o estilos. De esta forma, si en el HTML hay etiquetas que intentan cargar contenido externo no permitido, el navegador lo bloqueará automáticamente. ** **Haciendo uso de la explicación anterior, detalla cómo funciona esta protección con algunos ejemplos además, responde a lo siguiente: qué significa que `default-src 'self'` solo deje cargar recursos del mismo dominio, qué consigue `object-src 'none'` al prohibir objetos embebidos, y cómo `base-uri 'self'` limita el uso de la etiqueta `<base>` en el documento. Muestra ejemplos de código HTML que serían permitidos y otros que serían bloqueados.**
  
    SOL:
  
    `default-src 'self'` :Indica al navegador los orígenes permitidos para cargar recursos (scripts, hojas de estilo, imágenes fuentes...) en una web. Si en defacult-src ponemos 'self' indica que sólo se pueden coger recursos del mismo servidor.  Por ejemplo, es normal que en el frontend de la aplicación carguemos librería de iconos para hacerla más bonita, pero con esta política el navegador bloqueará esos recursos externos y no se verán los iconos. Por ejemplo, en el html, en la etiqueta head cargamos los iconos así, a partir del recursos web de iconos de un cdn:
  
    ```html
    <!DOCTYPE html>
    <html lang="es">
    <head>
      <meta charset="UTF-8">
      <title>Ejemplo bloqueado</title>
    
      <!-- Librería externa de iconos AQUI -->
      <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.2/css/all.min.css">
        
    </head>
    <body>
      <h1><i class="fa-solid fa-user"></i> Perfil de usuario</h1>
    </body>
    </html>
    
    ```
  
    Esto no se podría debido a la directiva actual: default-src 'self'. Deberíamos descargar los iconos, ponerlos en nuestro servidor a mano y hacer algo así:
  
    ```html
    <!DOCTYPE html>
    <html lang="es">
    <head>
      <meta charset="UTF-8">
      <title>Ejemplo permitido</title>
      <link rel="stylesheet" href="/css/fontawesome.min.css"> //esto está descargado en local y esta en nuestro server
      <link rel="stylesheet" href="/css/solid.min.css">
    
    </head>
    <body>
      <h1><i class="fa-solid fa-user"></i> Perfil de usuario</h1>
    </body>
    </html>
    
    ```
  
    
  
    `object-src 'none'`: Controla qué orígenes pueden usarse para cargar contenido embebido mediante etiquetas como por ejemplo, las siguiente:
  
    - `<object>`: sirve para inscrustar contenido binario, como un pdf en el html
  
    - `<embed>`: como object, pero obsoleto
  
    En nuestro caso, <u>al poner 'none'</u> prohibimos cualquier contenido embebido, de cualquier origen, incluso PROPIO.
  
    
  
    `base-uri 'self'`: La etiqueta <base> define la URL base para todas las rutas relativas de la página en el head del html.
  
    ```html
    <html lang="es">
    <head>
      <meta charset="UTF-8">
      <title>Ejemplo permitido</title>
      <base href="https://midominio.com/">
    ```
  
    Con `base-uri 'self'`, solo se permiten <base> que apunten al mismo dominio.
  
    
  
  - `Referrer-Policy "strict-origin-when-cross-origin"`
  
    **[PREGUNTA]: ¿Qué es referer en la cabecera y que hace este tipo de política?**
    
    Cada vez que un navegador (cliente) solicita un recurso al servidor, en la cabecera http se envía el siguiente contenido:
    ```text
    Referer: https://iaw.com/perfil?usuario=manolo
    ```
    
    En nuestro caso, tenemos la opcion: "Referrer-Policy: strict-origin-when-cross-origin" que hace lo siguiente:
    
    Si la solicitud es dentro del mismo dominio de tu web, se envía el referer completo como en el ejemplo anterior. Si la solicitud es a otro lado externo a mi dominio, se envía solo "https://iaw.com/"
    
    FIN SOL.
    
    

#### <u>- Archivo `.htaccess` (uso de reglas RewriteCond y RewriteRule)</u>

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

```text
RewriteEngine On

# Si la URL no apunta a un archivo real...
RewriteCond %{REQUEST_FILENAME} !-f
# ... y no apunta a un directorio real
RewriteCond %{REQUEST_FILENAME} !-d
# Redirigimos todo a index.php
RewriteRule ^ index.php [L]

```

Vamos a estudiar las condiciones de este .htaccess, que como es dicho, podrían estar directamente sobre el archivo de configuración de Apache y no necesariamente en el .htaccess

#####  - La primera condición

```
RewriteCond %{REQUEST_FILENAME} !-f
```

`%{REQUEST_FILENAME}`  es la ruta física del archivo que corresponde a la URL pedida.`! -f`  significa "NO es un archivo normal existente".

solo se aplicará la regla si el archivo real no existe

Ejemplo:

- URL: `/style.css` , sí existe `/var/www/html/style.css` : la condición NO se cumple.

- URL: `/api/usuarios` , no existe `/var/www/html/api/usuarios` como archivo : la condición se cumple.

  

##### - La segunda condición

    RewriteCond %{REQUEST_FILENAME} !-d

Solo se aplicará la regla si la ruta pedida no es un directorio real.

Ejemplos:

- Se ejecuta la URL: `/uploads/` , sí existe `/var/www/html/uploads/` como carpeta: la condición NO se cumple.
- se ejecuta la URL: `/api/usuarios` , no existe `/var/www/html/api/usuarios` como carpeta : la condición se cumple.

¿Por qué se tienen que cumplir las dos? Porque si existe un archivo o un directorio real, nuestra configuración decide que Apache debe servirlo directamente. Este tipo de configuración suele ser común y estándar y tenéis que saber cómo funciona. En conclusión, no queremos que una petición de `style.css` acabe en `index.php` (en caso de que sí exista style.css) ni que la carpeta `uploads/` sea redirigida. Solo cuando:

- No existe dicho archivo,
- No existe dicha carpeta,
   entonces aplicamos la regla final:

Hemos visto dos ejemplos de `RewriteCond` pero, en general, las RewriteCond siguen la siguiente estructura. Estudiemos su estructura y veamos más ejemplos.

##### - Estudio de las Condiciones (`RewriteCond`) en general, con más ejemplos

Sintaxis general:

```
RewriteCond TESTSTRING CONDITION

```

- **TESTSTRING** : es lo que se va a evaluar (ej: `%{REQUEST_FILENAME}`, `%{HTTP_HOST}`, `%{QUERY_STRING}`, etc.).

   **[PREGUNTA] ¿QUE OTRAS VARIABLES DE ENTORNO SE GENERAN EN APACHE CUANDO SE RECIBE UNA PETICIÓN HTTP?**

   SOL: `REQUEST_METHOD` ,`REQUEST_URI` , `REMOTE_ADDR`,`HTTP_USER_AGENT` ,`HTTP_HOST` ,`QUERY_STRING`
   FIN SOL.

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

##### -Tercera condición

```
RewriteRule ^ index.php [L]
```

- **`^` en el patrón**: Como ya sabéis, en expresiones regulares, `^` significa “inicio de la cadena”. Aquí el patrón es `^`, lo que equivale a decir “cualquier cosa que empiece en el principio”, o sea, todas las URLs. Normalmente se combina con las condiciones de`RewriteCond` ya explicadas para filtrar casos, como ya haces con `!-f` y `!-d`.

- **`index.php` en el destino** Significa: “reescribe la URL para que Apache sirva `index.php`”. Esto no cambia lo que el usuario ve en el navegador, simplemente Apache decide que la petición `/productos/42` en realidad la atienda el archivo `index.php` y no por un archivo o carpeta que estuviera en `var/www/html`.

- **`[L]` (flag “Last”)** Indica que esta debe ser la última regla evaluada si se cumple. Una vez que Apache aplica esta regla, deja de mirar otras reglas de reescritura en ese contexto. Es como un `break` en un bucle de programación.

Al igual que con las `RewriteCond` vemos la sintáxsis general de las `RewriteRule`

##### - Estudio de las `RewriteRule` en general

```
RewriteRule patrón destino [flags]
```

- **patrón** : es una expresión regular (ver Anexo) que Apache aplica sobre la URL solicitada (normalmente sobre el `REQUEST_URI` sin el dominio inicial).

  **[PREGUNTA] ¿Que es request_URI?**

  SOL: `http://iaw/es/aburrida` : `REQUEST_URI = /es/aburrida` Es decir, parte de la url después del dominio. FIN SOL.

- **destino** : adónde se debe redirigir o reescribir la petición. Puede ser un archivo local (ej: `index.php`) o incluso una URL completa (`https://otro.com/`).

- **[flags]** : opciones que modifican el comportamiento (como `L`, `R`, etc.).

**[PREGUNTA]: Busca información sobre otro flag que se pudiera añadir e indica qué significaría**.

SOL. No os lo voy a preguntar. Sólo valoraré que lo hayáis escrito correctamente y lo hayáis explicado y ejemplificado. FIN SOL.

#### <u>- index.php con conexión a MySQL y dos endpoints simples</u>

Ahora un ejemplo un poco más útil, que prueba la conexión a la base de datos con las credenciales que definiste en las variables de entorno de `docker-compose.yml` (No tienen porqué ser las mismas que yo puse).

**[PREGUNTA] Esta pregunta es sólo para ver si estáis siguiendo la lectura ya que lo que pregunto a continuación ya ha sido explicado **

- **¿En qué casos php y bajo qué peticiones del cliente ejecutará index.php APACHE?**

  SOL: Cuando la URL solicitada no corresponde a un archivo o carpeta real. FIN SOL.

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

**[PREGUNTA]: **

SOL (valoraré hasta donde hayáis ejemplificado):

- **¿Cuáles son esos dos endpoints? **

  Son dos recuros, index.php y productos.php, pero en total, definen 3 endpoints.
  /api/usuarios; /api/ping; /productos.php

- **¿Cuál es el flujo que hace Apache desde la petición del cliente hasta que devuelve la respuesta del endpoint? (Mirar .htaccess)**

  Despues de pasar las reglas de VirtualHost, pasa al .htaccess, donde el módulo `mod_rewrite` comprueba si el recurso solicitado existe físicamente en el servidor. Si no existe, Apache redirige internamente la petición hacia `index.php`

- **¿Tienen verbos http definidos estos endpoints? ¿Responderían ante cualquier tipo de verbo http? **

  En `/api/usuarios` y `/api/ping`, el código no comprueba el método así que en la práctica da igual  `GET`, `POST`, `PUT`.

  En`/productos.php` se Exige POST.

- **¿Qué tipo de "Content-Type" envían ambos al cliente? **

  header("Content-Type: application/json");

- **¿Si tuvieras que darle un verbo http, que verbo le darías a cada endpoint anterior? Justifica tu respuesta**

  GET, GET y POST, respectivamente según el orden `/api/usuarios`, `/api/ping` y `/productos.php`:

FIN SOL:



#### <u>**productos.php**</u>

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



**[PREGUNTA]: Tal y como esta configurado apache, el archivo php anterior representa un endpoint por si mismo. **

- **¿Cómo se hace la petición a ese archivo? ¿Qué tipo de verbo http sería?** 
  http://localhost/productos.php en el navegador es un error, ya que no puedo hacer un post y hará por defecto un GET. la petición se hace con un fetch de javaScript, con Postman, con un curl o con un formulario HTML, eligiendo POST y en Content-Type correcto.

- **¿Qué tipo de formato de datos accepta la petición?** **¿Qué tipo de formato devuelve?**

  POST. El cliente envía JSON y recibe JSON. Justificalo un poco más pero esa es la respuesta.

- **Prueba a a hacer una petición con POSTMAN y otra con CURL y adjunta capturas de la respuesta.**

  Os habéis hartado de hacer esto en la práctica 1. No tiene mucho misterio.

  ![image-20251101153829665](/home/manolo/.config/Typora/typora-user-images/image-20251101153829665.png)

  ​	Con curl sería así:
  
  ```bash
  curl -X POST http://localhost/productos.php -H "Content-Type: application/json" -d '{"nombre": "Camiseta"}'
  
  ```
  
  

**[PREGUNTA] En esta práctica habéis configurado manualmente vuestro servidor Apache con `000-default.conf` y un `.htaccess` para redirigir (algunas) lURLs a `index.php`. Todo el control de rutas, autenticación y caché lo habéis hecho directamente en el servidor web.**

- **¿Qué cambia cuando usamos un framework como Laravel?**

  Tendremos un script universal, el cual se suele llamar index.php que levantará el servidor recibirá las peticiones, hará una conexión persistente a la base de datos y definirá endpoints (sin necesidad de que sean recursos del servidor), muy parecido a lo que ahcemos en esta práctica con index.php.

- **¿Dónde se definen las rutas en ese caso: en el servidor Apache o dentro del framework?**

  Las rutas o endpoints ya no son un script del servidor que se pide y se ejecuta, es decir, no son archivos php si no que son sentencias lógicas dentro de otros scripts. En Laravel se definen en archivos como:

  - `routes/api.php` 

  Por ejemplo, se puede definir la ruta `http://locahost/productos` escribiendo esta sentencia en el script de Laravel:

  ```php
  Route::get('/productos', [ProductoController::class, 'index']);
  ```

  Donde definimos la ruta `/productos`, que será de tipo `GET` y lo que hará estará o proporcionará este endpoint al cliente estará definido en el método `index` de la clase `ProductoController`

- **¿Qué ventajas y desventajas tiene cada enfoque (controlarlo en Apache vs. dejarlo al framework)?**

  Cuando Apache gestiona las rutas mediante archivos como `.htaccess` o `000-default.conf`, es el servidor quien decide qué archivo PHP responderá según la URL, pero a medida que el proyecto crece, mantener las rutas y configuraciones en el servidor se vuelve muy dificil ya que tienes que manejar al mismo tiempo la configuración de Apache y la lógica de PHP. Ahora bien, cuando usas un framework como por ejemplo: Laravel (uno de los más usados para PHP), los endpoints se definen directamente en el código. Esto permite alejarnos un poco de la configuración del servidor y aprovechar herramientas del framework como (controladores, middleware, validación, autenticación, caché o políticas de acceso...), que podrán tener muchas más versatilidad que muchas de las opciones que nos ofrece apache y en esencia, harán lo mismo.

- **¿Por qué creéis que en proyectos grandes se recomienda que Apache solo apunte a `public/` y que el framework gestione las rutas y la lógica?**

  Esta es la clave. No configuraremos nada en Apache, solamente, apuntará a public donde estará el script universal que hablamos en la pregunta 1, el famoso: `index.php` . De esta forma, delegamos todas las configuraciones en el framework para dejar las configuraciones de Apache de lado.

**[PREGUNTA] A lo largo de la práctica has visto que se ha utilizado la extensión mysqli para hacer querys. **

- **¿Qué desventaja tiene esto en un servidor Web real?**

  Son muchas las cosas que podéis poner aquí. Hago una lista de las que considero más importantes.

  -EL código es confuso. Cada vez que el cliente hace una petición hay que conectarse a la BDD lo que es repetitivo y a la hora de escalar en aplicaciones más grandes, complejo.

  -La conexión a la BDD no es persistente, cada vez que se levanta el servidor y el cliente envía una petición a Apache, se abre una conexión y al terminar la respuesta del servidor, se cierra.

  -El lenguaje SQL se escribe directamente (raw). Si no eres experto en bases de datos, es fácil cometer errores: no solo puedes escribir consultas vulnerables a inyecciones SQL, sino que muchas pueden no estar muy optimizadas y por tanto consumir demasiados recursos del servidor.

- **¿Qué es un ORM y que relación tiene con lo anterior?**

  Un ORM (Object-Relational Mapping) permite trabajar con la base de datos usando objetos prefedinifidos por una librería en específico (por ejemplo, Mongoose para MongoDB) y clases, en lugar de escribir directamente SQL. Esto ahce que el código sea mucho más seguro ya que está preparado para afrontar vulnerabilidades que un usuario no experimentado no llegue a comprender.

- **¿Qué ventajas ofrecen los ORM que nos proporcionan algunos frameworks como Larabel?**

  En mi opinión personal, la ventaja principal es la siguiente (aunque si me ponéis cualquier otra cosa que tenga sentido, la aceptaré): la base de datos se gestiona de forma más eficiente con los ORM que nos proporciona Larabel, porque se reutiliza la conexión a la BDD en lugar de abrirla y cerrarla en cada consulta (esto es justo lo que pasa con mysqli). Gracias a esto la misma conexión se mantiene activa durante la petición y puede volver a usarse sin que se cierre al terminar la ejecución de código que el server hace al cliente. Esto reduce el consumo de recursos y evita posibles errores.

### Anexo (Cosas básis de regexp con algunos ejemplos)

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







