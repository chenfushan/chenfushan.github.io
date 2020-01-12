# Spring Boot 测试

通常测试会使用集成测试和单元测试两个部分. 

- 单元测试
单元测试是指对软件中的最小可测试单元进行检查和验证。如C语言中单元指一个函数，Java里单元指一个类。

- 集成测试
集成测试是单元测试的逻辑扩展。最简单的形式是：把两个已经测试过的单元组合成一个组件，测试它们之间的接口。集成是指多个单元的聚合，许多单元组合成模块。

## Spring Boot 单元测试

一般来说, 单元测试要控制在当前一个极小范围. spring boot应该控制在一个类范围内. 这样测试就比较合适.

但是类之间都是有很多调用关系的, 例如:

```
Class A {
	void method_A(){
	}
}

Class B {
	void method_B(){
		A.method_A();
	}
}
```

这个时候要测试`Class B::method_B`方法 那么直接测试就超出了一个类的原则. 容易让单元屙屎变得不可控. 这样如果`Class A::method_A`修改了, 反而会导致`Class B`的单元测试不通过. 所以需要通过`Mock`来去对其他类和方法进行模拟代替. 达到测试的时候, 依赖不相关.

### 真实代码案例

假设要测试的代码是下面这个`Service`类

```java
@Service("testService")
public class TestServiceImpl implements TestService {

    private static final Logger LOGGER = LoggerFactory.getLogger(TestServiceImpl.class);

    @Resource
    private CacheComponent cacheComponent;

    @Override
    public String testCache() {
//        cacheComponent.set("alps", "chenfushan");
        String v = cacheComponent.get("alps");
        if (LOGGER.isInfoEnabled()) {
            LOGGER.info("cache value:" + v);
        }
        return v;
    }

    @Override
    public void testInsertCache() {
        cacheComponent.set("alps", "chenfushan", 2000);
        if (LOGGER.isInfoEnabled()) {
            LOGGER.info("set cache key: alps. value:chenfusha");
        }
    }
}
```

要测试方法: `testCache` 这个时候, 需要对`cacheComponent.get`这个方法进行`Mock`.

```java
@SpringBootTest
@RunWith(SpringRunner.class)
public class ApplicationTest {

    @Autowired
    private TestService testService;

    @MockBean
    private CacheComponent cacheComponent;

    @Before
    public void before(){

    }

    @After
    public void after(){

    }

    @Test
    public void test(){
        Mockito.when(cacheComponent.get(Mockito.anyString())).thenReturn("cache success.");
        String v = testService.testCache();
        Assertions.assertThat(v).isEqualTo("cache success.");
    }

    @Test
    public void testInsertCache(){
        testService.testInsertCache();
        Mockito.verify(cacheComponent).set("alps", "chenfushan", 2000);
    }

}
```

这里面的`test`方法是测试一个查询, 所以对`cacheComponent.get`方法进行了mock 这样`testService`类就不再依赖`cacheComponent`就可以测试了.

`Mockito.when(cacheComponent.get(Mockito.anyString())).thenReturn("cache success.");` 这里是对一个方法进行mock来达到定向返回的目的.

然后下面的插入, 则是对关键的插入方法节点进行验证.


## Spring Boot 集成测试