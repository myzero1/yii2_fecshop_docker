Fecshop Docker
=============


> 用于快速的，使用docker搭建fecshop的环境，方便快速部署，通过docker compopse
> 通过下载镜像，自己构建镜像，把fecshop需要的各个软件以及扩展都安装好，您
> 可以根据下面的说明操作



目录结构介绍
---------


`./app`: 这里是代码文件，fecshop的代码文件放到这里

`./db`: 这里是环境部分-数据库部分

`./db/mongodb`: 这里是mongodb数据库的部分 

`./db/mongodb/data`: 这里是数据库的数据存放的部分，也就是数据库的库表部分数据。 


`./db/mongodb/etc/mongod.conf`: Mongodb数据库的配置文件 


`./db/mongodb/logs`: Mongodb的logs部分 


`./db/mysql`: mysql数据库

`./db/mysql/data`: mysql 数据库表数据存放的位置

`./db/mysql/conf.d`: mysql 配置文件
 
`./services`: 服务软件部分，譬如php nginx等

`./services/php`: php部分

`./services/php/docker/Dockerfile`: php镜像构建的dockerfile文件

`./services/php/etc/php7.1.13.ini`: php的配置文件

`./services/web`: nginx部分

`./services/web/nginx/conf`: nginx的配置部分

`./services/web/nginx/conf/conf.d/default.conf`：nginx 网站 server 部分的配置文件

`./services/web/nginx/logs`: nginx的log日志文件部分

`./docker-compose.yml`: docker compose配置文件




安装docker和docker compose
-------------------------

linux内核需要大约3.1.0 ,下面是centos 7 下面部署的过程：


1、安装docker

```
sudo curl -sSL https://get.daocloud.io/docker | sh
```

2、安装 docker compose，资料：[install-compose](https://docs.docker.com/compose/install/#install-compose)

```
sudo curl -L https://github.com/docker/compose/releases/download/1.18.0/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
```



### docker compose 安装部署环境

下载当前库文件，通过`git clone`下载：

```
git clone https://github.com/fecshop/yii2_fecshop_docker.git
```

1.进入上面下载完成后的文件夹 yii2_fecshop_docker，打开 `docker-compose.yml`

1.1更改mysql的密码：
```
- MYSQL_ROOT_PASSWORD=fecshopxfd3ffaads123456
```

1.2更改redis的密码：

```
打开文件：`./db/redis/etc/redis-password`,更改里面的redis密码即可。
```

mysql和redis的密码要记住，后面配置要用到。


2.构建：

> 第一次构建需要下载环境，时间会比较长，除了下载docker中心的镜像，还要构建镜像
> 看网速，如果用阿里云，15分钟差不多完成，使用下面的命令构建环境

```
docker-compose build
```

完成后，运行：

```
docker-compose up  // 按下ctrl+c退出停止。
```

后台运行：（守护进程的方式）

```
docker-compose up -d
```

查看compose启动的各个容器的状态：

```
docker-compose ps
```

进入某个容器,譬如php：

```
docker-compose exec php bash
```

退出某个容器

```
exit
```


停止 docker compose启动的容器：

```
docker-compose stop
```

到这里我们的环境就安装好了，也讲述了一些docker compose常用的命令，
下面我们测试一下我们的环境


### 启动docker ，下载安装fecshop



> 对于docker ，一定要切记，docker不是虚拟机！docker不是虚拟机！docker不是虚拟机！
> 每一个服务，对应一个docker 容器，譬如mysql
> 一个容器，php一个容器，redis一个容器，mongdob一个容器，
> 每一个容器的数据和配置文件都是在宿主主机上面，通过`volumes`
> 挂载到容器的相应文件夹中，（我们在`./docker-compose.yml`
> 配置文件中的`volumes`做了映射）
> 
> 因此，对于docker 容器，里面涉及到存储的部分，都应该通过
> 挂载的方式映射到宿主机上面，而不是在容器里面。

`宿主机`: 就是您的linux主机

`容器主机`：就是docker容器虚拟的主机。


1、启动: 

进入yii2_fecshop_docker目录，执行：

`docker-compose up -d`



2、composer 安装fecshop

我们通过命令进入到php的容器：

```
docker-compose exec php  bash
// 进入成功后，在php容器中执行：
cd /www/web
// 将`1.3.0.3` 替换成相应的fecshop版本。下面提示需要token，参看这里获取Token：http://www.fecshop.com/topic/412
composer create-project fancyecommerce/fecshop-app-advanced  fecshop 1.3.0.3
   
```

2.1.2 yii2_mongodb扩展bug的处理（官方还未发布新版本）

另开一个xshell窗口，在宿主主机  ./app/fecshop中打开composer.json，在require中加入
`"yiisoft/yii2-mongodb": "dev-master"`, 如下：

```
"require": {
        "php": ">=5.4.0",
        "yiisoft/yii2-mongodb": "dev-master",  // 将前面的这个配置加入进去即可。
        
        ...
    },
```
保存退出
，
回到3.1部分的xshell窗口，执行：

```
cd fecshop
composer update
./init
```
执行完后，通过composer加载的文件就完成了。


> 参考资料：[Fecshop 安装](http://www.fecshop.com/doc/fecshop-guide/develop/cn-1.0/guide-fecshop-about-hand-install.html)

2.2 百度云盘完整版

> 通过百度网盘安装(不建议),如果因为墙无法使用composer，可以访问百度云盘，
> 下载地址为：http://pan.baidu.com/s/1hs1iC2C 下载日期最新的压缩包即可

如果您使用的是百度云盘完整版，
那么将文件解压到宿主机 `./app/` 下面即可，将文件夹的名字改成`fecshop`
，完成后  `./app/fecshop` 就是fecshop系统包的根目录

```
cd fecshop   
./init
```

> 参考资料：[Fecshop 安装](http://www.fecshop.com/doc/fecshop-guide/develop/cn-1.0/guide-fecshop-about-hand-install.html)



### 配置fecshop


> 参考：[Fecshop 初始配置](http://www.fecshop.com/doc/fecshop-guide/develop/cn-1.0/guide-fecshop-about-config.html)

> 为了更方便的配置，Terry在 `./example_data/` 中已经进行了一些默认配置，
> 您可以使用默认配置先搭建起来，然后在按照自己的需要进行更改。
> 下面介绍的是在`./example_data/`里面的各个配置和其他的一些东西，
> 您可以进入`./example_data/`文件件，
> 将默认的配置覆盖到fecshop中。



1、本机（浏览器所在的电脑，也就是您的window本机），添加host(打开C:\Windows\System32\drivers\etc\hosts，添加如下代码,如果是其他IP，将 127.0.0.1 替换成其他IP即可。)

```
127.0.0.1       my.fecshop.com       # mysql的phpmyadmin的域名指向
127.0.0.1       appadmin.fecshop.com # 后台域名指向
127.0.0.1       appfront.fecshop.com # 前台pc端域名指向
127.0.0.1       appfront.fecshop.es  # 前台pc端 es 语言的域名指向
127.0.0.1       apphtml5.fecshop.com # 前台html端的域名指向
127.0.0.1       appapi.fecshop.com   # api端的域名指向
127.0.0.1       appserver.fecshop.com # server端的域名指向
127.0.0.1       img.fecshop.com		#appimage/common   图片的域名指向
127.0.0.1       img2.fecshop.com	#appimage/appadmin 图片的域名指向
127.0.0.1       img3.fecshop.com	#appimage/appfront 图片的域名指向
127.0.0.1       img4.fecshop.com	#appimage/apphtml5 图片的域名指向
127.0.0.1       img5.fecshop.com	#appimage/appserver图片的域名指向
```



2.更改配置文件

数据库配置：

打开 ./example_data/fecshop/common/config/main-local.php
,将mysql的密码，redis的密码，以及redis在session cache中使用的密码，都配置一下，
密码使用上面进设置的密码。

3.配置图片部分的域名（**使用默认即可，如果您要自定义域名，需要修改**）

文件：`./app/fecshop/common/config/fecshop_local_services/Image.php`


4、nginx做路径指向设置

配置文件为`./services/web/nginx/conf/conf.d/default.conf`
（**已经配置好域名部分使用默认即可，如果您要自定义域名，需要修改**）

5、Store的配置（**使用默认即可，如果您要自定义域名，需要修改**）

`./example/fecshop/` 下三个入口的store配置

```
@appfront/config/fecshop_local_services/Store.php 

@apphtml5/config/fecshop_local_services/Store.php 

@appserver/config/fecshop_local_services/Store.php 
```


6.例子数据修改完成后，复制到fecshop文件夹中

进入`./example_data/`文件，执行：

```
// 复制配置文件，也就是下面的各个store 域名 以及数据库配置
\cp -rf ./fecshop/* ../app/fecshop/
unzip -o ./example_img_and_db_data/appimage.zip  -d  ../app/fecshop/
```




7.创建mysql数据库

7.1在根目录（./yii2_fecshop_docker）下执行，进入mysql的容器

```
docker-compose exec mysql bash
```

执行`mysql -uroot -p` 进入mysql

```
use fecshop;
create database fecshop;
show databases;
exit;
```

exit，退出容器

7.2 Yii2 migratge方式导入表结构。


mysql(导入mysql的表，数据，索引):

```
./yii migrate --interactive=0 --migrationPath=@fecshop/migrations/mysqldb
```

mongodb(导入mongodb的表，数据，索引):

```
./yii mongodb-migrate  --interactive=0 --migrationPath=@fecshop/migrations/mongodb
```

8.测试数据

8.1安装mongodb数据库的测试数据

在根目录下（github下载完成后的文件夹下）进入mongodb容器

```
docker-compose exec mongodb bash
```

```
mongo mongodb:27017/fecshop --quiet /data/example_db/mongo-fecshop_test-20170419-065157.js
```

执行exit退出容器

8.2安装mysql数据库的测试数据

在根目录(docker-compose.yml文件所在目录)下执行，进入mysql的容器

```
docker-compose exec mysql bash
```

执行`mysql -uroot -p` 进入mysql

```
use fecshop;
source /var/example_db/mysql_fecshop.sql
exit
```

执行exit退出容器

9.初始化搜索引擎数据

在根目录(docker-compose.yml文件所在目录)下执行，进入php的容器

```
docker-compose exec php bash
cd /www/web/fecshop/vendor/fancyecommerce/fecshop/shell/search
sh fullSearchSync.sh
```


10.后台的默认用户名密码

```
admin
admin123
```


### 安装VUE部分


1.进入 yii2_fecshop_docker 
，参看文档：https://github.com/fecshop/vue_fecshop_appserver
，进行安装


2.操作到第4步后，进行下面的配置


2.1.src/config/store.js

将 `demo.fancyecommerce.com` 改成 `vue.fecshop.com`

2.2.config/prod.env.js

将 

```
module.exports = {
  NODE_ENV: '"production"',
  API_ROOT: '"//fecshop.appserver.fancyecommerce.com"',
  WEBSITE_ROOT: '"http://demo.fancyecommerce.com"'
}
```

改成：

```
module.exports = {
  NODE_ENV: '"production"',
  API_ROOT: '"//appserver.fecshop.com"',
  WEBSITE_ROOT: '"http://vue.fecshop.com"'
}
```

2.3.config/dev.env.js

将 

```
module.exports = merge(prodEnv, {
  NODE_ENV: '"development"',
  API_ROOT: '"//fecshop.appserver.fancyecommerce.com"',
  WEBSITE_ROOT: '"//demo.fancyecommerce.com"'
})
```

改成：

```
module.exports = merge(prodEnv, {
  NODE_ENV: '"development"',
  API_ROOT: '"//appserver.fecshop.com"',
  WEBSITE_ROOT: '"http://vue.fecshop.com"'
})
```



2.4执行`npm run build`

2.5访问：http://vue.fecshop.com







