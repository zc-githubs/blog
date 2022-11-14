---
date: 2022-07-13-星期三 10:54:55
update: 
tags: [time/year2022,time/month07]
id: 20220713105455
---
# 语法
1 每个保留关键字（指令）都是必须是大写字母  
2 执行从上到下顺序执行  
3 # 表示注释  
4 每一个指令都会创建一个新的镜像层，并提交


```
FROM		#基础镜像，一切从这里开始构建
MAINTAINER	#镜像是谁写的，姓名+邮箱
RUN			#镜像构建的时候需要运行的命令
ADD			#步骤，tomcat镜像，这个tomcat的压缩包！添加内容
WORKDIR		#镜像的工作目录	
VOLUME		#挂载的目录
EXPOSE		#暴露端口配置
CMD			#指定这个容器启动的时候要运行的命令，只有最后一个会生效，可被替代
ENTRYPOINT	#指定这个容器启动的时候要运行的命令，可以追加命令
ONBUILD		#当构建一个被继承 Dockerfile 这个时候就会运行ONBUILD 的指令
COPY 		#类似ADD，将我们文件拷贝到镜像中
ENV			#构建的时候设置环境遍量
```
![](https://images.cubox.pro/1657681834877/749489/image.png)

# 实例
：多行命令不要写多个RUN，原因是Dockerfile中每一个指令都会建立一层.多少个RUN就构建了多少层镜像，会造成镜像的臃肿、多层，不仅仅增加了构件部署的时间，还容易出错,RUN书写时的换行符是\
# 指定基础镜像
```
FROM java:8
# 维护者信息
MAINTAINER  author_name "xxx@qq.com"

RUN echo "-------------------- 后端 api 环境配置 --------------------"

COPY ruoyi.jar /app.jar
``
# 暴露8080端口
EXPOSE 8080
# 设置环境编码UTF-8
ENV LANG C.UTF-8
```
# Quote
