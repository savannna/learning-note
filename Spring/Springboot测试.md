https://icodewalker.com/blog/123/
# 先配置test的启动类，为在test类中指定启动类配置
> 给上下文@ContextConfiguration(classes = DemoApplication.class)
***
# controller 测试：
@WebMvcTest
- https://www.zhangbj.com/p/1280.html
- https://icodewalker.com/blog/123/ 
- 关于测试bean：https://www.cnblogs.com/dyh004/p/11584129.html
- 手动mock：https://blog.csdn.net/qq_36688143/article/details/9739394
***
# service 测试：
### junit 4
## 方法一：启动整个项目：此方法没有真正隔离，进行了真实数据库
> ### autowire测试类+mockbean
```
//下面两个注解 是为了给auto wire提供上下文
@SpringBootTest(classes = DemoApplication.class, webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@RunWith(SpringJUnit4ClassRunner.class)

public class ManagerServiceTest {

    @Autowired
    private ManagerService managerService;
    
    @MockBean
    ParkingLotMapper parkingLotMapper;

    @Test
    public void testManagerServiceCreateLot() {
        //given
        ParkingLotVO parkingLotVO = new ParkingLotVO("A",4);

        // TODO 手动mock Dao层的方法  让测试的该方法中调用的DAO层，返回永远为true
        when(parkingLotMapper.saveParkingLotInBuilding(any(ParkingLotPO.class))).thenReturn(true);
        when(parkingLotMapper.saveEmptySpot(any(ParkingSpotPO.class))).thenReturn(true);

        //when
        boolean result = managerService.createParkingLot(parkingLotVO.convertVOToDTO());

        Assert.assertTrue(result);
    }


```
## 方法二：不启动上下文：真正隔离 （不启动整个项目）
> ### new测试类+初始化+mock+injectMocks  
> 只用@Runwith就行，因为没有autowire，不用上下文
> https://www.helloworld.net/p/4317936842
```
@RunWith(SpringJUnit4ClassRunner.class)

public class ManagerServiceTest {
    
    @InjectMocks
    ManagerService managerService;
    @Mock
    private ParkingLotMapper parkingLotMapper;

    public ManagerServiceTest() {
        //TODO 初始化写在外面可行，写在setup不可行？？？
        managerService = new ManagerServiceImpl(parkingLotMapper);
    }
    
    @Test
    public void testManagerServiceCreateLot() {
        //given
        ParkingLotVO parkingLotVO = new ParkingLotVO("A",4);

        // TODO 手动mock Dao层的方法  让测试的该方法中调用的DAO层，返回永远为true
        when(parkingLotMapper.saveParkingLotInBuilding(any(ParkingLotPO.class))).thenReturn(true);
        when(parkingLotMapper.saveEmptySpot(any(ParkingSpotPO.class))).thenReturn(true);

        //when
        boolean result = managerService.createParkingLot(parkingLotVO.convertVOToDTO());

        Assert.assertTrue(result);
    }

 
```
***
# Spy与mock
https://sspai.com/post/48245
## 一、mock与mockbean（整个bean的替换）  
> mockbean+**when(某对象.某方法).thenReturn(某对象)**

> mockbean:生成一个 Bean 的替代品装配到类中(会将目标对象的所有方法全部mock)
- 在做单元测试时，如果想要 mock UserRepository 的逻辑，只需要声明一个变量并在上面加上 @MockBean 的注释即可，之后使用 when().thenReturn() 来设定 mock UserRepository 的行为。
- 在运行时 SpringBoot 会扫描到你注释的 mock ，并自动装配到被测试的 controller 里面。这也是和 @Mock 注释不同的地方，后者只能生成一个 Mock 类，但是并不能自动装配到其它类里面。
> @MockBean **只能 mock 本地的代码——或者说是自己写的代码**，对于储存在库中而且又是以 Bean 的形式装配到代码中的类无能为力。

> mock:mock一个类，不自动装配
## 二、spy与spybean（监听一个bean，指定mock某个方法，不影响其他）
> spybean+**doReturn(某对象).when(某对象).某方法**  

> spybean:它不会生成一个 Bean 的替代品装配到类中。而是监听一个真正的 Bean 中某些特定的方法，并在调用这些方法时给出指定的反馈。却不会影响这个 Bean 其它的功能。


