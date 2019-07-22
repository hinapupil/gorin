# Redis
```
$apt install redis-server
$redis-cli set skills:index "Today is the 2019-01-01 and current time is 00:00"
```
~/today.sh
```
#!/bin/bash
today=$(date "+%Y-%m-%d")
curtime=$(date "+%H:%M")
redis-cli set skills:index "Today is the ${today} and current time is ${curtime}"
```
