# 常用命令

#### Git全局设置

##### 设置用户信息

​	git	config	--global	user.name	"itcast"

​	git	config	--global	user.email	"hello@com.cn"

##### 查看配置信息

​	git	config	--list

##### 获取Git仓库,在本地初始化Git仓库

​	在当前目录打开Git bash窗口,执行命令:	git	init



#### 本地仓库操作

##### git	status:	命令用于查看文件状态

##### git	add:	将文件的修改加入暂存区

##### git	reset:	将暂存区的文件取消暂存或者是切换到指定版本

##### git	commit:	将暂存区的文件修改提交到版本库	-i(忽略报错,并提交)

​	git	commit	-m	"日志信息"	user.java

##### git	log:	查看日志

#### 远程仓库操作

##### git	remote:	查看远程仓库

​	git	remote	-v:	显示详细信息

##### git	remote	add:	添加远程仓库

​	语法:	get	remote	add	<shoriginName>	<url>:添加新的远程仓库,同	时指定一个可以引用的别名(origin)

​	更新远程仓库地址:	git	remote	set-url	origin <url>

##### git	clone:	从远程仓库克隆

​	语法:	git	clone	[远程仓库地址]

##### git	pull:	从远程仓库拉取最新版本并合并到本地仓库

​	语法:	git	pull	[remote-name](远程仓库别名)	[branch-name](远程仓库分支名称)

​	如果当前本地仓库不是从远程仓库克隆,而是本地仓库创建,并且仓库中存在文件,	此时从远程仓库拉去文件时会报错

​	解决此问题可在git	pull命令后加入参数--allow-unrelated-histories

##### git	push:	将本地仓库内容推送到远程仓库

​	语法:	git	push	[remote-name](远程仓库别名)	[branch-name](远程仓库分支名称)

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

#### 分支操作

##### git	branch	查看分支

​	git	branch	列出所有本地分支

​	git	branch	-r	列出所有远程分支

​	git	branch	-a	列出所有本地分支和远程分支

##### git	branch	[name]		创建分支

##### git	checkout	[name]	切换分支

##### git	push	[shortName](远程仓库名称)	[name](本地分支)	推送至远程仓库分支

##### git	merge	[name](分支name)	合并分支



#### 标签操作

##### git	tag	列出已有的标签

##### git	tag	[name]	创建标签

##### git	push	[shortName]	[name]	将标签推送至	远程仓库

##### git	checkout	-b	[branch]	[name]	检出标签