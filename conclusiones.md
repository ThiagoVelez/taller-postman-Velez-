# Conclusiones

## Idempotencia en métodos HTTP

**¿Qué significa la idempotencia?**
Un método HTTP es idempotente cuando el resultado de ejecutar la misma petición una sola vez es exactamente el mismo que si la ejecutas múltiples veces seguidas. Es decir, después de la primera petición exitosa, el estado del servidor (la base de datos) no vuelve a cambiar por más que repitas la misma acción.

**Clasificación de los métodos:**
* **GET (Idempotente):** Solo lee información. Puedes consultar un recurso 100 veces y no alterarás ningún dato en el servidor.
* **PUT (Idempotente):** Reemplaza un recurso por completo. Si envías la misma actualización 50 veces, la primera sobrescribe el dato y las demás vuelven a guardar exactamente lo mismo en el mismo lugar. El estado final del servidor se mantiene igual tras la primera ejecución.
* **DELETE (Idempotente):** Elimina un recurso. La primera vez lo borra; si lo intentas de nuevo, el recurso simplemente sigue sin existir (aunque el código de respuesta cambie a 404, el estado de la base de datos no sufre nuevas alteraciones).
* **POST (No idempotente):** Se usa para crear. Si ejecutas la misma petición 10 veces, le estás indicando al servidor que cree 10 registros nuevos y duplicados, cambiando el estado en cada ejecución.
* **PATCH (No idempotente):** Por definición estándar no lo es, ya que una instrucción de actualización parcial (por ejemplo, "súmale 1 a la cantidad") alterará el estado del servidor y sumará datos cada vez que se dispare la petición.

**Comprobación práctica en Postman (PUT vs POST):**
* Al ejecutar **POST** varias veces seguidas, el comportamiento observado es que se intenta agregar un elemento nuevo en cada clic. En una API real, esto se traduciría en la creación de múltiples registros duplicados en la base de datos, incrementando el volumen de datos con cada envío.
* Al ejecutar **PUT** varias veces seguidas apuntando a un recurso específico (ej. `/posts/1`), el primer clic actualiza la información. Las ejecuciones posteriores simplemente vuelven a enviar los mismos datos para reemplazar el mismo recurso, por lo que el sistema no crea registros adicionales ni modifica nada nuevo. El estado final queda idéntico al del primer intento.

---

## Análisis de Cabeceras (Headers) de Respuesta

**1. Content-Type**
* **Qué significa:** Indica el tipo de formato (MIME type) en el que el servidor está enviando los datos de la respuesta. Por ejemplo: `application/json; charset=utf-8`.
* **Para qué sirve:** Le avisa al cliente (tu navegador o Postman) cómo debe interpretar, procesar y mostrar la información recibida. 
* **Por qué es vital al probar una API:** Porque define si el sistema receptor entenderá los datos. Si una API devuelve un archivo JSON perfecto, pero la cabecera dice `text/plain` o se omite, tu aplicación frontend (o Postman) tratará el JSON como una simple cadena de texto sin formato. Esto causará errores críticos al intentar acceder a las variables y propiedades del objeto.

**2. Cache-Control**
* **Qué significa:** Define las reglas y directivas sobre cómo, dónde y durante cuánto tiempo se puede almacenar en memoria caché la respuesta.
* **Para qué sirve:** Optimiza el rendimiento y ahorra recursos. Si el servidor responde con `max-age=43200`, le está ordenando a tu sistema que guarde esos datos localmente durante 12 horas. Si repites exactamente la misma petición dentro de ese tiempo, tu sistema no contactará al servidor de nuevo, sino que usará la copia guardada.

**3. Server**
* **Qué significa:** Revela información sobre el software, sistema operativo o infraestructura que procesó la solicitud en el backend.
* **Para qué sirve:** Se utiliza para diagnóstico y auditorías técnicas (en JSONPlaceholder suele indicar `cloudflare`, revelando que usan esa infraestructura para gestionar su tráfico). En entornos de producción reales, los desarrolladores suelen ocultar o alterar esta cabecera por seguridad, para no darle pistas a los atacantes sobre las tecnologías específicas (como Nginx, Express o Apache) que podrían vulnerar.
