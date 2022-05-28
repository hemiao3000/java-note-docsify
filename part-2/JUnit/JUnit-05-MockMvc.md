# <font color="orange">MockMvc 对 Spring MVC 进行测试</font>

## 1. @WebAppConfiguration 注解

我们将使用 Spring 的 **@WebAppConfiguration** 注释来测试 web 控制器。@WebAppConfiguration 是一个类级注释，它将加载特定于 web 的 ApplicationContext，即 WebApplicationContext 。

> 虽然 @WebAppConfiguration 从命名上并未体现出与测试的关系，但是在它的注释中明确写道： the ApplicationContext loaded for an integration test should be a WebApplicationContext.

@WebAppConfiguration 所指定的 web application 的目录的默认路径是 `src/main/webapp` 。<small>当然，你可以用自己指定的值来覆盖这个默认值。</small>

另外，**@WebAppConfiguration 必须与 @ContextConfiguration 结合使用。**<small>@ContextConfiguration 则是用于指定Spring 配置文件或配置类的。</small>

```java
@RunWith(SpringJUnit4ClassRunner.class)
@WebAppConfiguration
@ContextConfiguration(...) 
public class TestMockMvc {
    ...
}
```

## 2. MockMvc 

Spring 3.2 之后出现了 `org.springframework.test.web.servlet.MockMvc` 类，对 Restful 风格的 Spring MVC 单元测试进行支持。

```java
package com.xja.test;

import com.jiaoyiping.baseproject.privilege.controller.MeunController;
import com.jiaoyiping.baseproject.training.bean.Person;
import junit.framework.Assert;
import org.junit.Before;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.MediaType;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;
import org.springframework.test.context.web.WebAppConfiguration;
import org.springframework.test.web.servlet.MockMvc;
import org.springframework.test.web.servlet.ResultActions;
import org.springframework.test.web.servlet.setup.MockMvcBuilders;
import org.springframework.web.servlet.ModelAndView;

import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.*;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.*;

@RunWith(SpringJUnit4ClassRunner.class)
@WebAppConfiguration
@ContextConfiguration(locations = {
    "classpath:spring/spring-*.xml" 
}) 
public class TestMockMvc {

    // @Autowired
    // private org.springframework.web.context.WebApplicationContext context;

    /*
     * SSM 项目中我们需要亲自动手创建 MockMvc 对象，在 spring boot 中可以使用 @WebMvcTest 注解，就可以直接 @Autowired 了。
     */
    MockMvc mockMvc;

    @Before
    public void before() {
        // mockMvc = MockMvcBuilders.webAppContextSetup(context).build();
        mockMvc = MockMvcBuilders.standaloneSetup(new HelloController()).build();
    }


    /**
     * 打印全部信息
     */
    @Test
    public void testDemo() throws Exception {
        mockMvc.perform(
            get("/dept.do")
            .param("deptno", "10")
        ).andDo(print());
    }

    /**
     * 测试普通请求
     */
    @Test
    public void testDemo1() throws Exception {
        mockMvc.perform(
            get("/dept.do")
            .param("deptno", "10")
        )
        .andExpect(status().isOk())
        .andExpect(view().name("hello"))
        .andExpect(model().attribute("name", "tom"))
        .andExpect(model().attribute("age", 20));
    }

    /**
     * 测试 Rest 请求
     */
    @Test
    public void testDemo2() throws Exception {
        mockMvc.perform(
                get("/dept/10")
        )
        .andExpect(status().isOk())
        .andExpect(content().contentType(MediaType.APPLICATION_JSON))
        .andExpect(jsonPath("$.id").value(10))
        .andExpect(jsonPath("$.name").value("Testing"))
        .andExpect(jsonPath("$.location").value("武汉"))
        .andDo(print());
    }

}
```