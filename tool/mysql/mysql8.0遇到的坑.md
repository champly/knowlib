# mysql配置遇到的坑

## 版本 

> - mysql 8.0.13

## 启动

``` bash
docker run --name mysql -p 3306:3306 -v /Users/champly/go/data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=123456 -d mysql
```

## 问题一

``` bash
ERROR 2059 (HY000): Authentication plugin 'caching_sha2_password' cannot be loaded: dlopen(/usr/local/mysql/lib/plugin/caching_sha2_password.so, 2): image not found 
```

## 解决

``` bash
ALTER USER 'root'@'%' IDENTIFIED WITH mysql_native_password BY 'root';
```

**原因**:当前版本mysql的加密方式不是mysql_native_password，修改为mysql_native_password即可

## 问题二

``` bash
ERROR 1045 (28000): Access denied for user 'root'@'172.17.0.1' (using password: YES)
```

## 解决：

``` bash
alter user 'root'@'%' identified by '123456';
```

**原因** 修改了加密方式之后要把密码修改回去

然后就可以连接了
