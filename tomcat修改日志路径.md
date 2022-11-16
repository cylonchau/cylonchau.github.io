# tomcat修改日志目录


### 修改logging.properties

```conf
/usr/local/apache-tomcat-8.5.32/conf/logging.properties
```


```bash
############################################################
# Handler specific properties.
# Describes specific configuration info for Handlers.
############################################################

1catalina.org.apache.juli.AsyncFileHandler.level = FINE
1catalina.org.apache.juli.AsyncFileHandler.directory = /data/logs
1catalina.org.apache.juli.AsyncFileHandler.prefix = catalina.

2localhost.org.apache.juli.AsyncFileHandler.level = FINE
2localhost.org.apache.juli.AsyncFileHandler.directory = /data/logs
2localhost.org.apache.juli.AsyncFileHandler.prefix = localhost.

3manager.org.apache.juli.AsyncFileHandler.level = FINE
3manager.org.apache.juli.AsyncFileHandler.directory = /data/logs
3manager.org.apache.juli.AsyncFileHandler.prefix = manager.

4host-manager.org.apache.juli.AsyncFileHandler.level = FINE
4host-manager.org.apache.juli.AsyncFileHandler.directory = /data/logs
4host-manager.org.apache.juli.AsyncFileHandler.prefix = host-manager.

java.util.logging.ConsoleHandler.level = FINE
java.util.logging.ConsoleHandler.formatter = org.apache.juli.OneLineFormatter
```

***替换命令***

```bash
:%s#${catalina.base}/logs#/data/logs#g
sed -i s#${catalina.base}/logs#/data/logs#g  /usr/local/apache-tomcat-8.5.32/conf/logging.properties
```

### 修改catalina.sh

```bash
CATALINA_OUT=/data/logs/catalina.out
JAVA_HOME=/usr/java/latest
JAVA_OPTS="${JAVA_OPTS} -Xms4096m -Xmx4096m -Xmn1024m -Xss1024K -XX:PermSize=4096m -XX:MaxPermSize=4096m"
```

***增加如下：***

```
CATALINA_OUT=/data/logs/catalina.out
```

```bash
org.apache.catalina.startup.Bootstrap "$@" start  2>&1 \
      | /usr/sbin/cronolog /data/logs/catalina.%Y-%m-%d.out >> /dev/null &
```
