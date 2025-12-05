# Práctica: *Recuperar el Muro María*

Eres un recluta recién graduado del 104º Cuerpo de Entrenamiento. Tu misión es demostrar tus habilidades técnicas (en Postman o con `curl`) para unirte a una legión y luchar contra los titanes. Cada endpoint de esta API representa un reto. Sigue el orden y usa el formato correcto, o fallarás en la misión. Para poder ejecutar esta práctica necesitas instalar:

1. Docker
2. Postman
3. Tener el paquete curl instalado en tu distribución debian (por defecto en Ubuntu)

![image-20251204105728932](./img/image-20251204105728932.png)



Cuando se establece una conexión HTTP entre servidor y cliente se envían cabeceras (request:petición; response:respuesta)

![image-20251204110206924](./img/image-20251204110206924.png)

Estas cabeceras tienen información fundamental, como el tipo de verbo HTTP y el Content-Type (contenido que se envía):

Verbos HTTP: Get, Post, Put, Delete.

Asociación de los verbos HTTP con cada "Content-Type" de sus cabeceras, en la siguiente imagen:

-Los verbos **PUT** (editar info de una BD) ,**POST** (enviar info a una BD) y **DELETE** (Borrar info de una BD) pueden llevar la cabecera **body** en el request. Dentro de este body se envía el contenido de la petición, que podrá ser formateada de las siguiente formas:

- **application/json**: tipo de dato más usado en internet a día de hoy debido a las API's REST.
- **multipart/form-data**: por defecto lo que envían los formularios HTML con archivos asociados.
- application/x-www-form-urlenconded: por defecto es lo que envían los formularios HTML.
- **text/plain**. No tan usado. Lo veremos.

-El verbo **GET** (recoger info de una BD) no tiene **body**, ya que su principal propósito no es enviar datos, sino recoger datos. Ahora bien, peude ser que queramos que los datos que nos envíe el servidor sean filtrados.  Al no tener body, la única forma de enviar datos (para filtrar la respuesta del servidor) es a través de la URL a la que se hace la petición. Para enviar esta info se podrá mediante:

- **Query params**
- **Route Params**

La siguiente imagen resume lo anterior

![image-20251204105840786](./img/image-20251204105840786.png)

## Parte 1 de la práctica (POSTMAN)

Esta práctica se basa en enviar peticiones HTTP a un servidor ya montado. El servidor sabe qué debe responder en cada momento pero vosotros debéis indicar correctamente el tipo de verbo que envía así como el tipo de dato. Lo haréis indicando el verbo y el Content-type (o incluso contenidos que no van por body si no por URL`s).

### 1. Registrar un recluta

Verbo:**?¿ **ruta**:`/register`** 

Envia los datos en el body y en JSON. El siguiente ejemplo muestra los datos que debes enviar para que un recluta se pueda alistar.

```
{
  "username": "eren",
  "password": "1234",
  "legion": "Exploradores",
  "mentor": "Erwin",
  "edad": 15
}

```

Respuesta esperada:

```
{
  "mensaje": "Recluta eren registrado en la legión Exploradores, bajo la inspiración de Erwin"
}
```

Pregunta: ¿Qué verbo http debes usar body?

Pregunta: ¿Qué debes enviar "claves" en el body?

Pregunta: ¿Qué pasa si eliges `"legion": "Caballería"` o `"mentor": "Levi"`?

Pregunta: ¿Qué pasa si el mentor no es ["Eren", "Erwin", "Armin", "Mikasa", "Reiner"]?

Pregunta: ¿Qué pasa si la legión no es ["Exploradores", "Guarnición", "Policía Militar"]?

Pregunta: ¿Qué error devuelve si la edad es menor de 14?

Pregunta: ¿Qué pasa si el recluta ya existe?

Imprime todas las respuestas!

### 2. Ver info de un recluta

Verbo:**POST **ruta**:`/login`** 
Prueba distintos formatos de datos para enviar la información de login y así ver la información confidencial de dicho recluta. Deberás enviar usuario y contraseña.

Respuesta esperada sobre del endpoint en caso de usar contraseña correcta:

```
{
  "mensaje": "Acceso concedido",
  "recluta": {
    "id": "...",
    "username": "eren",
    "legion": "Exploradores",
    "mentor": "Erwin",
    "edad": 15
  }
}
```

¿Qué mensajes devuelve la API en los siguientes casos?

0. ¿Con qué formato de datos enviaste el Post?

1. Si intentas hacer login con un usuario que **no existe** en la base de datos.
2. Si el usuario existe pero introduces una **contraseña incorrecta una vez**.
3. Si introduces una contraseña incorrecta **dos veces seguidas**.
4. ¿Qué ocurre en el **tercer intento fallido** de contraseña?
5. Intenta iniciar sesión con un usuario al que han echado del cuerpo



### 3. Obtener misión aleatoria

Verbo:**¿? **ruta**:`/mission`** 

0. ¿Qué verbo debes usar?

1. ¿Qué otras misiones puede asignarte la API?

2.  ¿Cuántas son en total?
3. Devuelve alguna  salida del endpoint

### 4. Preguntar al Comandante Erwin

**GET** `/pregunta/erwin?task=¿Que debo hacer?`

Utiliza este Get para preguntar mediante la "variable" task a Erwing una pregunta.

1. ¿Qué te responde Erwing? 
2. ¿Cómo se llama este envío de datos mediante un Get? 

3. ¿Es bueno para enviar una contraseña?
4. Devuelve la salida del endpoint

### 5. Hablar a otro personaje

**GET** `/pregunta/:personaje`

Utiliza este GET para hablar a otro personaje algo que tu quieras, en este caso responde a las siguientes preguntas:

1. ¿Puedes hablar con cualquier personaje?
2. ¿Eliges la pregunta que hace o solo a quien hablas?
3. ¿Cómo se llama este envío de datos para solicitar un recurso? 
4. Devuelve la salida del endpoint

### 6. Unirte a una Tropa

**POST** `/enlist`

Utiliza este POST para ir de expedición junto con un número de tropa. En el body debes enviar el número de tropa.

1. ¿En qué formato debes enviarlo?
2. ¿Qué rango de números puedes enviar?
3. Devuelve la salida del endpoint

### 7. Analizar Titan

**POST** `/titan`

Utiliza este POST para informar de los datos de un titán. En el body puedes incluir lo que quieras mientras que sea texto

1. ¿En qué formato puedes enviar la información?
2. Devuelva la salida del endpoint

### 8. Cumplir años

Verbo:**¿?** Endpoint:**`/recruit/:id/age`**

Utiliza este endpoint para hacer que un recluta que quieras cumpla un año

0. Establece una estrategia para encontrar el ID del recluta

1. ¿Qué verbo debe usar este endpoint?
2. Devuelva la salida del endpoint
3. ¿Existe body en este caso?

### 9. Retirar a un recluta del frente

Verbo:**¿?** Endpoint:**`/retire/:id`**

El objetivo de este endpoint es que después de que ejecutarlo desaparezca por completo la info del recluta y no se pueda volver a ver su info. Para ello, deberás enviar en el body la contraseña del recluta sola en un JSON.

0. Establece una estrategia para encontrar el ID del recluta
1. ¿Qué verbo debe usar este endpoint?
2. Devuelve la salida del endpoint
3. Prueba que, efectivamente después de borrar al recluta ya no vuelve a aparecer (intentando hacer login, por ejemplo).

## 10. Enviar mapa de datos (png)

Verbo:**¿?** Endpoint:**`/map`**

El objetivo de este endpoint es enviar un png con la información a armin sobre donde se encuentran los titanes más cercanos

1- ¿Qué verbo http debo usar?

2- Devuelve las respuesta al enviar un png

3- ¿Qué pasa si no envío un png?
4- ¿A qué salida carpeta de tu proyecto se está guardando en caso de respuesta satisfactoria?

5- ¿En qué formato de dato enviaste la petición?

## Parte 2 (CURL)

Este apartado es el más importante. Con cada uno de los endpoints anteriores se pide que hagas un CURL vía terminal para comprobar que las cosas funcionan bien. Adjunta en markdown, junto con capturas, loes ejemplos en cada endpoints junto con sus respuestas.