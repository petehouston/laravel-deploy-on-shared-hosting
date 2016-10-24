# Phương pháp triển khai ứng dụng Laravel trên shared hosting

Hướng dẫn đơn giản để triển khai ứng dụng Laravel và Lumen trên shared hosting.

Bạn có thể xem bản hướng dẫn nhanh (chắc hẳn nhiều trong số bạn đã từng đọc qua), trên medium, "[The simple guide to deploy Laravel 5 application on shared hosting](https://medium.com/laravel-news/the-simple-guide-to-deploy-laravel-5-application-on-shared-hosting-1a8d0aee923e#.7y3pk6wrm)"

## Yêu cầu

Before trying to deploy a Laravel application on a shared hosting, you need to make sure that the hosting services provide a fit [requirement to Laravel](https://laravel.com/docs/5.2#server-requirements). Basically, following items are required for Laravel 5.2:
Trước khi thực hiện triển khai ứng dụng Laravel trên shared hosting, bạn cần đảm bảo nhà cung cấp dịch vụ cho bạn môi trường với [các yêu cầu cần thiết cho Laravel](https://laravel.com/docs/5.2#server-requirements). Về cơ bản, yêu cầu cho Laravel 5.2 như sau:

* PHP >= 5.5.9
* OpenSSL PHP Extension
* PDO PHP Extension
* Mbstring PHP Extension
* Tokenizer PHP Extension

Tuy nhiên, yêu cầu này còn phụ thuộc vào phiên bản của Laravel bạn muốn sử dụng, hãy đọc tài liệu về [yêu cầu tương ứng của từng phiên bản Laravel](https://laravel.com/docs/master).

**Tiếp đến, bạn cần có quyền truy cập vào SSH tới tài khoản hosting của bạn; nếu không mọi thứ ở đây sẽ trở nên vô nghĩa.**

Ngoài các yêu cầu về phiên bản PHP và các extensions cần thiết, bạn cần có thêm một vài công cụ để cho việc triển khai được dễ dàng hơn.

* [Git](https://git-scm.com/)
* [Composer](https://getcomposer.org/)

Theo tôi như vậy là đủ. Hãy xem các phần bên dưới để tìm hiểu phương thức triển khai.

## Hướng dẫn

Cùng bắt đầu bằng việc tìm hiểu về cách chúng ta nên quản lý cấu trúc thư mục của ứng dụng Laravel. Đầu tiên, cùng nhìn xem danh sách các files và thư mục bạn có trên tài khoản.

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

Với tài khoản chính được gắn với domain chính của bạn, phần front-end sẽ phải nằm trong thư mục `public_html` hoặc `www`. Vì chúng ta không muốn để lộ các *thông tin nhạy cảm của Laravel* (ví như các file .env, v..v..) ra bên ngoài, chúng ta sẽ giấu chúng đi.

Tạo một thư mục mới để chứa toàn bộ code, đặt tên là `projects` hay bất cứ tên nào bạn muốn.

```bash
$ mkdir projects
$ cd projects
```

Từ đây, chúng ta sẽ sử dụng câu lệnh git để lấy code về,

```bash
$ git clone http://[GIT_SERVER]/awesome-app.git
$ cd awesome-app
```

Bước tiếp theo là làm cho thư mục `awesome-app/public` được tham chiếu tới `www`, symbol link sẽ hỗ trợ chúng ta việc này, nhưng chúng ta cần backup thư mục `public` trước đã.

```bash
$ mv public public_bak
$ ln -s ~/www public
$ cp -a public_bak/*.* public/
$ cp public_bak/.htaccess public/
```

Phần khoai nhất đã xong, phần còn lại sẽ là các bước cơ bản để thiết lập Laravel. Cấp quyền ghi cho thư mục `storage` là một việc quan trọng,

```bash
$ chmod -R o+w storage
```

**Hãy chỉnh cấu hình trong file `.env`. Đừng bỏ quên điều này!**

Cuối cùng, hãy cập nhật các packages cần thiết cho project Laravel sử dụng **composer** và thêm các cache cần thiết,

```bash
$ php composer install
$ php composer dumpautoload -o
$ php artisan config:cache
$ php artisan route:cache
```

**Chúc mừng! Bạn đã triển khai thành công một ứng dụng Laravel trên một dịch vụ shared hosting..**

## FAQs

> **1. Làm thế nào để lấy được quyền sử dụng SSH cho tài khoản của tôi?**

Hãy liên hệ trực tiếp với bên hỗ trợ của dịch vụ bạn sử dụng, họ sẽ cần xác nhận thông tin của bạn và sẽ cho bạn quyền sử dụng SSH một cách nhanh chóng.

> **2. Git nằm ở đâu? Tôi không tìm thấy.**

`git` thường được đặt ở vị trí này trong các dịch vụ hosting sử dụng CPanel, `/usr/local/cpanel/3rdparty/bin/git`. Vì vậy, bạn cần phải gõ đường dẫn đầy đủ tới `git` nếu bạn muốn thực thi một câu lệnh; hoặc là bạn của thể tạo một alias cho tiện.

```bash
alias git="/usr/local/cpanel/3rdparty/bin/git"
```

> **3. Làm thế nào để lấy composer?**

Bạn có thể sử dụng FTP hay SCP để upload file `composer.phar` lên host sau khi download trên máy cá nhân. Hoặc cũng có thể sử dụng `wget` hay `curl` để download file trực tiếp về host.

```bash
$ wget https://getcomposer.org/composer.phar

hoặc

$ curl -sS https://getcomposer.org/installer | php — –filename=composer
```

> **4. Cách này có thực hiện được cho Lumen hay không?**

Về cơ bản thì Laravel và Lumen như là anh em sinh đôi, vì vậy có thể áp dụng được với Lumen.

> **5. Tôi thử chạy `composer` nhưng mà không thấy hiển thị gì cả. Vấn đề ở chỗ nào vậy?**

Bạn cần cung cấp đường dẫn tới file cầu hình PHP để thực thi `composer`, nghĩa là, bạn không thể thực thi `composer` trực tiếp trên host. Vì thể để thực thi `composer`, bạn sẽ phải thực hiện như câu lệnh sau,

```bash
$ php -c php.ini composer [COMMAND]
```

> **6. Tôi có thể lấy `php.ini` ở đâu để thực thi `composer`?**

Bạn có thể copy file cấu hình `php.ini` mặc định, thường nằm tại `/usr/local/lib/php.ini`, hoặc có thể sử dụng câu lệnh sau để tìm kiếm,

```bash
$ php -i | grep "php.ini"
```

## Danh sách các nhà cung cấp dịch vụ đã được thực hiện kiểm tra và hoạt động

Các dịch vụ dưới đây đã được thực hiện kiểm tra và hoạt động 100%.

* [NameCheap](https://www.namecheap.com/)
* [JustHost](https://www.justhost.com/)
* [bluehost](https://www.bluehost.com/)
* [GoDaddy](https://godaddy.com/)
* [HostGator](http://www.hostgator.com/)

Nếu như bạn tìm thấy nhà cung cấp dịch vụ nào mà cũng hoạt động, hãy chia sẻ, tôi sẽ cập nhật để cho nhiều người khác cùng biết tới.

## Vẫn gặp vấn đề?

Nếu như bạn vẫn thất bại trong việc triển khai ứng dụng Laravel sau khi thực hiện đầy đủ các bước ở trên. Hãy cung cấp cho tôi [nội dung vấn đề của bạn](https://github.com/petehouston/laravel-deploy-on-shared-hosting/issues) một cách chi tiết, tôi sẽ cố gắng để giúp bạn.

## Hướng dẫn đóng góp

Hãy thực hiện fork và gửi một [pull request](https://github.com/petehouston/laravel-deploy-on-shared-hosting/pulls).
