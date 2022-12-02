
# 一个完整的api编写流程（前提：已连接数据库）
> 1. controller 写完  
> 使用 Controller 的时候需要引入 web 包
> 2. service
> 3. entity (实体层)  
> 实体类要实现implements Serializable（序列化接口）
> 4. mapper 数据库映射层 (就是写sql语句的地方) 
> 5. 数据库 
***
# 各部分用途

> - repository==DAO：**代替**了把**数据对象存入List，遍历寻找findByName**等。JPA直接提供了save，find By等等。
> - controller：决定了post/get请求返回的是什么，是页面还是response
> - service:例如创建用户、存入数据库/查找……写在这里
***
# 一、法一：JPA映射  
### **（不用创建表格，自动扫描生成）**
> 配置： implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
1. 依赖引入JPA
2. 开始使用:
- 数据实体类的创建
```
@Entity 注解：用于数据库的实体类均需要加上Entity注解

@Table 注解：顾名思义，这是一个表注解，可在name属性中赋值需要的表名。默认表名与类名相同

@Id : 表示主键ID，如果想让id自增则需要再加上@GeneratedValue(strategy = GenerationType.IDENTITY)注解

@Column 注解：表示这是一个字段

这里，我们希望将数据的创建时间与更新时间交给框架来管理，则我们需要加上@CreationTimestamp和@UpdateTimestamp注解。并将数据类型定义为LocalDateTime。
```
```
package com.yeliheng.test.domain;

import org.hibernate.annotations.CreationTimestamp;
import org.hibernate.annotations.UpdateTimestamp;

import javax.persistence.*;
import java.time.LocalDateTime;

@Entity
@Table(name = "users")
public class UsersDO {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private long id;
    @Column
    private String nickname;
    @Column
    private String phone;
    @Column
    private String password;
    @CreationTimestamp
    private LocalDateTime createDateTime;
    @UpdateTimestamp
    private LocalDateTime updateDateTime;

    public String getNickname() {
        return nickname;
    }

    public void setNickname(String nickname) {
        this.nickname = nickname;
    }

    public String getPhone() {
        return phone;
    }

    public void setPhone(String phone) {
        this.phone = phone;
    }

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }

    public LocalDateTime getCreateDateTime() {
        return createDateTime;
    }

    public void setCreateDateTime(LocalDateTime createDateTime) {
        this.createDateTime = createDateTime;
    }

    public LocalDateTime getUpdateDateTime() {
        return updateDateTime;
    }

    public void setUpdateDateTime(LocalDateTime updateDateTime) {
        this.updateDateTime = updateDateTime;
    }

    public long getId() {
        return id;
    }

    public void setId(long id) {
        this.id = id;
    }
}
```

# 二、法二：mybatis映射（前提：已经建好表了）

https://www.jianshu.com/p/b0cf9c476d7b
> ### 第一步：引入依赖包 
> 在项目的 pom.xml 中添加 MyBatis、MySQL、Junit 、web、JPA的依赖；
``` 
    //配置mybatis 数据源
    implementation group: 'org.mybatis.spring.boot', name: 'mybatis-spring-boot-starter', version: '2.1.3' 
```
***
> ### 第二步：mybatis-config.xml (是全局配置)
> (用来配置数据源及mapper映射文件)  
> (在 main/resources 文件下新建 mybatis-config.xml)  
> 里面的方式都需要接口和xml在统一文件夹下  
> https://www.cnblogs.com/deolin/p/8195565.html
- 两种config方式（就是数据库的敏感信息是否抽成变量）：https://blog.csdn.net/Evankaka/article/details/45647401
- 抽成变量的写法/全流程编写：https://blog.csdn.net/junR_980218/article/details/124145948?spm=1001.2101.3001.6650.17&utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromBaidu%7ERate-17-124145948-blog-124429866.pc_relevant_3mothn_strategy_recovery&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromBaidu%7ERate-17-124145948-blog-124429866.pc_relevant_3mothn_strategy_recovery&utm_relevant_index=22
```
<?xml version="1.0" encoding="UTF8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">

<!--核心配置文件-->
<configuration>

   <!--引入外部文件-->
    <properties resource="db.properties"/>

    <settings>
        <!--开启驼峰命名转换-->
        <setting name="mapUnderscoreToCamelCase" value="true"></setting>
        <!--开启二级缓存-->
        <setting name="cacheEnabled" value="true"/>
        <!--开启默认的日志-->
        <setting name="logImpl" value="STDOUT_LOGGING"/>

    </settings>

    <!--给实体类起别名-->
    <typeAliases>
        <typeAlias type="com.li.pojo.Student" alias="student"/>
    </typeAliases>

    <!--配置环境-->
    <environments default="development">
        <environment id="development">

            <transactionManager type="JDBC"/><!--事务管理——jdbc-->
            <!--连接数据库 这里通过引入外部文件的方式-->
            <dataSource type="POOLED">
            <property name="driver" value="${driver}"/>
                <property name="url" value="${url}"/>
                <property name="username" value="${username}"/>
                <property name="password" value="${password}"/>
           </dataSource>

        </environment>

    </environments>

    <!--mapper映射-->
    <mappers>
        <mapper resource="com/thoughtworks/demo/StudentMapper.xml"/>
    </mappers>

</configuration>

```
resources文件下 创建db.properties
```
#连接数据库信息 根据自己情况改写
driver=com.mysql.cj.jdbc.Driver
url=jdbc:mysql://localhost:3306/mybatis?useSSL=true&useUnicode=true&characterEncoding=UTF-8
username=root
password=123456

```
***
> ### 第三步：写实体类
> 1. 实体类（同JPA)  
> lombok与mybatis: https://codeantenna.com/a/aNBzS6L78n
> 2. 接口DAO（增删改查) 
> 写法：http://www.mydlq.club/article/122/
  ***
> ### 第四步：写接口+SQL语句
### 方法一：一个接口Mapper对应一个Mapper.xml    

- **位置一**：resources/mapper/…….xml(要额外在yml里指定路径)
- Mybatis mapper接口与xml文件路径分离：https://www.cnblogs.com/deolin/p/8195565.html
```
# 指定xml目录，告诉MyBatis去哪里扫描
mybatis:
  mapper-locations: classpath:mapper/*.xml
```
## *！！！！！！ 注意：文件路径*  
> application.yml中的文件路径是为了gradle打包用的  
> 因此路径不应该写成:src/main/resources/mapper/UserMapper.xml。  
> 应该写成:mapper/UserMapper.xml  
> （因为gradle会自动寻找resources，再写一遍的话就是按照src/main/resources/src/main/resources/路径寻找了，查无此文件）
- **位置二不推荐**：和接口在一个包下 即直接写dao层** (要额外在build.gradle中加配置，防止打包丢掉)
```
processResources {
    from('src/main/java') {
        include '**/*.xml'
    }
}
```
- ### 编写映射文件（实体类与表的映射)  
文件内容：书写原则  
https://blog.csdn.net/qq_43437874/article/details/117734556  
例子：https://blog.csdn.net/junR_980218/article/details/124145948?spm=1001.2101.3001.6650.17&utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromBaidu%7ERate-17-124145948-blog-124429866.pc_relevant_3mothn_strategy_recovery&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromBaidu%7ERate-17-124145948-blog-124429866.pc_relevant_3mothn_strategy_recovery&utm_relevant_index=22
```
<?xml version="1.0" encoding="UTF8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="com.li.dao.StudentMapper"><!--namespace指向 接口的配置文件***.xml-->
    <!--添加二级缓存-->
    <cache
            eviction="FIFO"
            flushInterval="60000"
            size="512"
            readOnly="true"
    /><!--添加全局缓存 采用FIFO方式清理缓存  flushInterval 每隔60秒清理缓存-->

<select id="queryStudent" resultType="student">
        <!--id指的是 接口中该方法名
    resultType 指的是实体类的com.li.pojo.Student
    这里写student是因为在mybatis中给这个实体类起了个别名-->

        select * from li.student
 </select>

<select id="queryStudentId" parameterType="student" resultType="student">
    <!--通过id来查询-->
        select * from li.student
        where id=#{id};
    </select>
    <!--动态查询-->
<select id="queryStudentDong" parameterType="map" resultType="student">
            select *
            from li.student
            where 1=1
            <if test="name != null">
                      and name = #{name}
            </if>
</select>

<insert id="addStudent" parameterType="student">
         <!--添加数据-->
       insert into li.student(id,name,tid,grade,sex)
       values(#{id},#{name},#{tid},#{grade},#{sex})
</insert>


 <update id="updateStudent" parameterType="student">

            <!--通过id来修改 更新数据-->
            update li.student
            set
            name =#{name},
            tid=#{tid},
            grade=#{grade},
                sex=#{sex}
            where id=#{id};
</update>

<delete id="deleteStudentId" parameterType="_int">
        <!--通过id删除数据-->
                delete
                from li.student
                where id=#{id};

 </delete>
</mapper>

```
### 方法二：直接在接口的注解里写sql(不用映射文件了)  
https://zhuanlan.zhihu.com/p/417570930
