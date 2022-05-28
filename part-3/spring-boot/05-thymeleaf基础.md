# <font color="orange">JSP 的替代者：Thymeleaf</font>

## 1. Thymeleaf 简介

Thymeleaf 旨在提供一个优雅的、高度可维护的创建模板的方式。为了实现这一目标，Thymeleaf 建立在自然模板的概念上，将其逻辑注入到模板文件中，**不会影响模板设计原型**，从而改善了设计的沟通，弥合了设计和开发团队之间的差距。

Thymeleaf 从设计之初就遵循 Web 标准——特别是 HTML 5 标准，如果需要，Thymeleaf 允许创建完全符合 HTML 5 验证标准的模板。 

Spring Boot 体系内推荐使用 Thymeleaf 作为前端页面模板，并且 Spring Boot 2.0 中默认使用 Thymeleaf 3.0，性能提升幅度很大。

与其他的模板引擎<small>（JSP、Velocity、FreeMarker）</small>相比较，它有如下三个极吸引人的特点：

1. Thymeleaf 可以在浏览器查看页面的静态效果。

2. Thymeleaf 开箱即用的特性。它支持标准方言和 Spring 方言，可以直接套用模板实现 JSTL、OGNL 表达式效果。

4. Thymeleaf 方便与 SpringMVC 集成。


对比 Velocity、FreeMarker 与 Thymeleaf 打印出同一条消息：

```html
Velocity: <p>$message</p>
FreeMarker: <p>${message}</p>
Thymeleaf: <p th:text="${message}">Hello World!</p>
```

上面我们可以看出来 Thymeleaf 的作用域在 HTML 标签内，类似标签的一个属性来使用，这就是它的特点。


## 2. 快速上手

### 2.1 pom.xml

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
```

### 2.2 application.properties

```properties
spring.thymeleaf.cache=false

logging.level.root=INFO
logging.level.xxx.yyy.zzz=DEBUG
logging.pattern.console=${CONSOLE_LOG_PATTERN:\
  %clr(${LOG_LEVEL_PATTERN:%5p}) \
  %clr(|){faint} \
  %clr(%-40.40logger{39}){cyan} \
  %clr(:){faint} \
  %m%n${LOG_EXCEPTION_CONVERSION_WORD:%wEx}}
```

关闭 Thymeleaf 的缓存。不然在开发过程中修改页面不会立即生效需要重启，生产可配置为 **true** 。

### 2.3 一个简单的页面

```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8"/>
    <title>Hello</title>
</head>
<body>
    <h1 th:text="${message}">Hello World</h1>
    </body>
</html>
```

所有使用 Thymeleaf 的页面『**必须**』在 HTML 标签声明 Thymeleaf：

```html
<html xmlns:th="http://www.thymeleaf.org">
```

表明页面使用的是 Thymeleaf 语法。

### 2.4 Controller

```java
@Controller
public class HelloController {

    @RequestMapping("/")
    public String index(ModelMap map) {
        map.addAttribute("message", "http://www.baidu.com");
        return "hello";
    }

}
```

## 3. 常用语法

### 3.1 赋值、字符串拼接

- 赋值和拼接：

  ```html
  <p th:text="${userName}">我是默认值</p>
  <span th:text="'Welcome to our application, ' + ${userName} + '!'">我是默认值</span>
  ```

- 字符串拼接还有另外一种简洁的写法（推荐）：

  ```html
  <span th:text="|Welcome to our application, ${userName}!|">我是默认值</span>
  ```

- string.html 页面：

  ```html
  ...
  <body>
    <div >
      <h1>text</h1>
      <p th:text="${userName}">我是默认值</p>
      <span th:text="'Welcome to our application, ' + ${userName} + '!'">我是默认值</span>
      <br/>
      <span th:text="|Welcome to our application, ${userName}!|">我是默认值</span>
    </div>
  </body>
  ```

- 后端传值：

  ```java
  @RequestMapping("/string")
  public String string(ModelMap map) {
      map.addAttribute("userName", "baidu");
      return "string";
  }
  ```

### 3.2 条件判断 If/Unless

Thymeleaf 中使用 **th:if** 和 **th:unless** 属性进行条件判断，在下面的例子中， `<a>` 标签只有在 **th:if** 中条件成立时才显示：

- 示例：
 
  ```html
  <a th:if="${flag == 'yes'}" 
     th:href="@{http://www.163.com/}">163</a>

  <a th:unless="${flag != 'no'}" 
     th:href="@{http://www.baidu.com/}">baidu</a>
  ```

  **th:unless** 与 **th:if** 恰好相反，只有表达式中的条件立，才会显示其内容。

- 页面 if.html：

  ```html
  <body>
    <div >
      <h1>If/Unless</h1>
      <a th:if="${flag == 'yes'}" 
         th:href="@{http://www.163.com/}">163</a><br/>
      <a th:unless="${flag != 'no'}" 
         th:href="@{http://www.baidu.com/}">baidu</a>
    </div>
  </body>
  ```

- 后端传值：

  ```java
  @RequestMapping("/if")
  public String ifunless(ModelMap map) {
      map.addAttribute("flag", "yes");
      return "if";
  }
  ```

### 3.3 for 循环

for 循环在我们项目中使用的频率太高，一般结合前端的表格来使用。


- 在后端定义一个列表，以键 `users` ，传递到前端：

  ```java
  @RequestMapping("/list")
  public String list(ModelMap map) {
      User user1 = new User("大牛",12,"123456");
      User user2 = new User("小牛",6,"123563");
      User user3 = new User("纯洁的微笑",66,"666666");

      List<User> list = new ArrayList<User>();
      list.add(user1);
      list.add(user2);
      list.add(user3);

      map.addAttribute("users", list);

      return "list";
  }
  ```

- list.html 进行数据展示：

  ```html
  ...
  <body>
    <div >
      <h1>for 循环</h1>
      <table>
        <tr th:each="user,iterStat : ${users}">
          <td th:text="${user.name}">neo</td>
          <td th:text="${user.age}">6</td>
          <td th:text="${user.pass}">213</td>
          <td th:text="${iterStat.index}">index</td>
        </tr>
      </table>
    </div>
  </body>
  ```

  iterStat 称作状态变量，属性有：

  ```
      属性 | 说明 
     index | 当前迭代对象的 index （从 0 开始计算）
     count | 当前迭代对象的 index （从 1 开始计算）
      size | 被迭代对象的大小
   current | 当前迭代变量
  even/odd | 布尔值，当前循环是否是偶数/奇数（从 0 开始计算）
     first | 布尔值，当前循环是否是第一个
      last | 布尔值，当前循环是否是最后一个
  ```

- 页面展示效果

  ```
  for 循环

  大牛 12 123456 0
  小牛 6 123563 1
  纯洁的微笑 66 666666 2
  ```

## 4. URL 处理

需要特别注意的是 Thymeleaf 对于 URL 的处理是通过语法 **@{...}** 来处理的。如果需要 Thymeleaf 对 URL 进行渲染，那么务必使用 **th:href**、**th:src** 等属性。

- 示例：

  ```html
  <a th:href="@{http://www.baidu.com/{xxx}(xxx=${type})}">link1</a>
  <a th:href="@{http://www.baidu.com/{pageId}/can-use-springcloud.html(pageId=${pageId})}">view</a>
  <form th:action="@{/order}">
  </form>
  ```

- 几点说明：

  上例中 URL 最后的 `(pageId=${pageId})` 表示将括号内的内容作为 URL 参数处理，该语法避免使用字符串拼接，大大提高了可读性；

  `@{...}` 表达式中可以通过 `{pageId}` 访问 Context 中的 `pageId` 变量；

  `@{/order}` 是 Context 相关的相对路径，在渲染时会自动添加上当前 Web 应用的 Context 名字，假设 context 名字为 app，那么结果应该是 `/app/order` 。


- 页面内容：

  ```html
  ...
  <body>
    <div >
      <h1>URL</h1>
      <a th:href="@{http://www.baidu.com/{type}(type=${type})}">link1</a>
      <br/>
      <a th:href="@{http://www.baidu.com/{pageId}/can-use-springcloud.html(pageId=${pageId})}">view</a>
      <br/>
      <div th:style="'background:url(' + @{${img}} + ');'">
          <br/><br/><br/>
      </div>
    </div>
  </body>
  ```

- 后端程序：

  ```java
  @RequestMapping("/url")
  public String url(ModelMap map) {
    map.addAttribute("type", "link");
    map.addAttribute("pageId", "springcloud/2017/09/11/");
    map.addAttribute("img", "http://www.baidu.com/assets/images/neo.jpg");
    return "url";
  }
  ```


## 5. 三目运算

三目运算是我们常用的功能之一，普遍应用在各个项目中。
  
- 例如：

  ```html
  <input th:value="${name}"/>
  <input th:value="${age gt 30 ? '中年':'青年'}"/>
  ```

- 说明：

  表示如果 age 大于 30 则输入框中显示中年，否则显示青年。

- 三目运算符关键字：

  ``` 
  关键字 | 说明 
     gt | great than（大于）
     ge | great equal（大于等于）
     eq | equal（等于）
     lt | less than（小于）
     le | less equal（小于等于）
     ne | not equal（不等于）
  ```

- 结合三目运算也可以将上面的 if else 改成这样：

  ```html
  <a th:if="${flag eq 'yes'}" 
     th:href="@{http://www.baidu.com/}"> 百度 </a>
  ```

- 页面内容：

  ```html
  ...
  <body>
    <div >
      <h1>EQ</h1>
      <input th:value="${name}"/>
      <br/>
      <input th:value="${age gt 30 ? '中年年:'青年'}"/>
      <br/>
      <a th:if="${flag eq 'yes'}" th:href="@{http://www.baidu.com/}"> 百度 </a>
    </div>
  </body>
  ```

- 后端程序：

  ```java
  @RequestMapping("/eq")
  public String eq(ModelMap map) {
    map.addAttribute("name", "neo");
    map.addAttribute("age", 30);
    map.addAttribute("flag", "yes");
    return "eq";
  }
  ```

- 页面效果：

  ```txt
  EQ

  neo
  青年

  百度 
  ```

  单击 `百度` 链接会跳转到：`http://www.baidu.com/` 地址。

## 6. switch 选择

switch/case 多用于多条件判断的场景下。

- 以性别举例：

  ```html
  <!DOCTYPE html>
  <html lang="en" xmlns:th="http://www.thymeleaf.org">
    <head>
      <meta charset="UTF-8"></meta>
      <title>Example switch</title>
    </head>
    <body>
      <div >
        <div th:switch="${sex}">
          <p th:case="'woman'">她是一个姑娘...</p>
          <p th:case="'man'">这是一个爷们!</p>
          <!-- *: case的默认的选项 -->
          <p th:case="*">未知性别的一个家伙。</p>
        </div>
      </div>
    </body>
  </html>
  ```

- 后端程序：

  ```java
  @RequestMapping("/switch")
  public String switchcase(ModelMap map) {
    map.addAttribute("sex", "woman");
    return "switch";
  }
  ```

在浏览器中输入网址：`http://localhost:8080/switch` 。

- 页面展示效果如下：

  ```
  她是一个姑娘...
  ```

  可以在后台改 sex 的值来查看结果。
