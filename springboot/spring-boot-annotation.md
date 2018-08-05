# Spring Boot Bean注解

Spring Boog中有很多注解, 这里只讲比较常用的. `@Component, @Service, @Controller, @RestController, @Autowired, @Resource, @Repository` 这些注解都是项目中常用的注解.

## Component注解

这个注解是作为常用的注解, 没有太具体的分类. 但是在项目中, 约定大于配置, 所以我常用它在组件中. 所谓组件就是常用的组件接口. 在业务层之下.

> 举个例子: 一个空调.
> 假如我们需要打开空调来制冷. 那么需要以下内容来构建整个系统.
> 1. Controller 层负责接收指令, 做一些必要的参数校验, 比如空调是否存在等等.
> 2. Service 层负责业务逻辑, 有个空调的Manager提供: 打开空调->调模式到制冷 这个接口.
> 3. Component 层负责组件逻辑, 有个空调的Component提供两个接口: 打开空调 & 调整模式

上面的样例我们可以很清楚的看到, 一个系统越靠近底层, 所做的事情越明确, 越单一. Component就是提供组件接口, 由业务层调用.

## Service注解

这个注解是作为业务层的逻辑代码, 注解到类上, 会自动注册到Spring容器中, 由Spring管理类的声明周期. 在注解的时候可以指定管理实例化的名字.

```java
@Service
class UserService{
	private String name;

	public String getName(){
		return this.name;
	}

	public void setName(String name){
		this.name = name;
	}
}
```

这样一个类,就成为了一个业务层的`service`. 可以向`controller`提供服务了.

## Controller & RestController

这两个注解都是`Controller`层使用的, 他们的不同很容易能看到的.

首先我们查看`@Controller`注解的代码:

```java
package org.springframework.stereotype;

import java.lang.annotation.Documented;
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Component
public @interface Controller {
    String value() default "";
}
```

然后看一下`@RestController`的代码:

```java
package org.springframework.web.bind.annotation;

import java.lang.annotation.Documented;
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;
import org.springframework.stereotype.Controller;

@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Controller
@ResponseBody
public @interface RestController {
    String value() default "";
}

```

这就很容易看到了, 所谓的`@RestController`就是一个`@Controller`的封装版本, 使用`@RestController`能够少加一些注解. 都是控制层控制的.

## Autowired & Resource注解

这两个注解放在一起的原因是因为这两个注解都是把声明为服务的类, 注入到其他的类中, 当然被注入的类, 需要也是由spring-boot管理的类. 那这两个的区别在于哪里呢. 在于注解的方式, 自动注解分为两种方式: `byName, byType`两种.

- byName

这种注解方式是说通过名字的方式注入到当前的类中. 

- byType

这种注解方式是说通过类型的方式注入到当前的类中.

如果没看懂, 下面写个样例.

```java
public interface UserServie{
	void say();
}

//@Service("userService")
@Service
public class UserServieImpl implements UserServie{
	@Override
	public void say(){
		//do something
	}
}


@Controller
public class UserController{
	@Resource("userServieImpl")
	//@Autowired
	private UserServie userService;

	public void indexController(){
		userService.say();
	}
}
```

上面这里声明了一个Service的类, 然后由Controller注入UserService进行调用. 先说上面的Service注解, 如果有参数, 则是声明这个bean的名字. 如果不声明则自动生成bean的名字. 生成规则: 首字母小写的类名字. 如果不是驼峰命名则是类名.

注入的时候, 使用`@Autowired`可以注入, 会找到对应类型的类, 或者子类实现去生成. 如果找不到则报错. 如果对应子类有多个`@Service`注解, 也会报错. 除非使用: `@Qualifier("class name")`来指定对应的`Service`. 或者使用`@Resource` 这个注解是通过名字来注入的. 找不到则通过Type来注入. 再找不到才报错.


## Repository注解

这个注解也是一个特殊的注解, 用于声明数据持久化层的内容. 例如DAO.

```java
@Repository(value="userDao")
public class UserDaoImpl extends BaseDaoImpl<User> {

}
```

这个内容等需要使用数据层的时候, 就需要掌握了. 后面在软件架构的文章中我会详细讲述这个层次.








