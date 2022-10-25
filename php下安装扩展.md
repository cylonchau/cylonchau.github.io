# PHP安装扩展


这里以mysqli为例子

```bash
cd /apps/php/ext/mysqli

./configure –prefix=/usr/local/mysqli –with-php-config=/usr/local/php5/bin/php-config –with-mysqli=/usr/local/mysql/bin/mysql_config

make

make test

make install
# 修改php.ini
extension_dir="/usr/local/php/lib/php/extensions/no-debug-non-zts-20060613/"
extension=mysqli.so
```

### 错误处理

```bash
Make sure that you run '/apps/php-5.5/bin/phpize' in the top level source directory of the module
```

解决方法：在需要扩展编译的PHP模块目录中进行 `/usr/local/php/bin/phpize` 这样才不会报错
