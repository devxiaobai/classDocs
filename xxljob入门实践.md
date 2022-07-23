# xxljob入门实践

## 常见任务调度方案

Timer

ExectorService

Spring @scheduled

quartz

xxljob （https://github.com/xuxueli/xxl-job）

elastic-job



## 简介

中文文档：https://www.xuxueli.com/xxl-job/

xxljob 分布式任务调度平台，由调度中心和执行器组成，调度中心提供一个web管理界面配置任务和执行器，调度中心通过rpc触发执行器运行。



高可用

弹性扩容

日志

动态分片

监控告警



## 架构

![](/Users/baiyingjun/Library/Application%20Support/marktext/images/2022-07-22-22-44-25-image.png)



## xxljob-admin实践



### 下载源码

![](/Users/baiyingjun/Library/Application%20Support/marktext/images/2022-07-22-22-52-32-image.png)



### 启动和初始化数据库

docker run -d --name mysql -v /home/mysql/data:/var/lib/mysql -v /home/mysql/conf:/ect/mysql/conf.d -p 3308:3306 -e MYSQL_ROOT_PASSWORD=123456 mysql:5.7



运行初始化脚本：xxl-job/doc/db/tables_xxl_job.sql



修改配置文件中的数据库连接信息

![](/Users/baiyingjun/Library/Application%20Support/marktext/images/2022-07-22-23-07-48-image.png)



修改dockerFile，暴露端口

EXPOSE 8080



打包并上传到xxljob-admin目录



编译xxljob-admin镜像

```
docker build -t admin:1.0 .
docker run -it --name admin -p 8090:8080 admin:1.0 /bin/bash
```

通过 http://10.211.55.19:8090/xxl-job-admin/ 访问

![](/Users/baiyingjun/Library/Application%20Support/marktext/images/2022-07-23-00-17-12-image.png)



## xxljob 执行器实践

修改配置

```xml
# web port
server.port=8080
# no web
#spring.main.web-environment=false

# log config
logging.config=classpath:logback.xml


### xxl-job admin address list, such as "http://address" or "http://address01,http://address02"
xxl.job.admin.addresses=http://10.211.55.19:8090/xxl-job-admin

### xxl-job, access token
xxl.job.accessToken=default_token

### xxl-job executor appname
xxl.job.executor.appname=xxl-job-executor-sample
### xxl-job executor registry-address: default use address to registry , otherwise use ip:port if address is null
xxl.job.executor.address=
### xxl-job executor server-info
xxl.job.executor.ip=10.211.55.19
xxl.job.executor.port=9999
### xxl-job executor log-path
xxl.job.executor.logpath=/data/applogs/xxl-job/jobhandler
### xxl-job executor log-retention-days
xxl.job.executor.logretentiondays=30
修改
```





修改dockfile

EXPOSE 8080  
EXPOSE 9999



打包上传，打包镜像，启动

```shell
cd xxljob-exector/

 docker build -t exector:1.0 .

docker run -it --name exector -p 8091:8080 -p 9999:9999 exector:1.0 /bin/bash

```




在admin界面配置执行器（http://10.211.55.19:9999），在任务管理配置任务，开启调度，查看调度日志和调度效果

![](/Users/baiyingjun/Library/Application%20Support/marktext/images/2022-07-23-00-13-27-image.png)



## 执行器集群部署

再新启动一个执行器服务

docker run -it --name exector -p 8092:8080 -p 9998:9999 exector:1.0 /bin/bash



在web界面修改执行器地址

http://10.211.55.19:9999,http://10.211.55.19:9998



任务配置中的路由策略修改为轮询，开启任务查看效果





## xxljob实践-执行shell

新增任务（shell类型）



编写脚本

在默认的脚本样本中增加以下两行

```shell
dir = $(date +%Y%m%d%H%M%S)

mkdir /home/$dir
```

启动任务

进入容器查看是否成功创建了目录

docker exec -it exector /bin/bash  # 进入容器

![](/Users/baiyingjun/Library/Application%20Support/marktext/images/2022-07-23-00-14-50-image.png)
