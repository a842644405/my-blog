# Docker部署时Mysql导入初始数据乱码问题

## 前言



## Docker MySQL启动时自动执行初始建表脚本

通过查找资料，了解到：在mysql官方镜像中提供了容器启动时自动执行`/docker-entrypoint-initdb.d`文件夹下的脚本的功能(包括shell脚本和sql脚本) `docker-entrypoint.sh`中下面这段代码就是干这事儿的。

```
ls /docker-entrypoint-initdb.d/ > /dev/null
for f in /docker-entrypoint-initdb.d/*; do
	process_init_file "$f" "${mysql[@]}"
done

# usage: process_init_file FILENAME MYSQLCOMMAND...
#    ie: process_init_file foo.sh mysql -uroot
# (process a single initializer file, based on its extension. we define this
# function here, so that initializer scripts (*.sh) can use the same logic,
# potentially recursively, or override the logic used in subsequent calls)
process_init_file() {
	local f="$1"; shift
	local mysql=( "$@" )

	case "$f" in
		*.sh)     echo "$0: running $f"; . "$f" ;;
		*.sql)    echo "$0: running $f"; "${mysql[@]}" < "$f"; echo ;;
		*.sql.gz) echo "$0: running $f"; gunzip -c "$f" | "${mysql[@]}"; echo ;;
		*)        echo "$0: ignoring $f" ;;
	esac
	echo
}
```

也就是说只要把自己的初始化脚本放到`/docker-entrypoint-initdb.d/`文件夹下就可以了，由于在docker-compose中没有找到copy指令，所以我用volumes代替，然后将MySql部分改成了这样：

```
version: '3.8'
services:
  mysql-service:
    image: mysql:8.0.33
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: root
      MYSQL_DATABASE: ry-vue
      CHARACTER_SET_SERVER: utf8mb4
      COLLATION_SERVER: utf8mb4_unicode_ci
    ports:
      - "3306:3306"
    volumes:
      - /opt/project/ruoyi/mysql-service/volumes/mysql:/var/lib/mysql
      - /opt/project/ruoyi/mysql-service/other/init:/docker-entrypoint-initdb.d/
      - /opt/project/ruoyi/mysql-service/other/children-sql:/opt/mysql-service/
      - /home/dockerdata/mysql/conf.d:/etc/mysql/conf.d
      - /home/dockerdata/mysql/log:/var/log/mysql
      - /etc/localtime:/etc/localtime:ro
# init目录下的init.sql
#init.sql
#SET NAMES 'utf8';
#source /opt/mysql-service/create-database-user.sql;
#source /opt/mysql-service/quartz.sql;
#source /opt/mysql-service/ry_20231130.sql;

    networks:
      - ruoyi-network

  redis-service:
    image: redis:7.2
    restart: always
    ports:
      - "6379:6379"
    volumes:
      - /opt/project/ruoyi/redis-service/volumes/redis.conf:/usr/local/etc/redis/redis.conf
    command: [ "redis-server", "/usr/local/etc/redis/redis.conf" ]
    networks:
      - ruoyi-network

  ruoyi-backend-service:
    image: ruoyi-backend:1.0
    networks:
      - ruoyi-network
    ports:
      - "8080:8080"
    depends_on:
      - mysql-service
      - redis-service

  ruoyi-frontend-service:
    image: ruoyi-front:1.0
    networks:
      - ruoyi-network
    ports:
      - "80:80"
    depends_on:
      - ruoyi-backend-service

networks:
  ruoyi-network:
    driver: bridge


```

## MySQL导入初始数据乱码问题

在创建mysql容器的时候，成功的初始化了数据库，但却有了一个新问题：数据库中的中文都是乱码的。

接着就开始找中文乱码的解决办法。网上的方法大概有两种：

- 通过在命令中加入`character-set-server`和`collation-server`两个选项来指定mysql的编码。

```
--character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci
```

- 挂载`my.cnf`，在`my.cnf`文件中设置字符编码

```
[mysqld]
character_set_server=utf8
```

这两种方式我都尝试了，但是，全都没有效果。

通过`show variables like 'character%'`命令查看时，mysql默认字符集确实已改为了`utf8`或`utf8mb4`，但是执行脚本导入的数据依然是乱码。

这个乱码问题困扰了我很久，一直不知道原因，后来我试着先创建mysql镜像，然后手动导入SQL，用Navicat for MySQL执行sql完全没问题，但是，我在docker容器中以命令行的方式导入，却是乱码的。

```mysql
mysql -uroot -p123456 blog < blog.sql
```

后来我在自己的windows系统上使用命令行的方式导入SQL，发现出现了同样的问题。我这里用的SQL文件是通过Navicat for MySQL导出的，接着我就把SQL文件换成了通过命令行的方式导出。

```mysql
mysqldump -uroot -p123456 blog > blog.sql
```

然后重新构建docker，发现不再乱码。

## 后续

问题虽然成功解决了，但是没找到真正的原因有点不甘心，对比两个SQL文件，最大的不同就是，通过mysqldump导出的的脚本中多了许多`/*! */;`这种语句，而这种语句是一种特殊的注释，其他的数据库产品不会执行。mysql特殊处理，会选择性的执行。可以认为是：预编译中的条件编译。所以问题应该就出在这上面。

后面在[Mysql字符集设置](https://www.iszhouhua.com/mysql-character-set-setting.html)中看到了这么一句话：

> 设置了表的默认字符集为utf8并且通过UTF-8编码发送查询，存入数据库的仍然是乱码。那connection连接层上可能出了问题。

解决方法是在发送查询前执行一下下面这句：`SET NAMES 'utf8'`;它相当于下面的三句指令：

```mysql
SET character_set_client = utf8;
SET character_set_results = utf8;
SET character_set_connection = utf8;
```

于是，我在Navicat for MySQL导出的sql文件的开头，加上了`SET NAMES 'utf8';`，重新构建docker，确实不再乱码。看来问题应该就是出在这里，而且mysqldump导出的SQL中也有这么一句`/*!40101 SET NAMES utf8 */;`，我将该语句删除后，mysqldump导出的SQL也出现了乱码问题！



