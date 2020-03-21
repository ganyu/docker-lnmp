# docker-lnmp
centos7下基于docker-compose搭建的lnmp环境，本环境搭建过程使用的是yii2进行相关测试，所以默认就支持yii2的运行

LNMP（Docker + Docker-compose + Nginx + MySQL5.7 + PHP7.2 + Redis5.0 + Memcached1.5 + Mongodb4.2）

LNMP项目特点：
1. `100%`开源，不含脚本运行，易学实用
2. `100%`遵循Docker标准
3. 支持数据文件、配置文件、日志文件挂载
4. 默认支持`pdo_mysql`、`mysqli`、`swoole`、`gd`、`curl`、`opcache`等常用扩展
5. 包含基本的已优化的配置文件

## 1.目录结构

```
├── data                        数据库数据目录
│   ├── mongo                   MongoDB 数据目录
│   ├── mysql                   MySQL 数据目录
│   └── redis                   Redis 数据目录
├── services                    服务构建文件、配置文件目录
│   ├── mysql                   MySQL8 配置、构建文件
│   ├── nginx                   Nginx 配置、构建文件
│   ├── php                     PHP 配置、构建文件
│   ├── memcached               Memcached 配置、构建文件
│   ├── mongo                   Mongo 配置、构建文件
│   └── redis                   Redis 配置、构建文件
├── logs                        日志目录
│   └── mysql                   
│   └── nginx                   
│   └── php                   
│   └── memcached                   
│   └── mongo                   
│   └── redis                   
├── docker-compose.yml         Docker 服务配置示例文件
├── .env                       环境配置
└── www                        PHP 代码目录 可在.env中nginx的WEB_DIR中任意指定
```

## 2.快速使用
1. 本地安装
    1. docker、docker-compose
    2. 阿里云docker加速：https://help.aliyun.com/document_detail/60750.html

2. clone项目：
    $ `git clone https://github.com/nianzhi1202/docker-lnmp.git`

3. 如果不是root用户，还需将当前用户加入docker用户组：
    $ `sudo gpasswd -a ${USER} docker`

4. 可根据项目需要自行添加其它php扩展 

5. 启动本项目
$ `docker-compose up --build --force-recreate`  #可以加 -d 后台运行，调试时不用加，方便查看日志


## 3.docker常用命令
- $ `systemctl start docker`    # 启动docker
- $ `docker start containername` # 启动容器
- $ `docker stop containername` # 停止容器
- $ `docker exec -it 13a676bb9bff /bin/bash` # 进入容器
- $ `docker rm containername` # 删除容器
- $ `docker rmi image` # 删除镜像

- $ `docker-compose up` # 启动所有容器 $ docker-compose up nginx php mysql # 启动指定容器
- $ `docker-compose up -d nginx php mysql` # 后台运行方式启动指定容器 
- $ `docker-compose up --build --force-recreate` # 强制启动
- $ `docker-compose start php` # 启动服务
- $ `docker-compose stop php` # 停止服务
- $ `docker-compose restart php` # 重启服务
- $ `docker-compose build php` # 使用Dockerfile构建服务

## 4.mongo基本操作
+ 命令行连接mongo
    + $ `docker exec -it 容器 /bin/bash`
    + $ `mongo` 即可操作mongo命令行
+ 安装时已开启验证，账号在.env中配置
    + 选择数据库（admin数据库是默认创建的）：$ `use admin`
    + 输入验证：`db.auth("root","123456")`
+ php常用的mongo客户端有两个：mongo和mongodb，这里使用mongodb
    + php中查询测试：
    ```
        <?php
        	$manager = new MongoDB\Driver\Manager("mongodb://root:123456@mongo:27017");  
        	// 插入数据
        	$bulk = new MongoDB\Driver\BulkWrite;
        	$bulk->insert(['x' => 1, 'name'=>'测试1', 'url' => 'http://www.baidu.com']);
        	$bulk->insert(['x' => 2, 'name'=>'测试2', 'url' => 'http://www.google.com']);
        	$bulk->insert(['x' => 3, 'name'=>'测试3', 'url' => 'http://www.taobao.com']);
        	$manager->executeBulkWrite('test.sites', $bulk);
        	$filter = ['x' => ['$gt' => 1]];
        	$options = [
        	    'projection' => ['_id' => 0],
        	    'sort' => ['x' => -1],
        	];
        	// 查询数据
        	$query = new MongoDB\Driver\Query($filter, $options);
        	$cursor = $manager->executeQuery('test.sites', $query);
        	foreach ($cursor as $document) {
        	    print_r($document);
        	}
        ?>
    ```
    
## 5.nginx配置说明
+ 默认的nginx配置已支持yii2的运行
+ 如项目放在 /var/www/order
    + .env中，WEB_DIR=/var/www/order
    + nginx.conf中， root /var/www/order/backend/web;

## 6.memcached使用
1. mem常用的php客户端有两个：memcached和memcache，这里使用memcached
2. yii2默认支持memcached作为缓存系统，只需如下配置：在common/config/main-local.php中
    ```
   'cache' => [
            'class' => 'yii\caching\MemCache',
            'servers' => [
                [
                    'host' => 'memcached',
                    'port' => 11211
                ],
            ],
            'useMemcached'=>true,
            'keyPrefix'=>'yikong'
    ],
   ```
    
3. 其他框架的环境正常连接即可，只需注意host是容器的名称，不能使用ip


## 7.redis使用
1. 使用predis客户端，直接在项目中`composer require predis/predis`安装即可
2. 连接测试，需注意host是容器名称
   ```
   <?php
   namespace backend\controllers;
   use Predis\Autoloader;
   use Predis\Client;
   use yii\web\Controller;
   
   class TestController extends Controller
   {
       public function actionIndex()
       {
           $client =new Client([
               'scheme' => 'tcp',
               'host'   => 'redis',
               'port'   => 6379,
           ]);
           var_dump($client);
           $client->set('foo', 'bar5');
           echo $value = $client->get('foo');
       }
   }
   ```


    

