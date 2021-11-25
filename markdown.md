# ¿Qué es?

> **Práctica**
> Realiza y documenta todos los puntos de esta entrada de blog 
> Debes crear todas las páginas mencionadas en el mismo y como resultado, debes crear un Dockerfile con dos instalaciones de apache y php: una para el atacante y otra para el hacker
> Además, debes crear un repositorio en GitHub con un commit por cada nuevo archivo crees o modifiques



Nunca hay que confiar en aquello que introducen los usuarios en un formulario web. Estamos acostumbrados a trabajar con ellos en cualquier aplicación web de hoy en día: Facebook, Twitter, Instagram, ... y realmente no nos damos cuenta de lo fácil que es atacar una web a través de ellos si no se toman las debidas precauciones al validar los datos de entrada.

## XSS (Cross-site-scripting)

Vamos a crear un formulario (`post.php`) para introducir entradas de posts (es básico porque no se va a guardar nada en la base de datos. Lo único que va a hacer el formulario es mostrar los datos que se han introducido).

```php
<!DOCTYPE html>
<html lang="es">
<head>
<meta charset="utf-8">
</head>
<body>

<?php
if ($_SERVER['REQUEST_METHOD'] == "GET") {
?>
<p>Nuevo Post</p>
<form action='post.php' method='post'>
	<textarea name="textarea" rows="10" cols="50">Escribe algo aquí</textarea>
	<input type = 'submit' value='enviar'>
</form>
<?php
}else
	echo $_POST["textarea"] ?? "";
?>
</body>
</html>
```

En principio parece inocuo. Para probar escribe unos cuantos posts.

![Post](/Ciberseguridad-PePS/assets/img/validacion/image-20210131193642983.png)

Pero ahora vas a actuar como un hacker y a introducir el siguiente texto 

```html
<script>alert('hackeado')</script>
```

Ahora ya no parece tan inocuo, ¿no?

![Formulario hackeado](/Ciberseguridad-PePS/assets/img/validacion/image-20210131193815141.png)

### Mitigar XSS

**En el servidor**

Como se ha comentado antes, **nunca pero nunca** se ha de confiar en lo que escriben los usuarios en los formularios. Siempre hay que sanear (**sanitize**) de caracteres peligrosos mediante las funciones que provea el lenguaje en el que escribimos la parte del servidor (o la parte cliente que siempre es javascript).

Para ello podemos usar en PHP la función `htmlspecialchars` o `htmlentities`, aunque mejor si usamos un purificador como por ejemplo [http://htmlpurifier.org/](http://htmlpurifier.org/)

Vamos a crear un nuevo archivo llamado `post_mejorado.php` que realiza el escape de los caracteres peligrosos.

```php
<!DOCTYPE html>
<html lang="es">
<head>
<meta charset="utf-8">
</head>
<body>

<?php
if ($_SERVER['REQUEST_METHOD'] == "GET") {
?>
<p>Nuevo Post</p>
<form action='post.php' method='post'>
	<textarea name="textarea" rows="10" cols="50">Escribe algo aquí</textarea>
	<input type = 'submit' value='enviar'>
</form>
<?php
}else
	echo htmlspecialchars($_POST["textarea"]) ?? "";
?>
</html>
```

 Ahora fíjate que al introducir 

```html
<script>alert('hackeado')</script>
```

lo único que ocurre es que se muestra en pantalla lo siguiente:

![Hackeo mitigado](/Ciberseguridad-PePS/assets/img/validacion/image-20210131195056822.png)

**En el cliente**

Mejor aún, si también usamos un purificador como [DOMPurify](https://github.com/cure53/DOMPurify) en la parte del cliente, pues según la información disponible en la página de Gitub:

> DOMPurify sanitizes HTML and prevents XSS attacks. You can feed  DOMPurify with string full of dirty HTML and it will return a string  (unless configured otherwise) with clean HTML. DOMPurify will strip out  everything that contains dangerous HTML and thereby prevent XSS attacks  and other nastiness. It's also damn bloody fast. We use the technologies the browser provides and turn them into an XSS filter. The faster your  browser, the faster DOMPurify will be.

Además, nos hemos de asegurar de que cuando insertamos datos en el DOM, estos se traten como un `String` y no como `DOM`. Por ejemplo:

* Menos seguro

```javascript
const cadena = '<strong>Texto en javascrit</strong>';
const div = document.querySelector('#comentario');
div.innerHTML = cadena; //Se interpreta como DOM
```

* Más seguro

```javascript
const cadena = '<strong>Texto en javascrit</strong>';
const div = document.querySelector('#comentario');
div.innerText = cadena; //Se interpreta como cadena
```