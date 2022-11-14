---
date: 2022-07-04-星期一 18:02:06
update: 
tags: [time/year2022,time/month07]
id: 20220704180206
---

# docker
```
`docker ps` 查看当前运行中的容器  
`docker images` 查看镜像列表  
`docker rm container-id` 删除指定 id 的容器  
`docker stop/start container-id` 停止/启动指定 id 的容器  
`docker rmi image-id` 删除指定 id 的镜像  
`docker volume ls` 查看 volume 列表  
`docker network ls` 查看网络列表
```
# docker-compose
Compose 是用于定义和运行多容器 Docker 应用程序的工具。使用 Compose，您可以使用 YAML 文件来配置应用程序的服务。然后，使用单个命令，从配置中创建并启动所有服务。若要了解有关“撰写”的所有功能的详细信息，请参阅[功能列表](https://docs.docker.com/compose/#features)。
```
在后台运行只需要加一个 -d 参数`docker-compose up -d`  
查看运行状态：`docker-compose ps`  
停止运行：`docker-compose stop`  
重启：`docker-compose restart`  
重启单个服务：`docker-compose restart service-name`  
进入容器命令行：`docker-compose exec service-name sh`  
查看容器运行log：`docker-compose logs [service-name]`
```
目录挂载





# Quote
