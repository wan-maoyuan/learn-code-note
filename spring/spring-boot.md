## 使用 ApplicationRunner 或 CommandLineRunner

- 如果在SpringApplication启动后需要运行一些特定的代码，则可以实现ApplicationRunner或CommandLineRunner接口。两个接口以相同的方式工作，并提供一个run方法,该方法在SpringApplication.run(…)完成之前被调用。

- CommandLineRunner接口以简单的字符串数组提供对应用程序参数的访问，而ApplicationRunner使用前面讨论的ApplicationArguments接口。以下示例显示使用run方法的CommandLineRunner：

```java
import org.springframework.boot.*;
import org.springframework.stereotype.*;

@Component
public class MyBean implements CommandLineRunner {

	public void run(String... args) {
		// Do something...
	}

}
```

## Spring Boot 使用外部化配置

- 属性值可以使用@Value注解直接注入到您的 bean 中，可以通过 Spring 的Environment抽象访问.

- 也可以是绑定到结构化对象到@ConfigurationProperties。

> 默认情况下，SpringApplication将任何命令行选项参数(即以--开头的参数，例如--server.port=9000)转换为property并将其添加到 Spring Environment。如前所述，命令行属性始终优先于其他属性源。
如果您不想将命令行属性添加到Environment，则可以使用SpringApplication.setAddCommandLineProperties(false)禁用它们。

### 应用程序属性文件

> SpringApplication从以下位置的application.properties文件中加载属性，并将它们添加到 Spring Environment中：该列表按优先级排序(在列表较高位置定义的属性会覆盖在较低位置定义的属性)

- 当前目录的/config子目录
- 当前目录
- Classpath/config包
- Classpath 根

### 属性中的占位符

```java
app.name=MyApp
app.description=${app.name} is a Spring Boot application
```



























