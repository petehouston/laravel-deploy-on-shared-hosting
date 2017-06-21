# Cómo desplegar aplicaciones Laravel en hosting compartido
[![API Documentation](http://img.shields.io/badge/en-English-yellow.svg)](README.md)
[![API Documentation](http://img.shields.io/badge/es-Español-brightgreen.svg)](README-es.md)
[![API Documentation](http://img.shields.io/badge/vi-Ti%E1%BA%BFng%20Vi%E1%BB%87t-yellow.svg)](README-vi.md)

Guía simple de cómo desplegar aplicaciones Laravel y Lumen en servicios de alojamiento compartidos.

Para una versión rápida de esta guía, lee el post original en Medium: "[The simple guide to deploy Laravel 5 application on shared hosting](https://medium.com/laravel-news/the-simple-guide-to-deploy-laravel-5-application-on-shared-hosting-1a8d0aee923e#.7y3pk6wrm)" (en inglés).

## Requisitos

Antes de intentar publicar tu aplicación en un servicio de alojamiento, necesitas asegurarte de que cumple con los [requisitos de Laravel](https://laravel.com/docs/5.2#server-requirements). Básicamente, Laravel `5.2` necesita:

- PHP >= `5.5.9`
- Extensiones PHP:
  - OpenSSL
  - PDO
  - Mbstring
  - Tokenizer

> Estos requisitos pueden variar dependiendo de la versión de Laravel que quieres instalar. Verifica los requisitos del servidor para la versión de Laravel apropiada en la [documentación oficial](https://laravel.com/docs/master).

Además de esto, **necesitas tener permisos de acceso SSH para tu cuenta en tu servicio de alojamiento; de otro modo, nada de lo que sigue funcionará.**

Además de PHP y esas extensiones requeridas, podrías necesitaralgunas utilidades para hacer tu despliegue más fácil:

- [Git](https://git-scm.com/)
- [Composer](https://getcomposer.org/)

Eso es todo por ahora. Por favor, sírvase leer las siguientes secciones para aprender más acerca del despliegue (deployment).

## Intrucciones

Iniciemos por entender cómo deberíamos organizar la estructura de nuestra aplicación Laravel. 

En primer lugar, deberías tener algo similar a estos archivos y carpetas en tu cuenta de tu alojamiento:


```bash
.bash_history
.bash_logout
.bash_profile
.bashrc
.cache
.cpanel
.htpasswds
logs
mail
public_ftp
public_html
.ssh
tmp
etc
www -> public_html
```

El código para el front-end para la cuenta principal, la cuál está vinculada con tu dominio principal, debería estar en `public_html` o `www`. Debido a que no queremos exponer las *cosas de Laravel* (como `,env`, etc..) al mundo exterior, las ocultaremos.

Crea un nuevo directorio para almacenar todo el código; colócale el nombre  `projects` o cualquiera que desees.

```bash
$ mkdir projects
$ cd projects
```

Bien. Desde aquí, simplemente ejecuta un comando `git` para obtener tu código:

```bash
$ git clone http://[GIT_SERVER]/awesome-app.git
$ cd awesome-app
```

El próximo paso es hacer que la carpeta `awesome-app/public` se mapee con la carpeta `www` mencionada anteriormente. Los enlaces simbólicos son de gran ayuda para esto, pero primero necesitamos respaldar la carpeta `public` de nuestro proyecto.

```bash
$ mv public public_bak
$ ln -s ~/www public
$ cp -a public_bak/* public/
$ cp public_bak/.htaccess public/
```

Debido a que creamos el enlace simbólico desde la carpeta `~/www` para hacer que se convierta en la carpeta `public` *virtual* en tu proyecto, debemos actualizar el archivo `~/www/index.php` para reemplazar las rutas viejas con las nuevas:

```diff
- require __DIR__.’/../bootstrap/autoload.php’;
+ require __DIR__.'/../projects/awesome-app/bootstrap/autoload.php';

- $app = require_once __DIR__.’/../bootstrap/app.php’;
+ $app = require_once __DIR__.'/../projects/awesome-app/bootstrap/app.php';
```

El archivo actualizado debera quedar así:

```php
require __DIR__.'/../projects/awesome-app/bootstrap/autoload.php';

$app = require_once __DIR__.'/../projects/awesome-app/bootstrap/app.php';
```

Ya lo difícil está hecho. El resto es hacer algunas configuraciones básicas de Laravel. 

Permitir permisos de escritura al directorio `storage` es importante:

```bash
$ chmod -R o+w storage
```

**Edita tu archivo`.env` con la configuración apropiada. ¡No lo pases por alto!**

Por último, actualiza los paquetes requeridos por tu proyecto Laravel usando **composer** y agrega algunas caché necesarias:

```bash
$ php composer install
$ php composer dumpautoload -o
$ php artisan config:cache
$ php artisan route:cache
```


**¡Felicidades! Has configurado exitosamente una aplicación Laravel en un servicio de alojamiento web compartido.**

## Preguntas Frecuentes (FAQ)

> **1. ¿Cómo adquiero permiso de acceso SSH para mi cuenta?**

Simplemente contacta con el soporte de tu proveedor de hospedaje. Necesitarán confirmar tu identidad y te permitirán acceso SSH inmediatamente.

> **2. ¿En dónde está `git`? No lo piedo encontrar.**

Por lo general, `git` es colocado en esta ruta para proveedores de alojamiento con CPanel, `/usr/local/cpanel/3rdparty/bin/git`. Así que debes acceder a él escribiendo la ruta completa si quieres ejecutar un comando `git`. O, mejor aún, crear un alias más conveniente:

```bash
alias git="/usr/local/cpanel/3rdparty/bin/git"
```

> **3. ¿Cómo obtener composer?**

Puedes usar FTP o comandos SCP para subir el archivo `composer.phar` a tu alojamiento remoto después de descargarlo localmente. También puedes usar `wget` o `curl` para descargar el archivo directamente en tu remoto:

```bash
$ wget https://getcomposer.org/composer.phar
```

ó

```bash
$ curl -sS https://getcomposer.org/installer | php — –filename=composer
```

> **4. ¿Estas instrucciones funciona con Lumen?**

Laravel y Lumen son como gemelos, así que lo mismo aplica para Lumen.

> **5. Intento ejecutar `composer` pero no muestra nada. ¿Cuál es el problema?**

Tienes que proveer una configuración PHP apropiada para ejecutar `composer`, lo que significa que no puedes usar `composer` directamente en algunos servicios de alojamiento. Así que para ejecutarlo, necesitarás ejecutar este comando:

```bash
$ php -c php.ini composer [COMMAND]
```

> **6. ¿De dónde puedo obtener el `php.ini` para cargar con `composer`?**

Puedes copiar la configuración predeterminada de PHP, la cual por lo general está en `/usr/local/lib/php.ini` o puedes encontrarlo con este comando:

```bash
$ php -i | grep "php.ini"
```

## Lista de proveedores de servicio comprobados en los que funciona

Los siguientes servicios de alojamiento compartidos han sido probadas estas instrucciones y funcionado perfectamente al 100%.

* [NameCheap](https://www.namecheap.com/)
* [JustHost](https://www.justhost.com/)
* [bluehost](https://www.bluehost.com/)
* [GoDaddy](https://godaddy.com/)
* [HostGator](http://www.hostgator.com/)
* [GeekStorage](https://www.geekstorage.com/)

Funciona en el plan compartido de [GeekStorage](https://www.geekstorage.com/), pero hay que habilitar PHP 5.6 a través de .htaccess: `AddHandler application/x-httpd-php56 .php`


Si encuentras algún otro proveedor de servicio en que funcione, por favor, notifícalo. Esta lista se actualizará para informar a otros acerca de ello también.

## ¿Aún tienes problemas?

Si sigues fallando al intentar desplegar tu aplicación, puedes crear una nueva [insidencia](https://github.com/petehouston/laravel-deploy-on-shared-hosting/issues)  con los detalles para ayudarte.

## Guía de Contribución

Siéntete libre de clonar (folk) el proyecto a tu cuenta y enviar una petición de [pull](https://github.com/petehouston/laravel-deploy-on-shared-hosting/pulls).
