# windows递归复制指定时间后修改过的文件


因为在拷贝web站点时，也会存在更新，需要定期覆盖新的内容，就是上次覆盖的时间和到这次时间内修改过的文件都复制。

实现命令xcopy

```
xcopy src dest
$ xcopy D:\WWW\back1\* D:\WWW\back4  /D:05-22-2018 /F /E /y
D:\WWW\back1\db_qbe.php -> D:\WWW\back4\db_qbe.php
D:\WWW\back1\docs.css -> D:\WWW\back4\docs.css
D:\WWW\back1\test\changelog.php -> D:\WWW\back4\test\changelog.php
复制了 3 个文件
```

`/D:mm-dd-yyyy`

`/F` 打印复制过程

`/E` 递归复制目录和子目录包括空目录

`/Y` 禁止提示
