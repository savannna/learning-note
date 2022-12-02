# 关于端口
## 1. dockerfile
```
dokerfile:
    expose:8080
```
- 声明**容器运行时**提供服务的端口(只是声明，在容器运行时**不会开启**这个端口的服务)  
- EXPOSE 仅仅是声明容器打算使用什么端口而已，并不会自动在宿主进行端口映射。
## 2. docker run -p 8080:8080  相当于权限转发
- 使用 -p 选项把本地主机指定端口映射到容器端口上，我们把本地的 10000 端口映射到容器 80 端口上。
- 这样我们就可以通过访问本机的 1000 端口来访问容器的 80 端口了，不然容器外部（本地主机除外）是无法通过网络来访问容器内的应用和服务的。
```
docker run -p <宿主端口>:<容器端口>
docker run -itd -p 10000:80 --name test1 centos:latest
```
-p，是映射**宿主端口**和**容器端口**(将容器的对应端口服务公开给外界访问)
## 3. docker ps ports
正在运行的容器
```
root@cp:~# docker ps -a
CONTAINER ID   IMAGE           COMMAND       CREATED         STATUS         PORTS                   NAMES
dd89ba3d1bdd   centos:latest   "/bin/bash"   7 seconds ago   Up 5 seconds   0.0.0.0:10000->80/tcp   test1
```
- PORTS: （本机：容器端口）容器的端口信息和使用的连接类型（tcp\udp）。
## 4. docker-compose.yml
https://www.jiyik.com/w/docker/docker-compose
```
version: '3'
services:
  web:
    build: .
    ports:
     - "5000:5000"
  redis:
    image: "redis:alpine"
  expose:
    - "3000"
```
> #### 1.ports:类似docker run -p 8080:8080
> #### 2.expose:暴露端口，但不映射到宿主机，只被连接的服务访问。
## 5. JDBC的URL：协议名+子协议名+数据源名：
- https://blog.csdn.net/lss0217/article/details/89494177?spm=1001.2101.3001.6650.1&utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7ERate-1-89494177-blog-111028913.pc_relevant_aa2&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7ERate-1-89494177-blog-111028913.pc_relevant_aa2&utm_relevant_index=2
- https://www.cnblogs.com/dmego/p/6556821.html
- 
```
c．mysql:
　　驱动：com.mysql.jdbc.Driver
　　URL：jdbc:mysql://127.0.0.1:port/dbname

　　注：127.0.0.1:数据库所在机器的名称
　　　　port:端口号，默认是3306
　　　　dbname:数据库名称
```
> machine_name：数据库所在的机器的名称  
> 如果是本机则是127.0.0.1或者是localhost  
> 如果是远程连接，则是远程的IP地址;
***
# 关于数据库连接
## 一、application.yml连接的是什么
> 编写程序的时候连接的库
> #### 注意数据库所在机器ip:
> 1. 本机是localhost
> 2. 如果要从外部访问docker容器的数据库：  
> - 要修改ip为172.17.0.2（如何获取见容器化启动笔记）
> - 修改容器mysql权限:(mysql添加允许远程访问权限)
https://medium.com/tech-learn-share/docker-mysql-access-denied-for-user-172-17-0-1-using-password-yes-c5eadad582d3
## 二、docker-compose.yml配置数据库
### 1. environment: - MYSQL_ROOT_PASSWORD
创建容器时，需要root密码
```
  mysql:
    image: mysql:8.0
    environment:
      - MYSQL_ROOT_PASSWORD=123456
      - MYSQL_DATABASE=database
      - MYSQL_USER=root
      - MYSQL_PASSWORD=123456
    container_name: mysql-container
```
### 2.environment: SPRING_DATASOURCE_URL: 
```
    environment:
      SPRING_DATASOURCE_URL: jdbc:mysql://mysql:3306/parking_lot?useUnicode=true&characterEncoding=UTF-8&useSSL=true&serverTimezone=Asia/Shanghai
```
mysql:service名字 （可以使用另一个容器的名称作为其主机名来访问另一个容器） 
> 在spring.datasource.url此处将属性声明为环境变量，因此它会覆盖application.yml文件中拥有的任何内容。
- https://dimitri.codes/docker-spring-boot/
- https://stackoverflow.com/questions/66169935/how-to-set-spring-datasource-url-inside-a-docker-container
***
# 关于docker-compose 多容器启动（（docker-compose -build 创建images）（-up 创建并运行容器)）
1. 使用 Dockerfile 定义您的应用程序的环境
2. 使用 docker-compose.yml 定义组成您的应用程序的服务
3. 运行docker compose up启动并运行程序
## docker-compose.yml文件
### 一、version
### 二、build
> 最好将dockerfile和docker-compose都放到根目录下，方便找jar包
### 三、network
https://zhuanlan.zhihu.com/p/387840381
#### 同一个网络下才能通讯
- 默认情况下，Compose为您的应用程序设置单个网络。services 服务的每个容器都加入默认网络，并且可以被该网络上的其他容器访问。
```
version: "3.9"
services:
  web:
    build: .
    ports:
      - "8000:8000"
  db:
    image: postgres
    ports:
      - "8001:5432"
```
- 运行docker-compose up，会发生以下情况：
> 1. 创建了一个名为 myapp_default 的网络。
> 2. 把web加入网络。
> 3. 把db加入网络。
### 四、links
链接到另一个服务中的容器。指定服务名称和链接别名 ("SERVICE:ALIAS")，或仅指定服务名称。

> 它们不需要启用服务进行通信 
- 默认情况下，任何服务都可以以该服务的名称访问任何其他服务。
- 在以下示例中，web可以访问db，并且设置别名为database：
```
version: "3.9"
services:

  web:
    build: .
    links:
      - "db:database"
  db:
    image: postgres
```
### 五、depends_on
- 表示服务之间的依赖关系。服务依赖会导致以下行为：
- > 默认情况下 compose 启动容器的顺序是不确定的，但是有些场景下我们希望能够控制容器的启动顺序，比如应该让运行数据库的程序先启动。我们可以通过 depends_on 来解决有依赖关系的容器的启动顺序问题：
1. docker-compose up**按依赖顺序启动服务**。在下面的例子中，db和redis在 web之前启动。
2. docker-compose up SERVICE自动包含SERVICE的依赖项。在下面的示例中，docker-compose up web还创建并启动db和redis。
3. docker-compose stop按依赖顺序停止服务。在以下示例中，web在db和redis之前停止。
```
version: "3.9"
services:
  web:
    build: .
    depends_on:
      - db
      - redis
  redis:
    image: redis
  db:
    image: postgres
```
### 六、配置数据卷(volume) 一旦您销毁数据库容器并重新创建它，所有数据都将丢失。所以存到规定的卷位置
- https://www.cnblogs.com/sparkdev/p/9803554.html
- 卷理解：https://ssshooter.com/2020-02-22-docker-data-management/
> 卷相当于在光盘里创建文件夹存数据  

数据卷是处理容器中的持久化数据的主要方式，在 compose 中我们可以通过两种方式来指定数据卷：

> 1. 使用命名的数据卷
> 2. 直接指定主机上的路径来创建数据卷
```
version: "3.2"
services:
  web:
    image: nginx:alpine
    volumes:
    
      - type: volume
        source: mydata
        target: /data
        
        # 一个 bind 类型的 volume 在当前目录下的 nginx/logs 目录下保存 nginx 的日志：
      - type: bind
        source: ./nginx/logs
        target: /var/log/nginx
  jenkins:
    image: jenkins/jenkins:lts
    
    # 创建两个卷：分别是两个命名的数据卷 jenkins_home 和 mydata
    volumes:
      - jenkins_home:/var/jenkins_home
      # 共享数据卷
      - mydata:/data
volumes:
  mydata:
  jenkins_home:
```
> - 在这个例子中我们一共创建了三个数据卷，分别是两个命名的数据卷 jenkins_home 和 mydata：/一个 bind 类型的 volume 在当前目录下的 nginx/logs 目录下保存 nginx 的日志：
> - 其中的 jenkins_home 数据卷是给 jenkins 保存数据的。  
> - 如果要在多个容器之间共享数据卷，就必须在顶级的 volumes 节点中定义这个数据卷，
> - 比如 mydata 数据卷，它被 web 和 jenkins service 共享了。比如我们在 web service 中的 mydata 数据卷中创建一个名为 hello 的文件，该文件会同时出现在 jenkins service 中：
### 七、配置日志驱动
```
services:
  web:
    logging:
          driver: "json-file"
          options:
            max-size: "200k"
            max-file: "10"
```
