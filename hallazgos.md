# Hallazgos: Comparación de peticiones GET

## ¿En qué se diferencian los criterios de aceptación?

Aunque ambas peticiones devuelven `200 OK` y los mismos 4 campos por elemento, los criterios de aceptación difieren en lo siguiente:

### Al pedir un recurso individual (`GET /posts/1`):

* La respuesta es un objeto JSON (no un array).
* Se valida que el `id` del objeto coincida exactamente con el solicitado en la URL.
* Si el recurso no existe, se espera un `404 Not Found`.
* **El criterio clave es la identidad:** que sea el elemento correcto.

### Al pedir una colección (`GET /posts`):

* La respuesta es un array JSON.
* Se valida que el array no esté vacío y que contenga la cantidad esperada de elementos (en este caso 100).
* Se puede validar que todos los elementos tengan la misma estructura de campos.
* **El criterio clave es la completitud y estructura:** que lleguen todos los elementos con el formato correcto.

> **En resumen:** para un recurso individual importa qué elemento llegó; para una colección importa cuántos elementos llegaron y que todos tengan la estructura correcta.

---

## Análisis de Caso de Prueba: Error 404 (`GET /posts/9999`)

### ¿El caso de prueba pasó o falló?

**El caso pasó.**

Un caso de prueba no falla porque el código sea diferente de 200. Falla cuando el resultado obtenido difiere del resultado esperado. En este caso:

* **Resultado esperado:** `404 Not Found`, porque el recurso `/posts/9999` no existe.
* **Resultado obtenido:** `404 Not Found`.

Ambos coinciden, por lo tanto el comportamiento de la API es correcto y el caso de prueba es exitoso. El `404` no es un error del sistema, es la respuesta semánticamente correcta para un recurso inexistente según el estándar HTTP.

### ¿Qué pasaría si hubiera devuelto 200 con cuerpo vacío?

**Sí sería un defecto.**

Si la API devolviera `200 OK` con cuerpo `{}` o `null` para un recurso que no existe, el resultado obtenido diferiría del esperado (`404`). Eso constituye un defecto porque:

* **Semánticamente es incorrecto:** un `200` le dice al cliente "encontré lo que pediste", pero el cuerpo vacío contradice eso.
* **Rompe los criterios de aceptación:** el criterio para este caso es que la API comunique que el recurso no existe, y un `200` no lo hace.
* **Genera problemas en cascada:** cualquier cliente que consuma esta API interpretaría que el recurso existe pero está vacío, en lugar de manejarlo como un "no encontrado".

> **En resumen:** un defecto no lo define el código de estado en sí, sino la diferencia entre lo que se espera y lo que se obtiene. Un 404 puede ser un éxito; un 200 puede ser un defecto.

---

## Análisis de Peticiones POST (`POST /posts`)

### Resultados de las 5 ejecuciones:

| Ejecución | Status | id devuelto |
| :---: | :---: | :---: |
| 1 | 201 Created | 101 |
| 2 | 201 Created | 101 |
| 3 | 201 Created | 101 |
| 4 | 201 Created | 101 |
| 5 | 201 Created | 101 |

**Observación importante:** El id devuelto es siempre 101 en todas las ejecuciones. Esto se debe a que JSONPlaceholder es una API de prueba falsa (fake REST API) — no persiste datos realmente. Cada vez que haces un POST, simula que crea el recurso con el siguiente ID disponible (101, ya que hay 100 posts existentes), pero como no guarda nada, siempre responde con el mismo id: 101.

En una API real, cada POST crearía un recurso nuevo con un ID único e incremental (101, 102, 103...).

---

## Comparación entre PUT y PATCH (`PUT /posts/1` vs `PATCH /posts/1`)

Ejecuté ambas peticiones enviando únicamente:

```json
{
  "title": "foo updated"
}
```

`PUT /posts/1` respondió:

```json
{
  "title": "foo updated",
  "id": 1
}
```

`PATCH /posts/1` respondió:

```json
{
  "userId": 1,
  "id": 1,
  "title": "foo updated",
  "body": "quia et suscipit..."
}
```

La diferencia principal es que **PUT** reemplaza el recurso completo con los datos enviados. Como solo se envió `title`, la respuesta perdió los campos que no se enviaron, como `userId` y `body`.

En cambio, **PATCH** actualiza parcialmente el recurso. Solo modificó el campo `title` y mantuvo los demás campos existentes (`userId`, `id` y `body`).

Para corregir un error de escritura en un solo campo, usaría **PATCH**, porque permite modificar únicamente ese campo sin afectar ni reemplazar el resto de la información del recurso.

---

## Tarea 10: Encuentra el límite

### Identificación de límites en la API (`/posts`)
* **ID más alto que devuelve 200 OK:** `id = 100` (`GET /posts/100` responde `200 OK`).
* **Primer ID que devuelve 404 Not Found:** `id = 101` (`GET /posts/101` responde `404 Not Found`).

### ¿Cómo se llama este tipo de caso de prueba?
Se denomina **Prueba de Valores Límite** (en inglés, *Boundary Value Testing* o *Boundary Value Analysis - BVA*). Consiste en diseñar casos de prueba justo en los extremos o fronteras de las clases de equivalencia (en este caso, el último elemento válido y el primer elemento fuera de rango).

### ¿Por qué se dice que los defectos se concentran ahí?
En ingeniería de software y control de calidad, los defectos se concentran en las fronteras debido a los errores típicos de programación conocidos como **errores por uno (off-by-one errors)**. Los desarrolladores comúnmente cometen equivocaciones en operadores de comparación condicionales (por ejemplo, escribir `<` en lugar de `<=`, usar índices mal acotados o confusiones al iterar colecciones y arrays). Las pruebas en los límites son las más eficaces para descubrir estas fallas sin necesidad de probar exhaustivamente todos los valores intermedios.

---

## Tarea 11: Exploración de otros recursos y rutas anidadas

### 1. Pruebas de recursos adicionales de JSONPlaceholder

Se exploraron y probaron dos recursos nuevos que ofrece la API:

* **Recurso `/users` (`GET /users/1`):**
  * **Status devuelto:** `200 OK`
  * **Qué contiene:** Representa la entidad de un usuario completo (incluye nombre, username, correo electrónico, dirección física con coordenadas y datos de la empresa).
* **Recurso `/todos` (`GET /todos/1`):**
  * **Status devuelto:** `200 OK`
  * **Qué contiene:** Representa una tarea o pendiente (*to-do list*) con los campos `userId`, `id`, `title` y el booleano `completed` (estado de la tarea).

### 2. Prueba de ruta anidada (`GET /posts/1/comments`)
* **URL probada:** `https://jsonplaceholder.typicode.com/posts/1/comments`
* **Status devuelto:** `200 OK`
* **Qué se encontró:** Devuelve un array con todos los comentarios asociados específicamente a la publicación con `id: 1` (cada comentario incluye su propio `id`, `name`, `email`, `body` y la clave foránea `postId: 1`).

### ¿Cómo se dedujo la estructura de estas URLs?
Se dedujo siguiendo las convenciones estándar del diseño de **APIs RESTful**:
1. **Jerarquía y relación padre-hijo:** Cuando un recurso secundario depende o pertenece directamente a uno principal, la URL refleja esa relación jerárquica en su ruta:
   ```text
   /{recurso_padre}/{id_padre}/{recurso_hijo}
   ```
2. **Aplicación al caso:** En este modelo de datos, una publicación tiene múltiples comentarios asociados. Por lo tanto, para consultar únicamente los comentarios que pertenecen al post `1`, la estructura natural y RESTful es acceder a la colección `/posts`, especificar el identificador `/1` y solicitar su sub-recurso `/comments`.

---

## Tarea 13: Escribe tus propias pruebas con `pm.expect`

En Postman (y herramientas compatibles), las pruebas automatizadas se escriben en JavaScript dentro de la pestaña **Scripts / Tests** utilizando la sintaxis de aserciones de Chai (`pm.test` y `pm.expect`). A continuación, se diseñaron e implementaron cuatro pruebas distintas que evalúan aspectos clave de calidad:

### Prueba 1: Validación de tiempo de respuesta (Rendimiento)
* **Petición donde se aplica:** `GET /posts/1` o a nivel de colección.
* **Qué verifica:** Que el servidor procese y responda a la solicitud en un tiempo aceptable (menos de 500 milisegundos), garantizando que la API cumpla con los estándares de rendimiento y no genere retrasos perceptibles para el usuario.
* **Código de la prueba:**
```javascript
pm.test("El tiempo de respuesta es menor a 500 ms", function () {
    pm.expect(pm.response.responseTime).to.be.below(500);
});
```

---

### Prueba 2: Validación de tipos de datos y presencia de campos (Estructura y Esquema)
* **Petición donde se aplica:** `GET /posts/1`.
* **Qué verifica:** Que el cuerpo de la respuesta contenga los campos requeridos (`userId`, `id`, `title`, `body`) y que cada propiedad corresponda exactamente con el tipo de dato esperado (`number` para identificadores y `string` para texto), evitando que lleguen valores nulos o tipos incompatibles.
* **Código de la prueba:**
```javascript
pm.test("La respuesta contiene todos los campos requeridos con sus tipos correctos", function () {
    const post = pm.response.json();
    
    // Verificación de existencia de propiedades
    pm.expect(post).to.have.property("userId");
    pm.expect(post).to.have.property("id");
    pm.expect(post).to.have.property("title");
    pm.expect(post).to.have.property("body");

    // Verificación de tipos de datos
    pm.expect(post.userId).to.be.a("number");
    pm.expect(post.id).to.be.a("number");
    pm.expect(post.title).to.be.a("string");
    pm.expect(post.body).to.be.a("string");
});
```

---

### Prueba 3: Validación de cantidad de elementos en una colección (Integridad del Arreglo)
* **Petición donde se aplica:** `GET /posts`.
* **Qué verifica:** Que la respuesta sea un arreglo (*array*), que no esté vacío y que contenga exactamente la cantidad esperada de publicaciones (100 elementos), asegurando la integridad total de los datos devueltos por el catálogo.
* **Código de la prueba:**
```javascript
pm.test("La respuesta es un arreglo con exactamente 100 publicaciones", function () {
    const posts = pm.response.json();
    
    // Verifica que sea un array y no un objeto único
    pm.expect(posts).to.be.an("array");
    
    // Verifica la longitud exacta de la lista
    pm.expect(posts).to.have.lengthOf(100);
});
```

---

### Prueba 4: Validación de cabecera de contenido y codificación (Headers HTTP)
* **Petición donde se aplica:** `GET /posts` o `POST /posts`.
* **Qué verifica:** Que la cabecera `Content-Type` de la respuesta incluya `application/json` y codificación UTF-8, garantizando que el cliente receptor pueda interpretar y deserializar correctamente el contenido sin problemas de codificación de caracteres especiales.
* **Código de la prueba:**
```javascript
pm.test("La cabecera Content-Type especifica formato JSON y UTF-8", function () {
    const contentType = pm.response.headers.get("Content-Type");
    pm.expect(contentType).to.include("application/json");
    pm.expect(contentType).to.include("charset=utf-8");
});
```

---

## ¿Por qué es importante ver una prueba fallar antes de confiar en ella?

Es crucial ver una prueba fallar por varias razones:

* **Valida que la prueba realmente funciona:** Si una prueba nunca falla, podría estar mal escrita o no estar verificando realmente lo que debería. Al verla fallar, confirmas que está activa y detectando problemas.
* **Evita falsos positivos:** Una prueba que siempre pasa sin importar el estado real de la API es inútil. Ver que falla cuando las condiciones no se cumplen demuestra que está haciendo su trabajo.
* **Genera confianza en el resultado:** Cuando ves que la prueba pasa después de verla fallar, sabes que el resultado es confiable y no es un accidente.
* **Detecta problemas en la lógica de prueba:** Si esperas que falle pero no lo hace, significa que tu prueba tiene un error lógico que necesita corrección.
* **Implementa el ciclo TDD:** En desarrollo dirigido por pruebas (Test-Driven Development), primero escribes la prueba (que falla), luego haces que pase. Esto asegura que el código cumple con los requisitos.

> **En resumen:** una prueba que nunca falla es sospechosa. Necesitas verla fallar para confiar en que realmente está validando lo que debería.
