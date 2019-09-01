## docker

[docker菜鸟教程](https://www.runoob.com/docker/docker-tutorial.html)

1. 容器的创建两种方式

   * 新建一个容器启动它，并且执行一个命令

     ```markdown
     docker run [OPTIONS] IMAGE [COMMAND] [ARG...]
     ```

     OPTIONS说明：

     - **-a stdin:** 指定标准输入输出内容类型，可选 STDIN/STDOUT/STDERR 三项；
     - **-d:** 后台运行容器，并返回容器ID；
     - **-i:** 以交互模式运行容器，通常与 -t 同时使用；
     - **-P:** 随机端口映射，容器内部端口**随机**映射到主机的高端口
     - **-p:** 指定端口映射，格式为：**主机(宿主)端口:容器端口**
     - **-t:** 为容器重新分配一个伪输入终端，通常与 -i 同时使用；
     - **--name="nginx-lb":** 为容器指定一个名称；
     - **--dns 8.8.8.8:** 指定容器使用的DNS服务器，默认和宿主一致；
     - **--dns-search example.com:** 指定容器DNS搜索域名，默认和宿主一致；
     - **-h "mars":** 指定容器的hostname；
     - **-e username="ritchie":** 设置环境变量；
     - **--env-file=[]:** 从指定文件读入环境变量；
     - **--cpuset="0-2" or --cpuset="0,1,2":** 绑定容器到指定CPU运行；
     - **-m :**设置容器使用内存最大值；
     - **--net="bridge":** 指定容器的网络连接类型，支持 bridge/host/none/container: 四种类型；
     - **--link=[]:** 添加链接到另一个容器；
     - **--expose=[]:** 开放一个端口或一组端口；
     - **--volume**	绑定一个卷 （这两个参数作用一致）
     - **–volumes-from** 把其容器挂载的内容也挂载到当前容器中
     -  **-v** 绑定一个卷(同--volume)

     ### 实例

     使用docker镜像nginx:latest以后台模式启动一个容器,并将容器命名为mynginx。

     ```
     docker run --name mynginx -d nginx:latest
     ```

     使用镜像nginx:latest以后台模式启动一个容器,并将容器的80端口映射到主机随机端口。

     ```
     docker run -P -d nginx:latest
     ```

     使用镜像 nginx:latest，以后台模式启动一个容器,将容器的 80 端口映射到主机的 80 端口,主机的目录 /data 映射到容器的 /data。

     ```
     docker run -p 80:80 -v /data:/data -d nginx:latest
     ```

     绑定容器的 8080 端口，并将其映射到本地主机 127.0.0.1 的 80 端口上。

     ```
     $ docker run -p 127.0.0.1:80:8080/tcp ubuntu bash
     ```

     使用镜像nginx:latest以交互模式启动一个容器,在容器内执行/bin/bash命令。

     ```
     runoob@runoob:~$ docker run -it nginx:latest /bin/bash
     root@b8573233d675:/# 
     ```

     把其容器挂载的内容也挂载到当前容器中

     ~~~markdown
     docker create -v $PWD/datadir:/var/mydatadir --name datacontainer ubuntu
     docker run -it --volumes-from datacontainer --name newdatacontainer ubuntu /bin/bash 
     在新容器中就可以看到/var/mydatadir了
     ~~~

     

   * 新建一个容器但不启动它，参数同 docker run

     ~~~markdown
     docker create [OPTIONS] IMAGE [COMMAND] [ARG...]
     ~~~

     

## 常用命令

### 容器生命周期管理

- [run](https://www.runoob.com/docker/docker-run-command.html)  创建并运行一个新的容器
- [start/stop/restart](https://www.runoob.com/docker/docker-start-stop-restart-command.html)  启动、停止、重启一个容器
- [kill](https://www.runoob.com/docker/docker-kill-command.html) 关闭一个容器向容器发送关闭信号
- [rm](https://www.runoob.com/docker/docker-rm-command.html) 删除一个关闭的容器
- [pause/unpause](https://www.runoob.com/docker/docker-pause-unpause-command.html)  暂停 启动一个容器内所有进程
- [create](https://www.runoob.com/docker/docker-create-command.html) 创建一个新的容器但不运行
- [exec](https://www.runoob.com/docker/docker-exec-command.html) 在容器中运行命令

### 容器操作

- [ps](https://www.runoob.com/docker/docker-ps-command.html) 查看所有容器
- [inspect](https://www.runoob.com/docker/docker-inspect-command.html) 获取容器的元数据
- [top](https://www.runoob.com/docker/docker-top-command.html) 查看容器内进程信息 类似 Linux top命令
- [attach](https://www.runoob.com/docker/docker-attach-command.html) 链接正在运行的容器（慎用，此命令比较复杂，且貌似有bug）
- [events](https://www.runoob.com/docker/docker-events-command.html) 获取容器或镜像执行的命令和事件 （具体配合参数使用）
- [logs](https://www.runoob.com/docker/docker-logs-command.html) 获取容器的日志
- [wait](https://www.runoob.com/docker/docker-wait-command.html) 阻塞运行直到容器停止，然后打印出它的退出代码。
- [export](https://www.runoob.com/docker/docker-export-command.html) 将文件系统作为一个tar归档文件导出到STDOUT
- [port ](https://www.runoob.com/docker/docker-port-command.html) 列出指定的容器的端口映射，或者查找将PRIVATE_PORT NAT到面向公众的端口。

### 容器rootfs命令

- [commit](https://www.runoob.com/docker/docker-commit-command.html) 从容器创建一个新的镜像
- [cp](https://www.runoob.com/docker/docker-cp-command.html) 用于容器与主机之间的数据拷贝
- [diff](https://www.runoob.com/docker/docker-diff-command.html) 检查容器里文件结构的更改

### 镜像仓库

- [login](https://www.runoob.com/docker/docker-login-command.html) 登陆到一个Docker镜像仓库，如果未指定镜像仓库地址，默认为官方仓库 Docker Hub
- [pull](https://www.runoob.com/docker/docker-pull-command.html) 从镜像仓库中拉取或者更新指定镜像
- [push](https://www.runoob.com/docker/docker-push-command.html) 将本地的镜像上传到镜像仓库,要先登陆到镜像仓库
- [search](https://www.runoob.com/docker/docker-search-command.html) 从Docker Hub查找镜像

### 本地镜像管理

- [images](https://www.runoob.com/docker/docker-images-command.html)  列出本地镜像
- [rmi](https://www.runoob.com/docker/docker-rmi-command.html)  删除本地一个或多少镜像
- [tag](https://www.runoob.com/docker/docker-tag-command.html) 标记本地镜像，将其归入某一仓库
- [build](https://www.runoob.com/docker/docker-build-command.html)  命令用于使用 Dockerfile 创建镜像
- [history](https://www.runoob.com/docker/docker-history-command.html) 查看指定镜像的创建历史
- [save](https://www.runoob.com/docker/docker-save-command.html) 将指定镜像保存成 tar 归档文件
- [load](https://www.runoob.com/docker/docker-load-command.html) 导入使用 [docker save](https://www.runoob.com/docker/docker-save-command.html) 命令导出的镜像
- [import](https://www.runoob.com/docker/docker-import-command.html) 从归档文件中创建镜像

### info|version

- [info](https://www.runoob.com/docker/docker-info-command.html) 显示 Docker 系统信息，包括镜像和容器数
- [version ](https://www.runoob.com/docker/docker-version-command.html) 显示 Docker 版本信息。

**docker-compose**

* up -d  守护进程启动服务
* stop 停止服务运行
* rm 删掉服务中的容器
* logs 查看服务容器日志
* ps 列出服务相关容器