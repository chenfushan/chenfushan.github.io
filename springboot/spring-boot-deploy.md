# Spring Boot 多环境部署

@[toc]

我的目的是两个。
1. 从tomcat中启动
2. 多环境部署
3. 多环境配置

## 1. 修改为从tomcat容器中启动

一共两步, 增加pom配置. 第二步, 继承`SpringBootServletInitializer`类, 重写`SpringApplicationBuilder`方法.

###1.1 增加pom配置

在依赖里增加配置:

```
<!-- marked the embedded servlet container as provided -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-tomcat</artifactId>
            <scope>provided</scope>
        </dependency>
```

在`pom.xml`的`<version>`标签下增加增加war配置:

       <groupId>com.slow.tech</groupId>
        <artifactId>resource</artifactId>
        <version>1.0-SNAPSHOT</version>
        <packaging>war</packaging> //这行是增加的

### 1.2 继承类`SpringBootServletInitializer`

```
@Controller
@EnableAutoConfiguration
@ComponentScan
@SpringBootApplication
public class Application extends SpringBootServletInitializer{
    @Override
    protected SpringApplicationBuilder configure(SpringApplicationBuilder builder) {
        return builder.sources(Application.class);
    }

    public static void main(String[] args) throws Exception {
        SpringApplication.run(Application.class, args);
    }

}
```

这样就可以在tomcat里面启动了.

## 2. 增加远程部署

增加远程`server`里面的远程地址.

```xml
<build>
    <finalName>resource</finalName>
    <plugins>
        <plugin>
            <!-- https://mvnrepository.com/artifact/org.apache.tomcat.maven/tomcat8-maven-plugin -->
            <groupId>org.apache.tomcat.maven</groupId>
            <artifactId>tomcat7-maven-plugin</artifactId>
            <version>2.2</version>

            <configuration>
                <url>http://xx.xx.4.56:8080/manager/text</url>
                <server>slowtech</server>
                <username>slowtech</username>
                <password>slowtech</password>
                <update>true</update>
                <path>/resource</path>
            </configuration>
        </plugin>
    </plugins>
</build>
```

增加完`pom.xml`配置, 配置tomcat的远程配置 `$TOMCAT_HOME/conf/web.xml`.

```xml
<role rolename="admin-gui"/>
  <role rolename="admin-script"/>
  <role rolename="manager-gui"/>
  <role rolename="manager-script" />
  <role rolename="manager-jmx"/>
  <user username="xxxx" password="xxxx" roles="manager-gui,admin-gui"/>
  <user username="xxxx" password="xxxx" roles="manager-script,admin-script"/> //这里是用来上传的权限
```

本地增加`maven`配置`.m2/settings.xml`:
在`servers`标签里增加配置:

```xml
<server>
    <id>slowtech</id>
    <username>xxxxx</username>
    <password>xxxxx</password>
</server>
```

然后就可以远程部署了.

```
部署:
mvn tomcat7:deploy
重新部署:
mvn tomcat7:redeploy
```

## 多环境部署并指定配置

多环境部署需要指定在`deploy`的时候的远程地址, 也就是上面的`<url>${deploy.url}</url>`的值. 方法是通过在`pom.xml`中创建多个`profile` 通过在打包部署的时候, 指定`-P`参数 来为不同的环境进行部署打包.

```xml
<profiles>
    <profile>
        <id>local</id>
        <properties>
            <profiles.active>local</profiles.active>
            <deploy.url>http://localhost:8080/manager/text</deploy.url>
            <username>user_local</username>
            <password>pass_local</password>
        </properties>
        <activation>
            <activeByDefault>true</activeByDefault>
        </activation>
    </profile>
    <profile>
        <id>prod</id>
        <properties>
            <profiles.active>default</profiles.active>
            <deploy.url>http://xxx:8080/manager/text</deploy.url>
            <username>user_default</username>
            <password>pass_default</password>
        </properties>
    </profile>
</profiles>
```

增加这个配置后, 修改上面`build`的配置. 

> 记住这里面的`profiles.active`下面配置有用.


```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.tomcat.maven</groupId>
            <artifactId>tomcat7-maven-plugin</artifactId>
            <version>2.2</version>
            <configuration>
                <url>${deploy.url}</url>
                <server>tomcat</server>
                <path>/sample</path>
                <username>${username}</username>
                <password>${password}</password>
            </configuration>
        </plugin>
    </plugins>
</build>
```

这个时候部署的时候, 要加上`-P`参数.

```
部署本地:
mvn tomcat7:deploy -Plocal
mvn tomcat7:redeploy -Plocal
部署远端:
mvn tomcat7:deploy -Pprod
mvn tomcat7:redeploy -Pprod
```

现在实现了多环境部署, 那么不同环境配置是如何做到的呢? 这个时候要看`resources/config`文件夹下的配置, `application.properties`文件, 这个是所有环境都会使用的文件, 再创建如下两个文件:

```
application-default.properties

application-local.properties
```
发现这两个文件的名字中, 带有环境的参数. 还记得上面小结记住的`profiles.active`么? 这样就可以指定不同环境不同配置了.


