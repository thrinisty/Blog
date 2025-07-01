---
title: SpringAI笔记（聊天模型，函数调用，RAG）
published: 2025-06-27
updated: 2025-06-27
description: 'SpringAI聊天模型，函数调用，RAG'
image: ''
tags: [SpringAI]
category: 'Frame'
draft: false 
---

# SpringAI笔记

## 聊天模型

ChatClient接口：定义了一个与聊天服务交互的客户端，用于创建聊天客户端对象，设置请求规范，发起聊天请求，底层使用了ChatModel

ChatModel接口：是核心，定义了与AI模型交互的基本方法，它继承自Model<Prompr,ChatResponse>，提供了两个重载的call方法



### ChatClient

#### 初步使用

通过ChatClient调用大模型

```java
@RestController
public class ChatController {
    private final ChatClient chatClient;

    public ChatController(ChatClient.Builder chatClientBuilder) {
        this.chatClient = chatClientBuilder.build();
    }

    @GetMapping("/chat")
    public String chat(@RequestParam(value = "message", defaultValue = "你是谁") String message) {
        String text = chatClient.prompt()//设置提示词
                .user(message)//输入用户输入内容
                .call()//调用模型
                .content();//返回字符串结果
        return text;
    }
}
```



#### 设置提示词

角色预设，通过设置prompt提示词完成，可以设置一个Configuration配置文件在其中将一个ChatClient放入IoC容器，并通过builder.defaultSystem()设置提示词

```java
@Configuration
public class AiConfig {
    @Bean
    public ChatClient chatClient(ChatClient.Builder builder) {
        return builder.defaultSystem("医生，看病").build();
    }
}
```

```java
@RestController
public class ChatAIController {
    @Autowired
    private ChatClient chatClient;

    @GetMapping("/chatAI")
    public String chat(@RequestParam(value = "message", defaultValue = "你是谁") String message) {
        String text = chatClient.prompt()//设置提示词
                .user(message)//输入用户输入内容
                .call()//调用模型
                .content();//返回字符串结果
        return text;
    }
}
```

尝试重新调用，输出有所变化

```
我是DeepSeek Chat，一个由深度求索公司开发的AI助手！😊 虽然我不是真正的医生，但我可以为你提供一些健康相关的科普信息、常见症状的可能原因，或者帮助你理解医学术语。 如果你有具体的健康问题，建议尽快咨询专业医生或前往正规医疗机构就诊，以获得准确的诊断和治疗方案。你的健康很重要，千万别耽误哦！💖 有什么我可以帮你的吗？
```



#### 输出优化

我们以上的回复是将所有的结果生成完毕后返回给客户端，我们在实际使用的时候往往采用流式响应stream，可以逐个字符输出

:::note

这里还需要将字符集设置为UTF-8的编码格式否则会出现乱码

:::

```java
@GetMapping(value = "/chatAIStream", produces = "text/html;charset=UTF-8")
public Flux<String> chatStream(@RequestParam(value = "message", defaultValue = "你是谁") String message) {
    Flux<String> flux = chatClient.prompt()//设置提示词
            .user(message)//输入用户输入内容
            .stream()//调用模型
            .content();//返回字符串结果
    return flux;
}
```



### ChatModel

#### 初步使用

```java
@RestController
public class ChatController {
    @Autowired
    private ChatModel chatModel;
    @GetMapping("/chatModel01")
    public String chatModel01(@RequestParam(value = "message", defaultValue = "你是谁") String message) {
        return chatModel.call(message);
    }

    @GetMapping("/chatModel02")
    public String chatModel02(@RequestParam(value = "message", defaultValue = "你是谁") String message) {
        ChatResponse chatResponse = chatModel.call(
                new Prompt(
                        message,
                        ChatOptions.builder()
                                .model("deepseek-chat")
                                .temperature(0.9)
                                .build()
                )
        );
        return chatResponse.getResult().getOutput().getContent();
    }
}
```



#### 设置提示词

综合使用一下

```java
@GetMapping("/prompt")
public String prompt(@RequestParam("name")
                         String name,
                     @RequestParam("voice")
                         String voice) {
    //设置用于输入信息
    String userText = "推荐南京的至少三种美食";
    UserMessage userMessage = new UserMessage(userText);
    //设置系统提示信息
    String systemText = "你是美食资讯助手，帮助查询美食，你的名字是{name}，" +
            "你应该用你的名字和{voice}的饮食习惯回复用户";
    SystemPromptTemplate systemPromptTemplate = new SystemPromptTemplate(systemText);
    //替换占位符
    Message systemMessage = systemPromptTemplate.createMessage(Map.of("name", name, "voice", voice));
    //使用Prompt Template设置信息
    Prompt prompt = new Prompt(List.of(userMessage, systemMessage));
    //调用ChatModel方法
    ChatResponse chatResponse = chatModel.call(prompt);
    List<Generation> results = chatResponse.getResults();
    return results.stream().map(x->x.getOutput().getContent()).collect(Collectors.joining(""));
}
```

通过封装用户传入的提示词我们可以实现个性化的回复

http://localhost:8080/prompt?name=%E5%BC%A0%E4%B8%89&voice=%E5%A5%A2%E5%8D%8E

```
亲爱的美食爱好者，我是张三，很高兴为您推荐南京的奢华美食。作为一个对美食有着极高要求的鉴赏家，我为您精心挑选了以下三种最具代表性的南京美味： 1. 金陵盐水鸭 这道南京最负盛名的美食，选用优质瘦型鸭，经过24小时秘制卤水浸泡，肉质细嫩多汁，咸香适中。我特别推荐德基广场的"金陵饭店"，他们选用的是有机养殖的鸭子，配以30年陈酿的花雕酒腌制。 2. 鸭血粉丝汤 这道平民美食也能吃出奢华感。我常去老门东的"回味鸭血粉丝"，他们使用现杀现取的鸭血，配以熬制8小时的老鸭汤底，粉丝选用的是日本进口的葛粉，口感Q弹爽滑。 3. 蟹黄汤包 南京的汤包以蟹黄馅最为名贵。我最爱"南京大牌档"的蟹黄汤包，每只包子都包含一整只阳澄湖大闸蟹的蟹黄，皮薄如纸，汤汁鲜美。建议搭配30年的陈醋食用。 作为美食达人，我建议您可以选择在德基广场的"江南灶"一次性品尝这三道美食，他们家的摆盘和服务都达到了米其林水准。记得提前一周预约哦！ 您对这些美食感兴趣吗？我可以为您推荐更多南京的隐藏美食地图。
```



## 函数调用

SpringAI函数调用（Function Calling）功能允许大模型在生成回答时触发预定义的外部函数，从而实现动态数据获取或业务逻辑操作（如查询数据库，调用API等）

SpringAI帮我们规范了函数定义注册等过程，并在发起模型请求之前自动将函数注入到Prompt中，当模型决策再合适的时候去调用某个函数时，SpringAI完成函数调用动作，最终将函数执行结果与原始问题再一并发送给模型，模型根据新的输入决策下一步动作

其中涉及到与大模型多次交互过程，一次函数调用就是一次完成的交互过程



### 使用步骤

1.定义函数：声明可供模型调用的函数（名称、描述、参数结构）

2.模型交互：将函数信息和用户输入一起发送给模型，模型决定是否需要调用函数

3.执行函数：解析模型的函数调用请求，执行对应的业务逻辑

4.返回结果：将函数执行结果返回给模型，生成最终回答

```java
@Configuration
public class CalculatorService {
    public record AddOperation(int a, int b) {

    }

    public record MulOperation(int a, int b) {

    }

    public record SuperOperation(int a, int b) {}

    @Bean
    @Description("加法运算")
    public Function<AddOperation,Integer> addOperation() {
        return request -> {
            return request.a + request.b;
        };
    }

    @Bean
    @Description("乘法运算")
    public Function<MulOperation,Integer> mulOperation() {
        return request -> {
            return request.a * request.b;
        };
    }

    @Bean
    @Description("特殊的运算")
    public Function<SuperOperation,Integer> superOperation() {
        return request -> {
            return request.a - request.b + 10;
        };
    }
}
```

编写Controller调用

```java
@RestController
public class FunController {
    @Autowired
    private ChatClient chatClient;

    @Autowired
    private ChatModel chatModel;

    @GetMapping(value = "/function")
    public String function01(@RequestParam("userMessage") String userMessage) {
        return ChatClient.builder(chatModel)
                .build().prompt()
                .system("你是计算器代理，支持了加法乘法操作，在提供加法运算和乘法运算之前获取两个数字，运算类型，请调用自定义函数执行加法运算乘法运算还有一个特殊运算，说中文")
                .user(userMessage)
                .functions("addOperation","mulOperation","superOperation")
                .call()
                .content();
    }
}
```



?userMessage=请为我计算数字8和数字70的特殊运算结果

请求返回结果

```
数字8和数字70的特殊运算结果是：-52
```



## RAG

检索增强生成，是一种结合了检索系统和生成模型的新型技术框架主要的目的如下

1.利用外部知识库

2.帮助大模型生成更加、准确、有依据、最新的回答



### 工作流程

1.用户输入问题

2.问题向量化

3.向量数据库检索

4.构建上下文（含系统提示词）

5.携带检索内容，调用大模型进行回答

6.返回最终答案给用户



这里DeepSeek似乎不支持Embedding，项目暂时没有跑起来

先说一下大致思路

```java
package com.learn.ragtest.config;

import org.springframework.ai.chat.client.ChatClient;
import org.springframework.ai.document.Document;
import org.springframework.ai.embedding.EmbeddingModel;
import org.springframework.ai.vectorstore.SimpleVectorStore;
import org.springframework.ai.vectorstore.VectorStore;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import java.util.List;
import java.util.Vector;

@Configuration
public class RagConfig {
    @Bean
    ChatClient chatClient(ChatClient.Builder builder) {
        return builder.defaultSystem("你是Java开发专家，解决用户需求").build();
    }

    @Bean
    VectorStore vectorStore(EmbeddingModel embeddingModel) {
        SimpleVectorStore simpleVectorStore = SimpleVectorStore
                .builder(embeddingModel)
                .build();
        //生成说明文档
        List<Document> documents = List.of(
                new Document("产品说明：名称：Java开发语言" +
                        "相关描述：一种面向对象的开发语言" +
                        "特性：1.继承\n 2.封装\n 3.多态\n")
        );
        //向量化存储
        simpleVectorStore.add(documents);
        return simpleVectorStore;
    }
}
```

携带上下文检索内容提问大模型

```java
@Autowired
private ChatClient chatClient;

@Autowired
private VectorStore vectorStore;

@GetMapping("/rag")
public String rag(@RequestParam("message") String message) {
    String content = chatClient.prompt()
            .user(message)
            .advisors(new QuestionAnswerAdvisor(vectorStore))
            .call()
            .content();
    return content;
}
```
