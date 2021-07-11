# Swagger

- 号称世界上最流行的api框架
- RestFul api文档在线自动生成工具=>api文档与api定义同步更新
- 直接运行，可以在线测试api接口
- 支持多种语言

在项目中使用Swagger需要springbox

- swagger2
- ui

## SpringBoot集成Swagger

导入依赖

- swagger2
- swagger-ui

```xml
        <!-- https://mvnrepository.com/artifact/io.springfox/springfox-swagger2 -->
        <dependency>
            <groupId>io.springfox</groupId>
            <artifactId>springfox-swagger2</artifactId>
            <version>2.9.2</version>
        </dependency>
        <!-- https://mvnrepository.com/artifact/io.springfox/springfox-swagger-ui -->
        <dependency>
            <groupId>io.springfox</groupId>
            <artifactId>springfox-swagger-ui</artifactId>
            <version>2.9.2</version>
        </dependency>
```

使用默认配置

```java
package com.zhongmingyuan.config;

import org.springframework.context.annotation.Configuration;
import springfox.documentation.swagger2.annotations.EnableSwagger2;

@Configuration
@EnableSwagger2
public class SwaggerConfig {

}

```

之后，即可访问 localhost:8080/swagger-ui.html 页面

## 配置Swagger

```java
package com.zhongmingyuan.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import springfox.documentation.service.ApiInfo;
import springfox.documentation.service.Contact;
import springfox.documentation.spi.DocumentationType;
import springfox.documentation.spring.web.plugins.Docket;
import springfox.documentation.swagger2.annotations.EnableSwagger2;

import java.util.ArrayList;

@Configuration
@EnableSwagger2
public class SwaggerConfig {
    //配置Swagger的Docket的Bean实例
    @Bean
    public Docket docket() {
        return new Docket(DocumentationType.SWAGGER_2).apiInfo(apiInfo());
    }

    //配置Swagger信息 => apiInfo
    private ApiInfo apiInfo() {
        //作者信息
        Contact contact = new Contact("zhongmingYuan", "null", "13556018510@163.com");
        return new ApiInfo(
                "HelloSwagger",
                "use Swagger",
                "v1.0",
                "null",
                contact,
                "Apache 2.0",
                "http://wwww.apache.org/licenses/LICENSE-2.0",
                new ArrayList()
        );
    }
}

```

