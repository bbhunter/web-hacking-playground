# Soluciones

## Etapa 1: Acceder con cualquier usuario
Para empezar, accedemos a http://whp-socially/ y nos encontramos con la siguiente pantalla:

![](img/MainPage.png)

Vemos que en el menú lateral izquierdo tenemos la opción de "Login", pero no es importante dado que no tenemos ninguna cuenta y no podemos registrarnos.

Si revisamos las publicaciones de la página, vemos que hay una publicación de un usuario llamado "admin" con un enlace que nos lleva a la página de Google.

![](img/InterestingHyperlink.png)

Este enlace es relevante, porque no redirecciona directamente a Google, sino que usa un parámetro llamado "next" para redireccionar a la página que nosotros queramos.

Esto recuerda a una vulnerabilidad llamada "Open Redirect", que consiste en que un atacante puede redireccionar a un usuario a una página que no es la que el usuario espera, por ejemplo, a una página de phishing.

Para validar si esto es una vulnerabilidad, podemos manipular el parámetro "next", quedando el enlace de la siguiente manera:

http://whp-socially/?next=http://example.com

![](img/example.com.png)

Y al acceder a este enlace, nos redirecciona a la página example.com, lo que confirma la vulnerabilidad.

Open Redirect es una vulnerabilidad muy común, normalmente no es muy peligrosa y en muchos casos se reporta con criticidad baja o informativa. Sin embargo, hay casos en las que se puede explotar para realizar ataques más complejos, como por ejemplo, un Cross-Site Scripting (XSS). Vamos a verificar si esto es posible.

Para esto, tenemos que identificar cómo realiza la aplicación la redirección. Podemos usar Burp Suite para interceptar la petición y ver qué está pasando.

![](img/RequestOpenRedirect.png)

La aplicación redirecciona utilizando código JavaScript, más específicamente, utilizando la propiedad "href" del objeto "window.location".

Cuando se redirecciona mediante JavaScript, y no mediante una cabecera HTTP "Location", se puede escalar el Open Redirect a un XSS. Esto resulta muy útil para Bug Bounty, porque los XSS se reportan con mayor criticidad que los Open Redirect, brindando una mayor recompensa.

Intentemos explotar esto. Primero, probemos si "javascript:" nos funciona, para ver si podemos ejecutar código JavaScript.

![](img/javascriptBlocked.png)

Al parecer, "javascript:" está bloqueado. No obstante, mediante el uso del carácter %09 (tabulador codificado en URL), podemos evadir los filtros.

Para esto, agregamos dicho carácter entre la primera y la última letra de la palabra "javascript", quedando de la siguiente manera:

![](img/BypassFilter.png)

Este carácter genera un espacio en blanco, que es ignorado por el navegador.

Con esto, podemos ejecutar código JavaScript. Vamos a intentar llamar a la función "alert()" para ver si funciona.

![](img/alertBlocked.png)

Al parecer, la función "alert()" también está bloqueada. Sin embargo, podemos utilizar la función "print()", que genera una ventana de impresión.

![](img/printAllowed.png)

¡Perfecto, funciona! Accedamos desde el navegador para confirmar que se ejecuta el código JavaScript.

![](img/printExecuted.png)

El código JavaScript se ejecuta correctamente. No obstante, no es muy útil, ya que solo genera una ventana de impresión. Vamos a intentar algo más interesante, como por ejemplo, robar la sesión del usuario.

Pero antes, necesitamos identificar cómo se almacena la sesión del usuario. El código fuente de la página nos muestra que se utiliza un token almacenado en el Local Storage del navegador.