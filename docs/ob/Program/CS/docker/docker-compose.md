---
date: 2022-07-13-星期三 15:22:35
update: 
tags: [time/year2022,time/month07]
id: 20220713152235
---

# Docker容器管理之Dockercompose.yml文件格式说明


### 1.yml文件格式说明

这一部分的yml文件格式主要参考[YAML 入门教程](https://www.runoob.com/w3cnote/yaml-intro.html)，在docker-compose.yml文件中会使用到的基础格式如下。

*   基本语法
    
    > \*_文件格式: _.yml__-
    > **使用缩进表示层级关系**-
    > **缩进不允许使用tab，只允许空格**-
    > **缩进空格数不重要，相同层的元素左对齐即可**-
    > **注释符号:#**
    
*   数据类型-
    – 对象：键值对
    
    > key : value
    
    – 数组：按次序排列的值
    
    > Languages:-
    > \\quad \- Ruby-
    > \\quad \- Perl-
    > \\quad \- Python
    
    – 特殊符号
    
    > \\quad & 建立锚点-
    > \\quad \* 引用锚点-
    > \\quad << 合并当前数据
    

特殊符号的使用具体见文献[YAML 入门教程](https://www.runoob.com/w3cnote/yaml-intro.html)。

## 2.docker-compose.yml文件相关说明


*   docker-compose.yml文件格式说明

docker-compose.yml文件主要包括指定docker-compose的模式应用版本(version)，设计基本的容器配置服务(services)，以及设置容器之间通信的网络模式，基本的[YAML](https://so.csdn.net/so/search?q=YAML&spm=1001.2101.3001.7020)文件书写格式如下所示：

 ```
   version: 指定版本号
    services:
      容器1:
         容器设置参数
      容器2：
         容器设置参数
    networks:
      网络名：
         网络参数设置
```
    
        

简单的docker-compose.yml例子如下

  ```
  version: 3.8
	services:
      web:
        build: .
        ports:
          - "5000:5000"
        networks:
          - eth0
    networks:
    	eth0:
        name: net1
```
    
        

*   重要的关键字说明

– version: 用于设置版本号-
\\quad 其版本号的确定主要参考docker-compose文档[Compose file versions and upgrading](https://docs.docker.com/compose/compose-file/compose-versioning/)，每一个版本号都给定了相关的Docker-Engine的兼容性说明，具体如下图所示。-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F6ddf8d26216549a483230ece67f4ff15.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBAQlBEWlo%3D%2Csize_20%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16)-
\\qquad 值得注意的是，在docker-compose.yml文件中的最大版本的确定实际上是跟主机上安装的Docker-Engine的版本具有直接关系。在使用时，一定要确定自己主机安装的Docker-Engine的版本。-
– **services: 容器实体部分**-
services部分主要是用于确定集群中每个容器执行的行为，实际上services关于容器设置方面其实和Dockerfile文件具有一定的相关性，部分Dockerfile中设置容器的参数也可以放在services字段中进行设置。

> build: 指定构造镜像上下文路径-
> image: 指定容器运行的镜像-
> endpoint\_mode: 访问集群服务的方式-
> cap\_add(添加)，cap\_drop(删除)容器拥有的宿主机的内核功能。-
> links: 解决容器连接问题-
> env\_file: 从文件中读取环境变量-
> entropoint: 覆盖Dockerfile中的ENTROPOINT-
> ports: 建立主机端口到容器端口的映射-
> command: 覆盖容器启动的默认命令-
> container\_name: 指定容器名称-
> deploy: 容器部署的配置设置-
> tty: true 分配一个伪终端-
> volumes: 将主机的数据或者文件挂载到容器里面

– networks: 容器之间通信的网络部分

networks是主要设置容器之间通信所基于的网络，关于网络的具体配置参数可以参考文献[Networking in Compose](https://docs.docker.com/compose/networking/)，这里仅仅给出一些主要的配置参数。

> name: 网络名-
> network\_mode: 设置网络模型-
> driver: 指定哪种驱动将用于这个网络-
> enable\_ipv6: 启动ipv6-
> ipam(IP Address Management): IP地址管理

当启动一个要连接上其他容器的容器时，docker会创建一些环境变量。例如--link参数的使用。docker会暴露出所有源容器的环境变量，这些环境变量包括：Dockerfile中设定的环境变量，docker run启动使用-e,--env,--env-file参数指明的环境变量。注意：**一个容器的所有环境变量都可以被任何连接上它的容器访问。**

## 总结

这篇文章主要讲述了docker-compose.yml文件配置的基础格式和主要参数的配置信息。







# Quote
