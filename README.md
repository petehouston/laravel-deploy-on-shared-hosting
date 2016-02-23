# How to deploy Laravel applications on shared hosting

The simple guide to deploy Laravel and Lumen application on shared hosting.

For a quick version of the guide (many of you might already read about it), read my post on medium, "[The simple guide to deploy Laravel 5 application on shared hosting](https://medium.com/laravel-news/the-simple-guide-to-deploy-laravel-5-application-on-shared-hosting-1a8d0aee923e#.7y3pk6wrm)"

## Requirements

Before trying to deploy a Laravel/Lumen application on a shared hosting, you need to make sure that the hosting services provide a fit [requirement to Laravel](https://laravel.com/docs/5.2#server-requirements). Basically, they are the following items:

* PHP >= 5.5.9
* OpenSSL PHP Extension
* PDO PHP Extension
* Mbstring PHP Extension
* Tokenizer PHP Extension

Next, you need to have the SSH access permission for your hosting account; otherwise, none of these will work.

Besides PHP and those required extensions, you might need some utilities to make deployment much easier.

* [Git](https://git-scm.com/)
* [Composer](https://getcomposer.org/)

I guess that's enough for good. Please refer to below sections to learn more about deployment.

## Instruction

TBU.

## FAQs

> **1. How to acquire SSH access permission for my account?**

Just contact your hosting support, they will need to confirm your identity and will permit your SSH access in no time.

> **2. Where is git? I can't find it.**

`git` is often put under this place in CPanel hosting services, `/usr/local/cpanel/3rdparty/bin/git`. So you need to provide full path to `git` if you want to issue a `git` command; or, you can also create an alias for convenient use,

```
alias git="/usr/local/cpanel/3rdparty/bin/git"
```

> **3. How to get composer?**

You can use FTP or SCP command to upload `composer.phar` to the host after downloading it locally. Or use `wget` and `curl` to get the file directly on host,

```
$ wget https://getcomposer.org/composer.phar
```


## List of service providers tested and worked

The following shared hosting service providers have been tested and worked perfectly 100%.

* [NameCheap](https://www.namecheap.com/)
* [JustHost](https://www.justhost.com/)
* [bluehost](https://www.bluehost.com/)
* [GoDaddy](https://godaddy.com/)
* [HostGator](http://www.hostgator.com/)

If you found any hosting providers that works, please tell me, I will update the list for others to know about them, too.

## Still trouble?

If you still fail to deploy Laravel/Lumen applications after following all above steps. Provide me [your issue](https://github.com/petehouston/laravel-deploy-on-shared-hosting/issues) in details, I will help you out.

## Contribution Guide

Free free to fork the project and submit [a pull request](https://github.com/petehouston/laravel-deploy-on-shared-hosting/pulls).