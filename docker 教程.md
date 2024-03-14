## Docker 教程

| 操作                            | 命令(全)                                 | 命令(简)              |
| ------------------------------- | ---------------------------------------- | --------------------- |
| 容器的创建                      | `docker container run [镜像名]`          | `docker run [镜像名]` |
| 容器的列出(up)                  | `docker container ls`                    | `docker ps`           |
| 容器的列出(up和exit)            | `docker container ls -a`                 | `docker ps -a`        |
| 容器的停止                      | `docker container stop `                 | `docker stop `        |
| 容器的删除                      | `docker container rm `                   | `docker rm `          |
| 容器的运行模式(detach,后台运行) | `docker container run -d -p 80:80 nginx` |                       |
| 查看detach模式下log             | `docker attach `                         |                       |
| 清理空闲没有使用的容器          | `docker system prune -f `                |                       |
| 清理空闲没有使用的镜像          | `docker image prune -a`                  |                       |

### Container 快速上手

#### 运行模式

- attach 模式		例如：docker container run -p 80:80 nginx
- Detach 模式		例如：docker container run -d -p 80:80 nginx 容器会在后台运行

#### 批量停止

- 假设现在有以下三个容器`docker container ps `

```
CONTAINER ID   IMAGE     COMMAND                  CREATED          STATUS          PORTS     NAMES
cd3a825fedeb   nginx     "/docker-entrypoint.…"   7 seconds ago    Up 6 seconds    80/tcp    mystifying_leakey
269494fe89fa   nginx     "/docker-entrypoint.…"   9 seconds ago    Up 8 seconds    80/tcp    funny_gauss
34b68af9deef   nginx     "/docker-entrypoint.…"   12 seconds ago   Up 10 seconds   80/tcp    interesting_mahavira
```

- 方式一：`docker container stop cd3 269 34b`

- 方法二：`docker container stop $(docker container ps -q) `

#### 批量删除

- `docker container rm $(docker container ps -aq)`

#### 删除正在运行中的容器

`docker container rm [容器ID] -f`

`docker system prune -a -f`		可以快速对系统进行清理，删除停止的容器，不用的image(全部删除了，慎用)

#### 交互式运行容器

`docker container run -it busybox sh`		创建一个容器并进入交互式模式 

`docker container exec -it 33d sh`		在一个已经运行的容器里执行一个额外的command

#### docker container run 的背后

docker container run 的参数信息
- `-d` 以后台模式运行容器，并返回容器ID
- `-p` 指定端口映射，格式为 主机(宿主机)端口:容器端口
- `--name="xxx"` 为容器指定一个名称 
- `-it` i 已交互模式运行容器,t 为容器重新分配一个伪输入终端。一般结合使用
- `--rm` 一次性运行容器，容器运行结束后自动删除


docker container run -d -publish 80:80 --name webhost nginx

- 1. 在本地查找是否有nginx这个image镜像，但是没有发现
- 2. 去远程的image registry查找nginx镜像（默认的registry是Docker Hub)
- 3. 下载最新版本的nginx镜像 （nginx:latest 默认)
- 4. 基于nginx镜像来创建一个新的容器，并且准备运行
- 5. docker engine分配给这个容器一个虚拟IP地址
- 6. 在宿主机上打开80端口并把容器的80端口转发到宿主机上
- 7. 启动容器，运行指定的命令（这里是一个shell脚本去启动nginx）

### 镜像的创建管理与发布

#### 镜像的获取

默认从 Docker Hub 获取镜像。如何不指定版本，默认拉取最新版本。

- `docker pull [镜像名]`		从远程 Docker Hub 获取镜像
- `docker build -t [镜像名]:[标签] [上下文路径/URL/]`		使用 Dockerfile 构建自定义镜像
  - -t 指定镜像的名称和标签
  - -f 指定要使用的 Dockerfile 的路径，默认为当前目录下
  - --no-cache 禁用构建中的缓存，强制重新构建
  - --network 指定构建过程中使用的网络模式
- `docker load < [镜像文件]`		文件导入(离线形式)

#### 镜像的基本操作
| 操作     | 命令(全)                   | 命令(简) |
| -------- | -------------------------- | -------- |
| 查看镜像 | `docker image ls`          |          |
| 删除镜像 | `docker image rm [镜像ID]` |          |

#### 镜像的导出和导入

- `docker image save [镜像名:版本号] -o [自定义镜像名称.image]`		导出镜像到本地
- `docker image load -i [导出的本地的镜像]`				导入镜像

### Dockerfile 

#### Dockerfile 指令

##### FROM [镜像]	指定基础镜像

- 尽量选择体积小的镜像

- 固定版本 tag,而不是每次都使用 latest

- scratch 为基础镜像的话，表示一个空白的镜像

##### RUN	执行命令

  - shell 格式	类似在Linux命令行中输入命令一样。支持行尾添加 \ 换行以及行首 # 进行注释
  - exec 格式	RUN ["可执行文件","参数1","参数2"] 更像函数调用中的格式

##### COPY	复制文件

- COPY 源路径 镜像内目标路径	源路径可以是多个，也可以是通配符，需要满足 Go 的 filepath.Match 规则。目标路径也可以是工作目录的相对路劲。源文件的各种元数据会全部保存(读、写、执行权限等)
- 可以加上 --chown=[user]:[group] 来改变文件所属的用户和用户组

##### ADD	更高级的复制文件

- 和 COPY 基本保持一致
- 如果复制的是一个压缩文件时，ADD会自动解压文件

##### WORKDIR	指定工作目录

- WORKDIR <工作目录路径>
- COPY ADD 会将文件放到定义的工作目录里面。进入sh后默认会是指定的工作目录

##### ARG	构建参数

- 设置环境变量。ARG 只能保存在镜像构建的时候

##### ENV	设置环境变量

- 设置环境变量。ENV 设置的变量可以在Image中保持，并保持在容器中的环境变量里

##### CMD	容器启动命令

- 有两种格式 shell 和 exec 格式。 CMD可以用来设置容器启动时默认会执行的命令。
- shell 格式会被包装为 sh -c 进行执行。变量要使用 shell 格式或者 例如：CMD echo $HOME 实际执行 CMD [ "sh", "-c", "echo $HOME" ]
- `如果docker container run启动容器时指定了其它命令，则 CMD 命令会被覆盖`
- 可以设置 CMD 为空 [],覆盖默认镜像cmd，运行容器是自定义输入 cmd 命令。例如 docker container run -it myinfo ipinfo 8.8.8.8

##### ENTRYPOINT	入口点

- CMD 可以在 docker container run 时传入其他命令，覆盖掉 CMD 命令。但是 ENTRYPOINT 所设置的命令一定会执行。
- ENTRYPOINT 也可以设置容器启动时要执行的命令，可以在docker container run 时传入其它命令
- ENTRYPOINT 和 CMD 可以联合使用，ENTRYPOINT 设置执行的命令，CMD传递参数

##### VOLUME	定义匿名卷

- 容器运行时应该尽量保持容器存储层不发生写操作，对于数据库类需要保存动态数据的应用，其数据库文件应该保存于卷(volume)中。

