---
title:  "Minería de Datos en APIs Privadas"
date:   2015-08-17
description: Como para Extraer Información de APIs Privadas de Aplicaciones en tu Celular
---

## Introducción
Existen muchos escenarios dentro de los cuales es útil tener acceso a la información de servicios externos. La forma más común de extraer estos datos, es a través de webscraping con alguna herramienta como [Mechanize](https://github.com/sparklemotion/mechanize) y/o [Beautiful Soup](http://www.crummy.com/software/BeautifulSoup/). El problema con estas técnicas es que dependen de una interfaz web. Estas usualmente cambian frecuentemente dejando nuestros scripts de scraping sin funcionar. Otra alternativa es utilizar las APIs mediante las cuales se conectan los clientes oficiales, usualmente aplicaciones para dispositivos móviles. En este caso, aunque la interfaz de la aplicación móvil puede cambiar, usualmente los endpoints detrás siguen intactos. Otra ventaja es que normalmente la información que se obtiene de las APIs esta organizada y puede ser procesada con mayor facilidad.

El problema con este método es que en la mayoría de las ocasiones las comunicaciones se encuentran protegidas por SSL por lo que es necesario ensuciarse un poco las manos para poder tener acceso a los servicios.

En este post voy a explicar como intervenir las comunicaciones de tu dispositivo móvil para registrar las solicitudes que realiza y sus respectivas respuestas de los diferentes servicios a los que consulta.

## ¿Como Funciona?

![Funcionamiento de Mitmproxy]({{ site.baseurl }}assets/posts-images/como-para-extraer-informacion-de-apis-privadas-de-aplicaciones-en-tu-celular/explicit.png)

Este tipo de ataque se llama [Man in the Middle](https://en.wikipedia.org/wiki/Man-in-the-middle_attack) y consiste en desviar las comunicaciones de un dispositivo hacia un proxy donde registras todas las comunicaciones. El dispositivo debe tener instalado un certificado que permite al proxy desencriptar las comunicaciones generando certificados SSL de forma dinámica según el servicio que estés consultando. Para más información sobre como funciona este ataque les sugiero este [artículo de mitmproxy](https://mitmproxy.org/doc/howmitmproxy.html).

## Herramientas

Para hacer esto existen varias herramientas como [Wireshark](https://www.wireshark.org/), [Charles Proxy](http://www.charlesproxy.com/) y la que usaremos hoy que es [mitmproxy](https://mitmproxy.org/) ya que es gratuita y fácil de utilizar si estas acostumbrado a utilizar terminal.

Para explorar el API más fácilmente una vez que hayamos extraído la información de las solicitudes les sugiero utilizar [Postman](https://chrome.google.com/webstore/detail/postman/fhbjgbiflinjbdggehcddcbncdddomop?hl=en), otra herramienta que recomiendo si tienen Mac es [Paw](https://luckymarmot.com/paw).

También ocuparemos un dispositivo móvil. En este tutorial utilizaremos un iPhone. El proceso es muy similar en el caso de Android.

## Instalación

Primero debemos instalar mitmproxy desde la terminal:

`$ pip install mitmproxy`

En caso de requerir una explicación más detallada sobre como instalar mitmproxy les sugiero revisar [este artículo de su documentación](https://mitmproxy.org/doc/install.html).

Una vez instalado, ejecutaremos la aplicación para generar el certificado que instalaremos en nuestro móvil.

`$ mitmproxy`

Al iniciarse por primera vez mitmproxy generará los certificados y los guardará en el folder `~/.mitmproxy`. Ahora tomaremos el archivo `mitmproxy-ca-cert.pem` y lo enviamos por correo a nuestro dispositivo e instalamos el certificado. Esto lo hacemos ejecutando el archivo `.pem` adjunto y siguiendo las instrucciones de nuestro sistema operativo respectivo.

Ahora debemos obtener la dirección de IP de nuestra computadora en la red local y asegurarnos que nuestro dispositivo se encuentre en la misma red.

<img src="{{ site.baseurl }}assets/posts-images/como-para-extraer-informacion-de-apis-privadas-de-aplicaciones-en-tu-celular/ios.png" alt="Configuracion de Proxy en iOS" style="width: 250px" class="center">

En nuestro dispositivo debemos añadir el proxy en la red a la cual nos encontramos conectados. En el campo de dirección deberás ingresar la dirección de la computadora donde estas corriendo mitmproxy. En el campo de puerto usarás 8080 que es el que utiliza mitmproxy por default.

Una vez realizado esto podemos utilizar cualquier aplicación dentro de nuestro móvil que utilice internet y comenzaremos a ver el tráfico.

![mitmproxy en funcionamiento]({{ site.baseurl }}assets/posts-images/como-para-extraer-informacion-de-apis-privadas-de-aplicaciones-en-tu-celular/terminal1.png)

Pueden iterar sobre los resultados con las flechas o inclusive dando click a los resultados dentro de la terminal. Dentro de los resultados podemos iterar entre Request, Response y Detalles con tab.

![]({{ site.baseurl }}assets/posts-images/como-para-extraer-informacion-de-apis-privadas-de-aplicaciones-en-tu-celular/terminal2.png)

Utilizando la aplicación del servicio del que nos interese obtener información podemos comenzar a explorar como funciona. Por ejemplo, con la aplicación de Hellofood podemos obtener información sobre restaurantes cercanos a ciertas coordenadas que nosotros podemos especificar:

![]({{ site.baseurl }}assets/posts-images/como-para-extraer-informacion-de-apis-privadas-de-aplicaciones-en-tu-celular/terminal3.png)

Cuando estemos realizando las solicitudes desde Postman no debemos olvidar incluir los headers y parámetros requeridos. Estos pueden variar en diferentes servicios. En el caso de Hellofood, ellos requieren que les indiques el dispositivo del que estas ingresando.

![]({{ site.baseurl }}assets/posts-images/como-para-extraer-informacion-de-apis-privadas-de-aplicaciones-en-tu-celular/postman.png)

A partir de esto podemos comenzar a escribir nuestros scripts para accesar de forma programática a estos servicios. La información que esta en los headers, o en la estructura de la información también nos puede ayudar a determinar que servicios utilizan y la forma en la que esta construido el servicio.

## Conclusión

Con este tipo de métodos podemos extraer grandes cantidades de información de los servicios por lo que debemos ser responsables con la manera en la que la extraemos. Es conveniente hacer las solicitudes con intervalos de tiempo razonables para no saturar los servicios de los que estamos extrayendo información. Si identificamos vulnerabilidades que expongan información sensible debemos informar a los administradores de los servicios para que puedan tomar las medidas pertinentes.
