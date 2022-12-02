# @Component 注入service为空
- https://blog.csdn.net/weixin_47547044/article/details/122340110  
- https://zakariyya.github.io/2018/08/23/backEnd-spring-springboot-Component%E4%B8%8B-Autowired%E7%9A%84%E6%B3%A8%E5%85%A5%E4%B8%BAnull/
***
# POJO
- https://yanyan.blog.csdn.net/article/details/122638817?spm=1001.2101.3001.6661.1&utm_medium=distribute.pc_relevant_t0.none-task-blog-2%7Edefault%7ECTRLIST%7ERate-1-122638817-blog-109252347.pc_relevant_3mothn_strategy_recovery&depth_1-utm_source=distribute.pc_relevant_t0.none-task-blog-2%7Edefault%7ECTRLIST%7ERate-1-122638817-blog-109252347.pc_relevant_3mothn_strategy_recovery&utm_relevant_index=1
- https://juejin.cn/post/6952848675924082718
- 前端传入参数：VO➡️VO转化为DTO，DTO传给service➡️service根据DTO结构创建一个DO，调用DO的方法完成业务
VO：（get 返回VO对象）（传入参数？？是什么呢）表现对象：这里的VO只包含前端需要展示的数据即可，对于前端不需要的数据，比如数据创建和修改的时间等字段，出于减少传输数据量大小和保护数据库结构不外泄的目的，
DTO：（add 传入DTO）用于展示层（Controller）和服务层（Service）之间的数据传输对象。
***
# bean实例化和注入流程
> 找标有@Service等的 bean名字
1. 在某一时刻Spring调用了Bean工厂的getBean(beanName)方法。beanName可能是simpleController,或者simpleService，simpleDao，顺序没关系（因为后面会有依赖关系的处理）。我们假设simpleController吧

2. getBean方法首先会调用Bean工厂中定义的getSingleton(beanName)方法，来判断是否存在该名字的bean单例，如果存在则返回，方法调用结束（spring默认是单例，这样可以提高效率）

3. 否则，Spring会检查是否存在父工厂，如果有则返回，方法调用结束

4. 否则，Spring会检查bean定义（BeanDefinition实例，用来描述Bean结果，component-scan扫描后，就是将beanDefinition实例放入Bean工厂，此时还没有被实例化）是否有依赖关系，如果有，执行1）步，获取依赖的bean实例
> 尝试创建实例
5. 否则，Spring会尝试创建这个bean实例，创建实例前，Spring会检查调用的构造器，并实例化该Bean，（通过Constructor.newInstance(args)进行实例化）
> 填充实例属性
6. 实例化完成后，Spring会调用Bean工厂的populateBean方法来填充bean实例的属性，也就是自动装配。populateBean方法便是调用了BeanPostProcessor实例来完成属性元素的自动装配工作
> 装配autowire
7. 在元素装配过程中，Spring会检查被装配的属性是否存在自动装配的其他属性，然后递归调用getBean方法，知道所有@Autowired的元素都被装配完成。如在装配simpleController中的simpleService属性时，发现SimpleServiceImpl实例中存在@Autowired属性simpleDao,然后调用getBean(simpleDao)方法，同样会执行1）----7）整个过程。所有可以看成一个递归过程。

8. 装配完成后，Bean工厂会将所有的bean实例都添加到工厂中来。
