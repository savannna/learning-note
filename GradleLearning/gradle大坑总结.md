# build.gradle文件
***
## 文件本身
1. jar{}要写在dependency{}后面
2. plugins与apply plugin
```
//Using legacy plugin application
buildscript {
  repositories {
    maven {
      url "https://plugins.gradle.org/m2/"
    }
  }
  dependencies {
    classpath "org.springframework.boot:spring-boot-gradle-plugin:2.2.2.RELEASE"
  }
}

apply plugin: "org.springframework.boot"
```
> buildscript {}”块指定第三方库作为Gradle插件的话，指定插件就需要使用“apply plugin”了。
```
//Using the plugins DSL：
plugins {
 id "org.springframework.boot" version "2.2.2.RELEASE"
}

```
> plugins{}引入的插件得是Gradle官方
3. 其他部分
```
//组名（前缀包名）
group = 'com.thoughtworks.demo'

version = '0.0.1'

//JDK版本
sourceCompatibility = '11'
```
4.configuration  
https://www.jianshu.com/p/0e04186a6aca
https://www.cnblogs.com/davenkin/p/gradle-learning-7.html
> - configuration 用来管理一批 dependencies。
> - 对不同的构建场景声明不同的依赖集合
> - 每一组依赖称为一个Configuration，在声明依赖时，我们实际上是在设置不同的Configuration。  

> 自定义configurations，并添加依赖:
>```
> configurations {
>   myDependency
>}
>
>dependencies {
>   myDependency 'org.apache.commons:commons-lang3:3.0'
>}
>```  

> 没有声明configuration:
> - java Plugin会自动定义compile/testCompile/runtime/testRuntime/archives/default
> - 官方文档：https://docs.gradle.org/current/userguide/java_plugin.html#sec:java_plugin_and_dependency_management
> - 根据不同场景运行依赖
>```
>dependencies {
>  compile 'org.slf4j:slf4j-log4j12:1.7.2'
>   testCompile 'junit:junit:4.8.2'
>}
>```
5. repositories
> - 在声明依赖时，我们需要告知Gradle上哪去获取这些依赖。依赖保存的位置或路径称之为repository  
> - Gradle会在声明的reposity仓库中查找定位我们声明的依赖，从远程下载或从本地目录、仓库获取这些依赖的产物用于构建，并保存在本地缓存中。
> - Gradle根据声明的依赖，按Repository声明的顺序，从上到下依次查找。只要找到了一项就返回，不会继续往下查找！因此，需要**注意Repository声明的顺序**。
```
//全局仓库
repositories {
    //maven仓库中心
    mavenCentral()
}
```
***
## 打jar包
```
jar {
    enabled = false
    manifest {
        attributes 'Main-Class' : 'com.thoughtworks.demo.DemoApplication'
        attributes 'Class-Path' : configurations.runtimeClasspath.collect { 'libs/'+it.getName() }.join(' ')
    }
//    archiveBaseName = 'demo'
//    archiveVersion = '1.0.0'
}
```
1. gradle打完的jar包存在**build/libs**下
2. manifest指定main函数在哪
3. 生成**普通**jar包：会生成尾部为-plain.jar
> 该-plain.jar文件是一个普通的 jar 文件，没有任何 Spring Boot 的自定义。这意味着它不包含您的应用程序的依赖项

> 生成有spring boot依赖的jar包：
> ```
> jar {
>   enabled = false
> }
> ```
***
## 生成docker image
### 1. Dockerfile注意事项
Dockerfile
```
FROM --platform=linux/amd64 openjdk:11

VOLUME ["/tmp"]

ADD demo-0.0.1.jar app.jar

ENTRYPOINT ["java", "-Djava.security.egd=file:/dev/./urandom", "-jar", "app.jar"]

EXPOSE 8080

```
1. FROM是源image，M1芯片要额外加**platform=linux/amd64**
2. ADD后面的文件名要与build/docker里的jar包名一致
### 2. build.gradle注意事项
```
buildscript {
    ext {
        springBootVersion = '2.7.5.RELEASE'
    }

    repositories {
        mavenCentral()
    }

    dependencies {
        classpath('org.springframework.boot:spring-boot-gradle-plugin:${springBootVersion}')
        classpath('se.transmode.gradle:gradle-docker:1.2')
    }
}

apply plugin: 'docker'
apply plugin: 'application'

```
```
docker{
    baseImage 'openjdk'
    maintainer 'root'
    useApi true
    hostUrl 'http://myserver:4243'
    apiUsername 'savannna'
    apiPassword 'password'
    apiEmail 'jing.li3@thoughtworks.com'
}
```
或
```
distDocker{
    baseImage 'openjdk'
    maintainer 'root'
}
```
```
//他的依赖来源于buildscript
task buildDocker(type: Docker, dependsOn: build) {
    dockerfile = file('src/main/docker/Dockerfile')
    doFirst {
        copy {
            from "build/libs"
            into "build/docker"
        }
    }
}
```
1. 版本很重要，在通过gradle创建image的时候，出现了missing annotation
> 需要在gradle/wrapper降版本成6.?.?
2. task buildDocker作用：将build/libs的jar包复制到build/docker
3. > **buildscript**中的声明是gradle脚本自身需要使用的资源。  
   >- 可以声明的资源包括依赖项、第三方插件、maven仓库地址等。  
   > - (而这些插件、类库又不是直接服务于项目的，而是支持其它build脚本的运行。)  
   > - 所以你应当将这部分的引用放置在buildscript代码块中。gradle在执行脚本时，会优先执行buildscript代码块中的内容，然后才会执行剩余的build脚本。
   > - https://www.cnblogs.com/huang0925/p/3940528.html

   > 在**build.gradle**文件中直接声明的依赖项、仓库地址等信息是项目自身需要的资源。
***
