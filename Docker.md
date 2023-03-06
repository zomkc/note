```sh
systemctl start docker  # 启动docker服务

systemctl stop docker  # 停止docker服务

systemctl restart docker  # 重启docker服务
```





## 镜像操作命令

docker push:	推送镜像到服务

docker pull:	从服务拉去镜像

docker save:	保存镜像为一个压缩包

docker load:	加载压缩包为镜像

docker images:	查看镜像

docker rmi:	删除镜像

docker build:	构造镜像



## 容器相关命令

docker run命令的常见参数

​	--name: 指定容器名称

​	-p: 指定端口映射

​	-d: 让容器后台运行

查看容器日志的命令

​	docker log

​	添加 -f 参数可以持续查看日志

查看容器状态

​	docker ps		添加-a参数查看所有状态的容器

删除容器:

​	docker rm		不能删除运行中的容器,除非添加-f参数

进入容器:

​	docker exec -it [容器名] [要执行的命令]

运行中容器暂停

​	docker pause

暂停的容器运行

​	docker unpause

停止容器

​	docker stop

运行停止的容器

​	docker start

退出容器

​	exit



案例:

`docker run --name containerName -p 80:80 -d nginx`

​	docker run: 创建并运行一个容器

​	--name: 给容器起一个名字

​	-p: 将宿主机端口与容器端口映射,冒号左边是宿主机端口,右侧是容器端口4

​	-d: 后台运行容器

​	nginx: 镜像名称



案例: 进入容器,修改文件内容

`docker exec -it n bash`

命令解读:

​	docker exec:	进入容器内部,执行一个命令

​	-it:	给当前容器创建一个标准输入,输出终端,允许我们与容器交互

​	mn:	要进入的容器的名称

​	bash:	进入容器后执行的命令



## 数据卷

数据卷(volume) 是一个虚拟目录,指向宿主机文件系统中的某个目录

##### 数据卷操作的基本语法

docker	volume	[COMMAND]

docker	volume命令是数据卷操作,根据命令后跟随的command来确认下一步操作

* create  创建一个volume
* inspect  显示一个或多个volume的信息
* ls  列出所有的volume
* prune  删除未使用的volume
* rm  删除一个或多个指定的volume

数据卷挂载方式:

运行容器时使用  -v  参数挂载数据卷

-v	volumeName:/targetContainerPath

如果容器运行时volume不存在,会被自动创建

`docker run --name containerName -p 80:80 -v html:/usr/share/nginx/html -d nginx`





## Dockerfile

Dockerfile是一个文本文件,其中包含一个个的指令,用指令来说明要执行什么操作来构建镜像,每一个指令都会形成一层Layer

![QQ截图20221027143903](assets/QQ%E6%88%AA%E5%9B%BE20221027143903.png)