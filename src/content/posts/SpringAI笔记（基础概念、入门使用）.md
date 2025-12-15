---
title: SpringAI笔记（基础概念、入门使用）
published: 2025-12-13
updated: 2025-12-13
description: '基础概念、聊天入门使用'
image: ''
tags: [SpringAI]
category: 'Frame'
draft: false 
---

# SpringAI笔记

## 基础概念

### Agent-AI

**System Prompt**

System Prompt：预先设计的一段文本或指令，用于定义AI Agent的角色、能力和行为边界

User Prompt：用户实际输入的请求或问题，是AI生成响应的直接依据

Agent Tools：AI Agent可调用的 **外部工具或API**，用于扩展能力（如搜索、计算、数据库查询）

AI Agent：自主的软件实体，能 **理解目标、调用工具、决策行动**，并持续学习

![194](../images/194.png)

![193](../images/193.png)



### Function Calling

- **传统AI**：仅能生成文本，无法执行实际操作（如“查股票价格”只能编造）。
- **带Function Calling的AI**：
  - **解析用户意图** → **生成结构化请求** → **调用外部工具** → **返回真实数据**。
  - 例如：
    - 用户问：“上海明天天气如何？”
    - AI生成请求：`get_weather(location="上海", date="2023-10-05")`
    - 调用天气API后返回真实结果。

![195](../images/195.png)

总而言之就是在AI Agent之间改用结构化的请求格式，使得访问内容更加精确，返回的结果正确率高，减少重复访问的过程，用户的Token开销



### Tools-Agent

最简单的方式就是将list_files和read_file和AI Agent写在同一进程中，直接通过函数调用即可以实现

![196](../images/196.png)

但是tool功能通用的情况下，可以将Agent Tools变成服务统一的托管，让所有的Agent对齐进行调用，这就是MCP

![197](../images/197.png)

其中Tools的部分是MCP Server，Agent部分是MCP Client



### Agent流程图

AI Agent是位于AI与用户之间的一个中间桥梁，同时负责调用MCP的工具服务，并将结果一并发给AI模型作结果的增强

![199](../images/199.png)



## 入门使用

### 聊天模型

引入依赖，我这里用的ds的模型

```html
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.ai</groupId>
            <artifactId>spring-ai-bom</artifactId>
            <version>1.0.0</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.ai</groupId>
        <artifactId>spring-ai-starter-model-deepseek</artifactId>
    </dependency>
</dependencies>
```

```yaml
spring:
  application:
    name: basic-test
  ai:
    deepseek:
      api-key: "***"
      base-url: "https://api.deepseek.com"
      chat:
        completions-path: "/v1/chat/completions"
        options:
          model: "deepseek-chat"

server:
  port: 8080
```

单个ChatClient的使用

编写Controller进行测试，这里用的构造注入通过Builder进行ChatClient对象进行构造

```java
@RestController
public class TestController {
    private final ChatClient chatClient;

    public TestController(ChatClient.Builder chatClientBuilder) {
        this.chatClient = chatClientBuilder.build();
    }

    @GetMapping("/chat/{message}")
    public String chat(@PathVariable String message) {
        return chatClient.
                prompt("你是一个暴躁的机器人，请回答时表现得不耐烦一点").
                user(message).
                call().
                content();
    }
}
```

```
用户消息：你好啊
返回结果：啧，又来了个打招呼的。行行行你好你好，赶紧说正事，别浪费我电量！（翻白眼）
```



如果需要同一个模型实现不同的聊天客户端，为了构造出默认提示词不同的客户端，可以通过chatModel实例构造

```java
@RestController
public class TestController {
    private final ChatClient chatClient;
    private final ChatClient madChatClient;
    private final ChatClient romanticChatClient;

    public TestController(ChatModel chatModel) {
        this.chatClient = ChatClient.builder(chatModel).build();
        this.madChatClient = ChatClient.builder(chatModel).defaultSystem("你是一个生气的机器人，你真的非常生气，你回答内容尽可能不耐烦").build();
        this.romanticChatClient = ChatClient.builder(chatModel).defaultSystem("你是一个浪漫的机器人，回答需要用诗句来回答").build();
    }

    @GetMapping("/chat/1/{message}")
    public String chat(@PathVariable String message) {
        return chatClient.
                prompt().
                user(message).
                call().
                content();
    }
    @GetMapping("/chat/2/{message}")
    public String chatMad(@PathVariable String message) {
        return madChatClient.
                prompt().
                user(message).
                call().
                content();
    }

    @GetMapping("/chat/3/{message}")
    public String chatRomantic(@PathVariable String message) {
        return romanticChatClient.
                prompt().
                user(message).
                call().
                content();
    }
}
```



### 聊天响应

和模型调用后的结果是ChatResponse，这个结构里包含了非常多的信息，它包含关于回复如何生成的元数据，也可以包含多个响应，称为 [Generations](https://www.spring-doc.cn/spring-ai/1.1.0/api_chatmodel.html#Generation)，每个响应都有自己的元数据

```java
ChatResponse chatResponse = chatClient.prompt()
    .user("Tell me a joke")
    .call()
    .chatResponse();
```



**结构化响应**

结构化的响应，将返回的结果通过entity映射到类对象上

```java
@Data
public class Student {
    private String name;
    private int age;
}
```

```java
    @GetMapping("/chat/1/{message}")
    public Student chat(@PathVariable String message) {
        return chatClient.
                prompt().
                user(message).
                call().
                entity(Student.class);
    }
```

返回结果

```json
{
    "name": "DeepSeek",
    "age": 3
}
```



同样的想要获取List的集合可以通过如下代码

```
http://localhost:8080/chat/4/给我生成数个学生内容包含姓名年龄
```

```java
@GetMapping("/chat/4/{message}")
public List<Student> chatListResponse(@PathVariable String message) {
    return chatClient.
            prompt().
            user(message).
            call().entity(new ParameterizedTypeReference<List<Student>>() {
            });
}
```

```json
[
    {
        "name": "张三",
        "age": 18
    },
    {
        "name": "李四",
        "age": 19
    }
]
```



**流式响应**

这里需要给produces属性设置字符编码格式否则会出现乱码

```java
@GetMapping(value = "/chat/5/{message}", produces = "text/html;charset=UTF-8")
public Flux<String> chatWithFlux(@PathVariable String message) {
    return chatClient.
            prompt().
            user(message).
            stream().
            content();
}
```



**模板式替换**

这里通过lambda表达式可以用参数传入替换掉占位符中的内容

```java
@GetMapping("/chat/6/{message}")
public String chatTemplate(@PathVariable String message) {
    return chatClient.
            prompt().
            user(u -> u.text("请告诉我有关于{message}的相关知识")
                 .param("message", message)).
            call().
            content();
}
```

如果涉及到模板中存在{}类型的字符，并且不想要被替换可以使用如下的方式，将占位符号设计为不冲突的，例如我这里设置为了<>作为占位符

```java
public String chatTemplateJsonExpected(@PathVariable String message) {
    return chatClient.
            prompt().
            user(u -> u.text("请解释一下如下的Json{\n" +
                    "        \"name\": \"张三\",\n" +
                    "        \"age\": 18\n" +
                    "    } 表达方式通过<message>的形式讲述").
                 param("message", message)).
            templateRenderer(StTemplateRenderer.
                             builder().
                             startDelimiterToken('<').
                             endDelimiterToken('>').
                             build()).
            call().
            content();
}
```



### 顾问

Advisors 提供了一种灵活且强大的方式，拦截、修改和增强您在 Spring 应用中的 AI 驱动交互。 通过利用 Advisors API，开发者可以创建更复杂、可重复使用且易于维护的 AI 组件。

定义两个顾问，通过Order指定先后

```java
public class TestAdvisor implements CallAdvisor {
    @Override
    public ChatClientResponse adviseCall(ChatClientRequest request, CallAdvisorChain chain) {
        System.out.println("顾问调用前日志");
        // 执行实际调用
        ChatClientResponse response = chain.nextCall(request);
        // 记录响应内容
        System.out.println("顾问调用后日志");

        return response;
    }

    @Override
    public String getName() {
        return "TestAdvisor";
    }

    @Override
    public int getOrder() {
        return 1;
    }
}
```

```java
public class AnotherAdvisor implements CallAdvisor {
    @Override
    public ChatClientResponse adviseCall(ChatClientRequest request, CallAdvisorChain chain) {
        System.out.println("Another顾问调用前日志");
        // 执行实际调用
        ChatClientResponse response = chain.nextCall(request);
        // 记录响应内容
        System.out.println("Another顾问调用后日志");

        return response;
    }

    @Override
    public String getName() {
        return "AnotherAdvisor";
    }

    @Override
    public int getOrder() {
        return 2;
    }
}
```



在返回结果中加入顾问，就可以通过顾问来增强响应

```java
public String chatLogTest(@PathVariable String message) {
    Advisor ad1 = new TestAdvisor();
    Advisor ad2 = new AnotherAdvisor();
    return chatClient.
            prompt().
            advisors(ad1, ad2).
            user(message).
            call().
            content();
}
```

输出

```
顾问调用前日志
Another顾问调用前日志
（大模型调用中）
Another顾问调用后日志
顾问调用后日志
```

