# Java Serialization with Jackson Demo
https://github.com/gtb-2022-li-jing2/learning-java-serialization-with-jackson-withcomments

Demo for Java Serialization with Jackson

## 模块说明

- raw-jackson-demo：原生Jackson的完整使用示例
- spring-jackson-demo：Spring下Jackson的完整使用示例

另：blank结尾的模块表示不完整的示例，用于课上演示


## 文件说明
# 关于序列化
1. 原生Jason使用：JacksonUsageDemo文件 raw-jackson-demo-blank/src/main/java/
2. spring中Jason使用：spring-jackson-demo-blank/src/main/controller/JacksonUsageController.java
3. @RequestBody接收对象时自动用Jackson反序列化：spring-jackson-demo-blank/src/main/java/controller/DemoController.java

# 关于前端后端字段不一致的各种情况
4. JSON比JavaBean对象多字段：@JsonIgnoreProperties可以忽略指定字段，也可以忽略本类所有未知字段：spring-jackson-demo-blank/src/main/java/controller/annotation/FieldsMoreController.java
5. JSON比JavaBean少字段，在序列化时忽略多余字段:@JsonIgnore忽略类中的某个属性/ 用@JsonIgnoreType忽略整个类:spring-jackson-demo-blank/src/main/java/controller/annotation/FieldsLessController.java
6. 字段名字不一致：在该属性前面加@JsonProperty 修改字段名：spring-jackson-demo-blank/src/main/java/controller/annotation/FieldsNameDoseNotMatch.java
7. 结构不一样：后端是按照类的层级关系 前端希望没有层级关系：在那个出现层级关系的属性上加@JsonUnwrapped ：spring-jackson-demo-blank/src/main/java/controller/annotation/StructureDoesNotMatch.java
8. json中枚举是数字形式：后端是 male,female 前端变成：1，2:spring-jackson-demo-blank/src/main/java/controller/CustomSerializationController.java
9. 时间格式：@JsonFormat（parttern="adghgfbfgf"）:spring-jackson-demo/src/main/java/controller/annotation/TimeFormatController.java

# 自定义序列化
10. spring-jackson-demo/src/main/java/com/thoughtworks/capability/gtb/controller/CustomSerializationController.java
