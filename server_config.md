# Redis
```
$apt install redis-server
$redis-cli set skills:index "Today is the 2019-01-01 and current time is 00:00"
```

~/today.sh
```bash:today:sh
#!/bin/bash
today=$(date "+%Y-%m-%d")
curtime=$(date "+%H:%M")
redis-cli set skills:index "Today is the ${today} and current time is ${curtime}"
```

# Web service
```
$apt install apache2 php php-redis
```

/etc/apache2/mods-enabled/dir.conf  
2nd line: Set the file name accessible only by the directory name
  ```
  DirectoryIndex index.php
  ```
  
/var/www/html/index.php

```php:index.php
<?php
 echo ’<h1>’, gethostname(), ’</h1>’;
?>
```

```
$mkdir /var/www/html/redis
```

/var/www/html/redis/index.php

```php:index.php
<?php
 echo ’<h1>’, gethostname(), ’</h1>’;
 $redis=new Redis();
 $redis->connect(’192.168.101.1’);
 $content=$redis->get(’skills:index’);
 echo $content;
?>
```

# Web contents sync
```
$apt install rsync
$vi /root/web_sync.sh
rsync -auvz -delete /var/www www2@20.0.0.4:/var/www
```
