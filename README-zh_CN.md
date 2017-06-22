# 如何在共享主机上部署Laravel应用程序
[![API Documentation](http://img.shields.io/badge/en-English-brightgreen.svg)](README.md)
[![API Documentation](http://img.shields.io/badge/es-Español-yellow.svg)](README-es.md)
[![API Documentation](http://img.shields.io/badge/vi-Ti%E1%BA%BFng%20Vi%E1%BB%87t-yellow.svg)](README-vi.md)
[![API Documentation](https://img.shields.io/badge/zh_CN-%E4%B8%AD%E6%96%87%EF%BC%88%E4%B8%AD%E5%9B%BD%E5%A4%A7%E9%99%86%EF%BC%89-yellow.svg)](README-zh_CN.md)

本简要指南介绍如何在共享主机上部署Laravel和Lumen应用程序。

本指南有一个更加简明的版本（也许你已经读过），阅读该指南请移步"[在共享主机上部署Laravel 5应用程序的简单指南（英文）](https://medium.com/laravel-news/the-simple-guide-to-deploy-laravel-5-application-on-shared-hosting-1a8d0aee923e#.7y3pk6wrm)"

## 系统要求

尝试在共享主机上部署Laravel应用程序之前，需要确保主机服务提供适合[Laravel要求](https://laravel.com/docs/5.2#server-requirements)的环境。 例如，Laravel 5.2的基本要求如下：

* PHP >= 5.5.9
* OpenSSL PHP 扩展
* PDO PHP 扩展
* Mbstring PHP 扩展
* Tokenizer PHP 扩展

总之，具体取决于你想安装的Laravel版本，请检查相应版本的[Laravel文档的服务器要求页面](https://laravel.com/docs/master)。

**接下来，你必须拥有虚拟主机的SSH访问权限。否则下面的内容都可以忽略。**

除了PHP和上述必备扩展，下列实用程序可以使部署更加容易。

* [Git](https://git-scm.com/)
* [Composer](https://getcomposer.org/)

有了这些就好办了。接下来，请参阅以下内容，详细了解部署Laravel应用程序的更多信息。

## 部署说明

首先，让我们来了解如何组织Laravel应用程序文件结构。 一开始，您的虚拟主机用户文件夹下，会有类似下列目录和文件：

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
...
```

对于与主域名绑定的主账户，前端代码应该保存在“public_html”或“www”目录中。 我们当然不想将 *Laravel文件* （如.env 等...）暴露给全世界，所以我们要把它们放在其他目录中。

创建一个与“public_html”或“www”平行的目录，将它命名为`projects`或其他适合的名字。

```bash
$ mkdir projects
$ cd projects
```

好了，接下来我们可以利用`git`命令来获取代码。

```bash
$ git clone http://[GIT_SERVER]/awesome-app.git
$ cd awesome-app
```

下一步是要将`awesome-app/public`目录映射到`www`目录。这个时候符号连接（symbolic link）非常有用。不过，首先我们需要备份`public`文件夹。

```bash
$ mv public public_bak
$ ln -s ~/www public
$ cp -a public_bak/* public/
$ cp public_bak/.htaccess public/
```

因为我们把`www`目录映射成项目的*虚拟*`public`目录，所以，我们需要编辑`~/www/index.php`，更新文件路径：

```diff
- require __DIR__.’/../bootstrap/autoload.php’;
+ require __DIR__.'/../projects/awesome-app/bootstrap/autoload.php';

- $app = require_once __DIR__.’/../bootstrap/app.php’;
+ $app = require_once __DIR__.'/../projects/awesome-app/bootstrap/app.php';
```

更新后的文件应该是这样：

```php
require __DIR__.'/../projects/awesome-app/bootstrap/autoload.php';

$app = require_once __DIR__.'/../projects/awesome-app/bootstrap/app.php';
```

好了，麻烦的部分到此结束。接下来就似乎一些基本的Laravel设置了。首先，确保`storage`目录可写：

```bash
$ chmod -R o+w storage
```

**然后，编辑`.env`文件，确保配置正确。这步不要忘记！**

最后，使用**composer**来安装或更新必要的依赖包，并添加必要的缓存文件：

```bash
$ php composer install
$ php composer dumpautoload -o
$ php artisan config:cache
$ php artisan route:cache
```

**恭喜！到此，你已成功在共享虚拟主机上设置Laravel应用程序。**

## 常见问题

> **1. 如何获取我的帐户的SSH访问权限？**

请联系您的主机客服，确认您的身份后，您将立即可以获得SSH访问权限。

> **2. git在哪里？ 我怎么找不到。**

在CPanel主机服务中，`git`一般安装在`/usr/local/cpanel/3rdparty/bin/git`。所以，如果想要使用`git`命令，则要提供`git`的完整路径。 当然，也可以创建别名以方便使用：

```bash
alias git="/usr/local/cpanel/3rdparty/bin/git"
```

> **3. 如何安装composer？**

可以使用FTP或SCP命令来上传将下载好的`composer.phar`上传到虚拟主机。也可以直接使用`wget`或`curl`在主机上直接下载：

```bash
$ wget https://getcomposer.org/composer.phar
```

或者

```bash
$ curl -sS https://getcomposer.org/installer | php — –filename=composer
```

> **4. 这个指南可以用于Lumen吗？**

Laravel和Lumen就像一对双胞胎，所以Lumen也适用。

> **5. 我尝试使用`composer`命令，但什么也没显示。发生了什么？**

运行`composer`需要提供php环境信息。也就是说，在某些虚拟主机上不能直接运行`composer`。要运行`composer`，需要使用如下命令：

```bash
$ php -c php.ini composer [命令]
```

> **6. 怎么找到`composer`需要加载的`php.ini`文件？**

你可以将默认`php.ini`文件复制过来。这个文件一般保存在`/usr/local/lib/php.ini`。当然也可以用这个命令来查找：

```bash
$ php -i | grep "php.ini"
```

## 测试可用的提供商

经测试，以下共享主机服务提供商100％可部署Laravel应用。

* [NameCheap](https://www.namecheap.com/)
* [JustHost](https://www.justhost.com/)
* [bluehost](https://www.bluehost.com/)
* [GoDaddy](https://godaddy.com/)
* [HostGator](http://www.hostgator.com/)
* [GeekStorage](https://www.geekstorage.com/)
* [Site5](https://www.site5.com/)

在[GeekStorage](https://www.geekstorage.com/)的共享服务上也可用，不过要通过`.htaccess` 的
`AddHandler application/x-httpd-php56.php`来启用 PHP 5.6。

如果确定其他提供商可用，欢迎添加到此列表。

## 仍有问题？

如果采用上述步骤还没有解决安装问题，请提出你的详细[问题](https://github.com/petehouston/laravel-deploy-on-shared-hosting/issues)，我会来帮你。

## 如何贡献

欢迎fork这个项目，也欢迎提交[pull request](https://github.com/petehouston/laravel-deploy-on-shared-hosting/pulls).
