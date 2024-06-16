# Docker使用教程

# Docker使用教程
## Docker架构
* image(镜像)：可以看作是一个root文件系统，是对环境和软件的包装
* container(容器)：镜像类似于类，容器类似于实例
* repository(仓库)：镜像库
## Docker基本使用
### 启动本地dokcer软件
* sudo systemctl start docker
### 通用
* 查看所占空间`docker system df`
### repository使用
* 获取image`docker pull [选项] [仓库地址[:端口号]/仓库名[:标签]]`
    * 不加仓库地址，默认为docker hub
### image管理
* 列出本机的image文件:`docker image ls`
* 根据容器生成image文件:`docker commit [选项] <容器名ID或容器名> [<仓库名>[:<标签>]]`
* 删除本机的image文件:`docker image rm [imageName]`
* 删除所有处于停止状态的image:`docker image prune`
### container管理
#### 文件管理
* 列出本机运行的container文件:`docker container ls`
* 列出本机所有的container文件:`docker container ls --all`
* 导出容器:`docker export <containerID> > *.tar`
* 导入容器:`docker import *.tar <镜像名>:<标签>`
* 删除本机的container文件:`docker container rm <containerID>`
* 清理所有处于停止状态的容器:`docker container prune`
#### 运行管理
* 启动容器:`docker run [选项] [imageName] [sh终端]`
    * 选项可以省略
    * `-p 本地端口号：docker端口号`端口映射
    * `-d`后台运行不进入
    * `-i`是允许交互式操作
    * `-t`是终端
* 进入容器:`docker exec -it <容器名/id> [sh终端]`
* 终止容器:`docker container stop <容器名/id>`
* 启动已终止容器:`docker container start <容器名/id>`
* 重启容器:`docker container restart <容器名/id>`
### 网络管理
#### 概述
Docker容器的网络允许容器间、容器与宿主机、以及容器与外部网络的通信。
Docker默认提供几种网络模式：bridge（桥接模式）、host（主机模式）、none（无网络）和overlay（覆盖网络）。
* 查看与存在的网络:`docker network list`
* 创建自定义网络：`docker network create --driver <driver> <network_name>`
* 查看网络信息：`docker network inspect <network_name>`
* 连接/断开容器到网络：`docker network connect/disconnect <network_name> <container_name>`
##### Bridge模式
* Bridge模式是Docker默认的网络模式。
* 它通过一个虚拟的网络桥接器在宿主机上创建一个网络，使不同容器间可以相互通信。
* 适用于单机部署的多个容器需要相互通信的场景。
* 不加参数时，docker容器默认运行在bridge模式下
* 创建网络：`docker network create --driver bridge my_bridge`
* 容器连接到网络：`docker run --network=my_bridge -d nginx` 或 `docker network connect my_bridge my_container`
* 同一`bridge`可以通过`container name`作为`ip`来访问对方的网络
##### Host模式
* 在Host模式下，容器共享宿主机的网络命名空间，没有自己的IP地址。
* 这种模式下，容器可以直接使用宿主机的网络接口。
* 适用于需要直接访问宿主机网络资源的高性能应用。
* 运行容器：`docker run --network host -d nginx`

##### None模式
* None模式为容器提供了隔离的网络命名空间，但不为其配置网络接口。
* 容器在这种模式下无法进行网络通信。
* 适用于需要完全网络隔离的特定场景。
* 创建无网络容器：`docker run --network none -d nginx`
##### Overlay模式
* Overlay模式用于支持Docker Swarm集群中跨主机的容器通信。
* 它创建了一个分布式网络层，使得不同主机上的容器能够相互通信。
* 适用于大规模的分布式应用部署。
* 创建网络：`docker network create --driver overlay my_overlay`
* 部署服务：`docker service create --network my_overlay nginx`
##### Container模式
* Container模式允许一个容器共享另一个容器的网络栈。
* 这种模式常用于紧密连接的容器之间，例如当一个辅助容器需要使用主容器的网络接口时。
* 运行新容器时使用：`docker run --network container:<container_name_or_id> nginx`

## Podman
* Podman与docker完全兼容，只需要将docker命令里的docker更换为Podman即可

## docker-compose
dokcer-compose是本地docker服务编排工具，用于定义和管理多个docker服务。
### 常用命令
* `docker-compose -help` 查看帮助。
* `docker-compose config -q` 验证docker-compose.yml文件。当配置正确时，不输出任何内容，当配置错误时，输出错误信息。
* `docker-compose pull` 拉取服务依赖的镜像。
```
# 拉取工程中所有服务依赖的镜像
docker-compose pull
# 拉取工程中 nginx 服务依赖的镜像
docker-compose pull nginx
# 拉取镜像过程中不打印拉取进度信息
docker-compose pull -q
```
* `docker-compose up` 创建并启动所有服务的容器。
```
# 前台启动
docker-compose up
# 后台启动
docker-compose up -d
# -f 指定使用的 Compose 模板文件，默认为 docker-compose.yml，可以多次指定，指定多个 yml
docker-compose -f docker-compose.yml up -d
# 删除未定义的docker
docker-compose up --remove-orphans
```
* `docker-compose logs` 查看服务容器的输出日志。
```
# 输出日志，不同的服务输出使用不同的颜色来区分
docker-compose logs
# 跟踪日志输出
docker-compose logs -f
# 关闭颜色
docker-compose logs --no-color
```
* `docker-compose ps` 列出工程中所有服务的容器。
```
# 列出工程中所有服务的容器
docker-compose ps
# 列出工程中指定服务的容器
docker-compose ps nginx
```
* `docker-compose run` 在指定服务容器上额外执行一个命令。
```
# 在工程中指定服务的容器上执行 echo "helloworld"
docker-compose run nginx echo "helloworld"
```
* `docker-compose exec`进入服务容器。
```
# 进入工程中指定服务的容器
docker-compose exec nginx bash
# 当一个服务拥有多个容器时，可通过 --index 参数进入到该服务下的任何容器
docker-compose exec --index=1 nginx bash
```
* `docker-compose pause`暂停服务容器
```

# 暂停工程中所有服务的容器
docker-compose pause
# 暂停工程中指定服务的容器
docker-compose pause nginx
```
* `docker-compose unpause`恢复服务容器。
```
# 恢复工程中所有服务的容器
docker-compose unpause
# 恢复工程中指定服务的容器
docker-compose unpause nginx
```
* `docker-compose restart`重启服务容器。
```
# 重启工程中所有服务的容器
docker-compose restart
# 重启工程中指定服务的容器
docker-compose restart nginx
```
* `docker-compose start`启动服务容器。
```
# 启动工程中所有服务的容器
docker-compose start
# 启动工程中指定服务的容器
docker-compose start nginx
```
* `docker-compose stop`停止服务容器。
```
# 停止工程中所有服务的容器
docker-compose stop
# 停止工程中指定服务的容器
docker-compose stop nginx
```
* `docker-compose kill`通过发送SIGKILL信号停止指定服务的容器。
```
# 通过发送 SIGKILL 信号停止工程中指定服务的容器
docker-compose kill nginx
```
* `docker-compose rm`删除服务（停止状态）容器。
```
# 删除所有（停止状态）服务的容器
docker-compose rm
# 先停止所有服务的容器，再删除所有服务的容器
docker-compose rm -s
# 不询问是否删除，直接删除
docker-compose rm -f
# 删除服务容器挂载的数据卷
docker-compose rm -v
# 删除工程中指定服务的容器
docker-compose rm -sv nginx
```
* `docker-compose down`停止并删除所有服务的容器、网络、镜像、数据卷。
```
# 停止并删除工程中所有服务的容器、网络
docker-compose stop
# 停止并删除工程中所有服务的容器、网络、镜像
docker-compose down --rmi all
# 停止并删除工程中所有服务的容器、网络、数据卷
docker-compose down -v
```
* `docker-compose images`打印服务容器所对应的镜像。
```
# 打印所有服务的容器所对应的镜像
docker-compose images
# 打印指定服务的容器所对应的镜像
docker-compose images nginx
```
* `docker-compose port`打印指定服务容器的某个端口所映射的宿主机端口。
```
docker-compose port nginx 80
0.0.0.0:80
docker-compose port jenkins 8080
0.0.0.0:8088
```
* `docker-compose top` 显示正在运行的进程。
```
# 显示工程中所有服务的容器正在运行的进程
docker-compose top
# 显示工程中指定服务的容器正在运行的进程
docker-compose top nginx
```
### yml文件配置
```
version: '3'
networks:                         #自定义网络
  monitoring:
    driver: bridge
volumes:                          #定义一个docker命名卷，使用方式等同于本地路径，由docker自动管理位置，通常位于/var/lib/docker/volumes
  data: {}

services:
  jenkins:
    image: jenkins/jenkins:lts    #镜像名称
    container_name: jenkins       #设置容器名称
    #user: root                   #使用root用户启动
    user: 1000:994                #uid=1000(daizhao) gid=1000(daizhao) 组=1000(daizhao),993(docker) 使用uid代替username 避免报错Error response from daemon: unable to find user ubuntuu: no matching entries in passwd file
    privileged: true              #拥有root用户的权限
    restart: always               #跟随docker的启动而启动
    environment:                  #设置环境变量
        JAVA_OPTS: '-Djava.util.logging.config.file=/var/jenkins_home/log.properties'
    volumes:                      #冒号前为本地路径，冒号后为docker中的路径
      - /mnt/data/docker-mount/jenkins/:/var/jenkins_home                     #挂载jenkins工作目录
      - /etc/localtime:/etc/localtime                                             #挂载时间                                  
      - /var/run/docker.sock:/var/run/docker.sock
      - /usr/bin/docker:/usr/bin/docker
      - /usr/lib/x86_64-linux-gnu/libltdl.so.7:/usr/lib/x86_64-linux-gnu/libltdl.so.7
      - data:/data #挂载docker命名卷
    ports:
      - 8080:8080
    expose: #暴露给其他容器、link的端口号
      - 8080
      - 50000
    networks:
      - monitoring
  nginx:
    image: nginx:latest          #镜像名称 
    container_name: nginx        #设置容器名称
    restart: always              #跟随docker的启动而启动
    network_mode: host           #网络端口模式为主机 设置这个以后 不能再设置端口，类似docker --net: host
    volumes:                     #挂载卷命令
      - /mnt/data/docker-mount/nginx/conf/nginx.conf:/etc/nginx/nginx.conf                #映射配置文件入口文件
      - /mnt/data/docker-mount/nginx/html:/usr/share/nginx/html                           #nginx静态资源根目录挂载         
      - /mnt/data/docker-mount/nginx/logs:/var/log/nginx                                  #日志文件挂载        
      - /mnt/data/docker-mount/nginx/conf.d:/etc/nginx/conf.d                             #映射配置文件  
      - /home/daizhao/web:/home/daizhao/web                                                   #自定义扩展静态资源目录挂载
      - /home/daizhao/static:/home/daizhao/static                                             #自定义扩展静态资源目录挂载
    #ports:                       #宿主主机端口80 映射到 容器端口80
    # - 80:80   
  redis: 
    image: redis:latest          #镜像名称
    container_name: redis        #设置容器名称
    command: redis-server --appendonly yes --requirepass 'zbwZ1GqfPf7Kmx5*JS_s'   #开启持久化的支持并设置认证密码 
    restart: always              #跟随docker的启动而启动
    volumes:                     
      - /mnt/data/docker-mount/redis/data:/data                 #数据文件挂载
      - /mnt/data/docker-mount/redis/redis.conf:/usr/local/etc/redis.conf #配置文件挂载
    ports:                       #宿主主机端口6379 映射到 容器端口6379
      - 6379:6379
```
## dokcer使用技巧
### 更新docker镜像
1. `docker-compose pull`
2. `docker-compose up -d --remove-orphans`
3. `docker image prune`

### 自动更新docker镜像
* 运行`watchtower`  
```shell
$ docker run --detach \
    --name watchtower \
    --volume /var/run/docker.sock:/var/run/docker.sock \
    containrrr/watchtower
```
* 使用docker-compose
```
  watchtower:
    image: containrrr/watchtower:latest
    tty: true
    container_name: watchtower
    restart: unless-stopped
    environment:
      - WATCHTOWER_CLEANUP=true
      - WATCHTOWER_POLL_INTERVAL=3600
      - WATCHTOWER_ROLLING_RESTART=true
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
```

### 自动更新podman镜像
1. 为需要更新的容器添加标签`io.containers.autoupdate=registry/local`
2. 为该容器创建systemd服务
```shell
# root
sudo podman generate systemd --new --name container_name > /etc/systemd/system/container_name.service

# rootless
podman generate systemd --new --name container_name > ~/.config/systemd/user/container_name.service
```

3. 使用systemctl开启容器服务
```
# 使用podman停止服务

# root
sudo systemctl daemon-reload
sudo systemctl enable SERVICE_NAME.service

# rootless
systemctl --user daemon-reload
systemctl --user enable SERVICE_NAME.service

```

4. 开启自动更新服务
```shell
sudo systemctl enable podman-auto-update.service

# 或者手动自动更新
podman auto-update
```


### container访问host网络
* 使用`host`模式
* 使用`host.docker.internal`作为ip地址
> Linux系统需要添加`--add-host=host.docker.internal:host-gateway`到启动命令中
>
> `docker-compose`需要在容器定义中添加如下配置:
>
> ```
> extra_hosts:
    - "host.docker.internal:host-gateway"
> ```


