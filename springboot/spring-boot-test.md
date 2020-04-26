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

### 测试Controller

Controller是一类特殊的bean，我们无法像测试上文那样对其单独实例化。
Spring提供了特定的注解，配置用于测试Controller的上下文环境。就是`WebMVCTest`
但是这里面有些需要注意的, 就是:

> @SpringBootTest is the general test annotation. If you're looking for something that does the same thing prior to 1.4, that's the one you should use. It does not use slicing at all which means it'll start your full application context and not customize component scanning at all.
> 
> @WebMvcTest is only going to scan the controller you've defined and the MVC infrastructure. That's it. So if your controller has some dependency to other beans from your service layer, the test won't start until you either load that config yourself or provide a mock for it. This is much faster as we only load a tiny portion of your app. This annotation uses slicing.
>
> Referfence Doc: https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#test-auto-configuration

所以在WebMvcTest的时候, 很多Spring boot的auto configuration是没有引入的. 如果你使用了并且依赖了. 那么就要自己引入该configuration或者提供一个mock 这样才可以. 因为除了SpringBootTest其他的这些注解都是 slicing 的.

```java
package com.fushan.cfs;

import com.fushan.cfs.home.IndexController;
import com.fushan.cfs.repository.UserRepository;
import com.fushan.cfs.service.TestService;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.mockito.Mockito;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.data.redis.DataRedisTest;
import org.springframework.boot.test.autoconfigure.web.servlet.WebMvcTest;
import org.springframework.boot.test.mock.mockito.MockBean;
import org.springframework.data.redis.core.StringRedisTemplate;
import org.springframework.http.MediaType;
import org.springframework.test.context.TestPropertySource;
import org.springframework.test.context.junit4.SpringRunner;
import org.springframework.test.web.servlet.MockMvc;
import org.springframework.test.web.servlet.MvcResult;
import org.springframework.test.web.servlet.request.MockMvcRequestBuilders;
import org.springframework.test.web.servlet.result.MockMvcResultMatchers;
import org.springframework.transaction.PlatformTransactionManager;

/**
 * Created by alps on 2020/1/14.
 */
@RunWith(SpringRunner.class)
@WebMvcTest(IndexController.class)
@TestPropertySource({ "/application-test.properties"})
public class ControllerTest {

    @Autowired
    private MockMvc mockMvc;

    //controller使用了这个方法
    //但是因为是测试controller就不测试下一层
    //mock掉
    @MockBean
    private TestService testService;

    //必须手动mock掉 因为我项目使用了 并且是使用的auto-configration
    @MockBean
    private StringRedisTemplate stringRedisTemplate;

    //同上
    @MockBean
    private UserRepository userRepository;

    //同上
    @MockBean
    private PlatformTransactionManager platformTransactionManager;


    @Test
    public void normalHello() throws Exception {

    	//void的方法测试
        Mockito.doNothing().when(testService).testCache();

        MvcResult mvcResult = mockMvc.perform(MockMvcRequestBuilders.get("/hello").contentType(
                MediaType.APPLICATION_JSON_UTF8)).andExpect(MockMvcResultMatchers.status().isOk()).andReturn();

        System.out.println(mvcResult.getResponse().getContentAsString());
    }
}
```

### 测试持久化层



## Spring Boot 集成测试

集成测试就是单元测试的组装组合.

### 从Controller开始测试




### 从Service开始测试

```java
/**
 * unit test service
 * Created by alps on 2020/1/16.
 */
@SpringBootTest
@RunWith(SpringRunner.class)
public class TestServiceTest {
    @Autowired
    private TestService testService;

    @Test
    public void normalTestHandle(){
        boolean result = testService.testHandle();
        // AssertUtil 是我自己写的用于检测的类
        AssertUtil.isTrue(result);
    }
}
```








