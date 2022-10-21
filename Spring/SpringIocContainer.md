https://github.com/gtb-2022-li-jing2/learning-spring-ioc-container-withcomments
# 自己创建一个容器
1. controller中想用service，调用工厂.get
2. 写一个工厂or容器 包含实例和get方法
3. 或 写一个XML文件+写一个解析XML的东西reflect

## 一、容器or工厂 要求
1. 全局资源，可共享（见factory） /单例模式：饿汉/单例：容器中，一个bean只有一个对象
2. 避免直接依赖。别人塞给我的 set 构造（详见service）
### no
```
public class ServiceA{
    private final DateSourse source =new DateDource;
}
```
### yes
```
 public class ServiceA{
    private final DateSourse source;

    public A(Date source){
        this.source = source;
    }
}
```
#### 写一个构造方法，调用它，让source传进来，而不是new一个
3. 开发者不自己new实例（因为实例通常一个参数连着一个连着一个，构造起来很复杂）


## 二、配置文件表达了（见 resources 的bean.xml）
1. 有哪些实例在容器里
2. 这些实例的依赖关系

## 三、根据反射机制 获取类对象，创建实例（怎么解析bean.xml的）（见src/main/java/factory/BeanReflectFactory.java）
image.png

# Ioc模式：控制反转：组建不用自己创建，全归容器管
# DI：注入
### Ioc container= bean factory+reflect



# Spring中的Bean
## 配置bean的三种方法
## 一、XML配置(见application- bean.xml)
## 不常用
1. XML声明了需要一个service实例
2. 写一个service 里面有sayhello方法（见main/ioc/service/service .java）
3. 写一个config：说明是和哪个XML关联的（src/main/java/ioc/config/BeanConfig.java）
4. 实现调用在controller里（src/main/java/com/thoughtworks/capability/ioc/App.java）
## 二、注解配置
1. 例如@controller @service……（service2+web/WebController.java+app.java(启动扫描)）
## 三、@Bean+ @Configration
## 精确配置
1.  见serviceConfig.java

# DI 依赖注入的三种方法
### 例如在controller里想用service实例，就要注入
### 例：注入service3
## 一、
## 简单
```
@Autowired(required =true)
private Service3 service3;
```
## 二、get set 注入
## 繁琐
```
private Service3 service3;

@Autowired
public void setService(Service3 service3){
    this.service3=service3;
}
```
## 三、有参构造器注入
## 推荐
```
private final Service3 service3;

public Webcontroller(Service3 service3){
 this.service3=service3;
}
```


# DI注入多种注入（见ioc/work）
```
@Autowired
private Person person;
```
### 想注入person类，可实际上有两个类都implenments person,都实现了person接口。怎么决定注入哪一个呢
## 法一、加注解@Autowired+@Qualifier
```
  @Autowired
    @Qualifier(value = "teacher")
    //⬆️ person有好多个bean，这个注解表明了注入哪一个
    private Person person;
```
## 法二、@Resource（name = “teacher”）
```
  @Resource（name = “teacher”）
    private Person person;
```
## 或
```
  @Resource
    private Person teacher;
```

## 法三、@Primary
### 写在teacher类上
### 表示优先注入
```
@Primary
@Component
public class Student implements Person {
    @Override
    public String work() {
        return "Study!";
    }
}
```



#  Springboot 容器的创建、调用bean（让容器外的实例享受容器服务）
## 子接口 Application Context
src/main/java/ioc/util/ContextUtil.java
1. Spring容器是怎么创建的：在启动类（App.java)执行run方法。方法中创建了一个容器，并且当作参数返回。
2.  创建后，怎么在外面拿到容器：(获取后，提供了很多Api)
#### 法一：src/main/java/ioc/util/ContextUtil.java：
```
ContextUtil.getApplicationContext();
```
#### 法二：调用启动类里的get方法(不推荐，不希望启动类里有很多方法)
```
App.getContext();
```


# bean的生命周期、scope
## Scope
1. 默认是单例（可能线程不安全）：一个bean只有一个对象/每次调用（每次访问service）不会反复创建实例，例：容器中永远只有一个web controller
```
@Scope("singleton")
```
### 可以通过打印this判断
2. 多例，注意⚠️：这些实例都是多线程操作的。所以不要再controller里创建数据结构并进行操作
```
@Scope("prototype")
```
## 生命周期
### 法一：（见service2）
```
   @PostConstruct
    //在启动后调用
    private void init() {
        System.out.println("init");
    }

    public String anno() {
        return "service anno";
    }

    @PreDestroy
    //在销毁前调用
    private void clean() {
        System.out.println("clean");
    }
```
### 法二：在启动类App.java
```
 //在启动10秒后，关闭容器
        Thread.sleep(10000);
        context.close();
```
