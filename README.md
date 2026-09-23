# Tarea: Investigación sobre APIs y Postman

## Marco conceptual

Una API REST es una interfaz que permite la comunicación entre sistemas a través de internet utilizando el protocolo HTTP de manera estructurada. Que sea «REST» (Representational State Transfer) significa principalmente que las peticiones son "sin estado"; el servidor no guarda información de las sesiones anteriores, por lo que cada petición debe contener todo lo necesario para ser procesada. En esta arquitectura, un recurso es la entidad de información o el objeto que se desea consultar o manipular (por ejemplo, un "usuario", una "factura" o un "producto"), mientras que un endpoint es la URL o ruta web específica a la que se hace la petición para acceder a dicho recurso (por ejemplo, https://api.miaplicacion.com/v1/usuarios).

Un ejemplo de una aplicación de uso diario que depende de APIs es Spotify. La aplicación en el celular (el cliente) no almacena el catálogo de canciones; cada vez que buscas un artista o reproduces una lista, la aplicación se conecta a los endpoints de la API de Spotify para solicitar y recibir esos recursos en tiempo real.

* **Fuente consultada:** Amazon Web Services (AWS). "¿Qué es una API de RESTful?". Recuperado de: https://aws.amazon.com/es/what-is/restful-api/

---

## Métodos HTTP

| Método | Operación CRUD | Qué hace |
| :--- | :--- | :--- |
| **GET** | Read (Leer) | Solicita la representación de un recurso específico. Es una operación de solo lectura, sirve para obtener datos sin modificarlos en el servidor. |
| **POST** | Create (Crear) | Envía datos al servidor para crear un nuevo recurso o entidad desde cero. |
| **PUT** | Update (Actualizar) | Reemplaza todas las representaciones actuales del recurso de destino con los datos nuevos proporcionados en la petición. |
| **PATCH** | Update (Actualizar) | Aplica modificaciones parciales a un recurso. Se usa cuando solo se necesita actualizar un campo específico (ej. cambiar solo la contraseña). |
| **DELETE** | Delete (Eliminar) | Elimina un recurso específico del servidor. |

---

## Códigos de estado

Los códigos de estado HTTP indican si una petición se completó con éxito o no, y se agrupan en las siguientes cinco familias:

* **1xx (Respuestas informativas):** Indican que el servidor recibió la petición y el proceso continúa. Ejemplo: `100 Continue`.
* **2xx (Respuestas satisfactorias):** Indican que la petición fue recibida, entendida y aceptada exitosamente. Ejemplo: `200 OK` o `201 Created`.
* **3xx (Redirecciones):** Indican que el cliente debe tomar una acción adicional para completar la petición (usualmente ir a otra URL). Ejemplo: `301 Moved Permanently`.
* **4xx (Errores del cliente):** Indican que hubo un problema con la petición enviada por el cliente. Ejemplo: `404 Not Found` (el endpoint no existe) o `401 Unauthorized`.
* **5xx (Errores de los servidores):** Indican que el servidor falló al procesar una petición que aparentemente era válida. Ejemplo: `500 Internal Server Error`.

### ¿Por qué se separan los errores 4xx de los 5xx y qué cambia sobre quién tiene la culpa?

Se separan para identificar rápidamente de qué lado de la comunicación está el problema.

Desde el punto de vista de "quién tiene la culpa":

* En un error **4xx** la culpa es del cliente (quien hace la petición). Significa que nosotros enviamos mal la solicitud: escribimos mal la URL, nos faltó enviar un dato obligatorio, o no enviamos los permisos necesarios. La solución recae en corregir cómo hacemos la petición.
* En un error **5xx** la culpa es del servidor (el backend). Significa que nuestra petición estaba construida perfectamente, pero el servidor se bloqueó, el código falló internamente o la base de datos se cayó. La solución recae en el equipo de desarrollo backend para reparar el sistema.

* **Fuente consultada:** Mozilla Developer Network (MDN Web Docs). "Códigos de estado de respuesta HTTP". Recuperado de: https://developer.mozilla.org/es/docs/Web/HTTP/Status
