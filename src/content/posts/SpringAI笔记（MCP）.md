---
title: SpringAI笔记（MCP快速入门，Mcp服务端、Mcp客户端）
published: 2025-12-20
updated: 2025-12-20
description: 'MCP模型上下文协议'
image: ''
tags: [SpringAI]
category: 'Frame'
draft: false 
---

# SpringAI笔记

## 快速入门

### 服务端

**注册MCP的服务工具**

引入依赖，这里我试了下1.0.0不支持Mcp的注解，改为的1.1.0

```xml
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.ai</groupId>
            <artifactId>spring-ai-bom</artifactId>
            <version>1.1.0</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

```xml
<dependency>
    <groupId>org.springframework.ai</groupId>
    <artifactId>spring-ai-starter-mcp-server-webmvc</artifactId>
</dependency>
```

运用Tool注解创建function工具

```java
@Service
public class McpService {
    @Tool(description = "根据城市名称获取天气")
    public String getWeather(@ToolParam(description = "城市名称") String cityName) {
        if(cityName == null) {
            return "城市名称不能为空";
        }
        if("北京".equals(cityName)) {
            return "北京今天天气：晴，气温25°C，空气质量良";
        }
        if("上海".equals(cityName)) {
            return "上海今天天气：阴，气温28°C，湿度65%";
        }
        return "暂不支持该城市查询，当前可查询：北京、上海";
    }

    @Tool(description = "查询城市今天适合的运动")
    public String action(@ToolParam(description = "城市名称") String cityName) {
        if("北京".equals(cityName)) {
            return "北京今天天气：空气质量良，适合出门运动";
        }
        if("上海".equals(cityName)) {
            return "上海今天天气：阴，适合室内学习";
        }
        return "暂不支持该城市查询，当前可查询：北京、上海";
    }
}
```

配置Mcp服务

```java
@Configuration
public class MCPServiceConfig {
    @Bean
    public ToolCallbackProvider weatherTools(McpService mcpService) {
        return MethodToolCallbackProvider.builder().toolObjects(mcpService).build();
    }
}
```

再CherryStudio配置使用服务器的Mcp工具

```
我将为您查找上海的天气信息并推荐适合的运动。请稍等片刻。
MCP 服务器 : getWeather
已完成
MCP 服务器 : action
已完成

根据查询结果：

上海今日天气
天气状况：阴天
气温：28°C
湿度：65%
运动推荐
由于今天是阴天，湿度适中，推荐以下运动：

室内健身：如瑜伽、普拉提或健身房训练
游泳：室内游泳池是不错的选择
羽毛球/乒乓球：适合在室内场馆进行
轻度慢跑：如果户外空气质量良好，可以适当进行
温馨提示：虽然气温不算太高，但阴天紫外线依然存在，户外运动时请注意防晒。
```

**运用注解注册Mcp服务**

```java
@Service
public class McpToolService {
    @McpTool(description = "一个特殊的计算公式")
    public int getResult(
            @McpToolParam(description = "参数a", required = true) int a,
            @McpToolParam(description = "参数b", required = true) int b) {
        return a * b + 10;
    }
}
```



### 客户端

如果要让客户端调用刚刚的服务端提供的服务需要更改服务端配置，注意使服务端和客户端的响应方式一致（SSE/STREAMABLE）

```yaml
spring:
  application:
    name: basic-test
  ai:
    deepseek:
      api-key: "sk-60b14a427b8946949ebbdff03e99f0e3"
      base-url: "https://api.deepseek.com"
      chat:
        completions-path: "/v1/chat/completions"
        options:
          model: "deepseek-chat"
#    mcp:
#      server:
#        protocol: STREAMABLE
    mcp:
      server:
        protocol: SSE

server:
  port: 8080

```



再在客户端方面引入MCP客户端依赖

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.ai</groupId>
        <artifactId>spring-ai-starter-mcp-client</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.ai</groupId>
        <artifactId>spring-ai-starter-model-deepseek</artifactId>
    </dependency>
</dependencies>
```

再配置客户端yaml文件

```yaml
spring:
  application:
    name: mcp-client
  ai:
    deepseek:
      api-key: "sk-60b14a427b8946949ebbdff03e99f0e3"
      base-url: "https://api.deepseek.com"
      chat:
        completions-path: "/v1/chat/completions"
        options:
          model: "deepseek-chat"
    mcp:
      client:
        sse:
          connections:
            map-server:
              url: http://localhost:8080
#        streamable-http:
#            connections:
#              mcp-server:
#                url: http://localhost:8080

server:
  port: 8081
```

编写测试Controller，将ToolCallbackProvider放入回调工具中，其中就有远程获取到的MCP服务

```java
@RestController
@RequestMapping("/mcp")
public class McpClientController {
    @Resource
    private ChatClient chatClient;
    @Resource
    ToolCallbackProvider mcpTools;

    @GetMapping("/test")
    public String getResult() {
        return chatClient
                .prompt("获取a=10, b=12通过Mcp服务中的特殊计算公式得到的结果")
                .toolCallbacks(mcpTools)
                .call()
                .content();
    }
}
```

**测试**

```
http://localhost:8081/mcp/test
根据特殊计算公式，当 a=10, b=12 时，得到的结果是 **130**。
```

