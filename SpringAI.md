## langchain4j

引入依赖：

- 对于 `pom.xml` 中的 Maven：

```xml
<dependency>
    <groupId>dev.langchain4j</groupId>
    <artifactId>langchain4j-open-ai</artifactId>
    <version>1.1.0</version>
</dependency>
```

如果您希望使用高级 [AI 服务 ](https://docs.langchain4j.dev/tutorials/ai-services)API，则还需要添加以下依赖项：

```xml
<dependency>
    <groupId>dev.langchain4j</groupId>
    <artifactId>langchain4j</artifactId>
    <version>1.1.0</version>
</dependency>
```

## 配置类

```java

@Configuration
public class LLMConfig {
    @Bean(name = "Qwen")
    public ChatModel chatModelQwen() {
        return OpenAiChatModel.builder()
                .apiKey(System.getenv("DASHSCOPE_API_KEY"))
                .baseUrl("https://dashscope.aliyuncs.com/compatible-mode/v1")
                .modelName("qwen-plus")
                .build();
    }

    @Bean(name = "DeepSeek")
    public ChatModel chatModelDeepseek() {
        return OpenAiChatModel.builder()
                .apiKey(System.getenv("deepseek"))
                .baseUrl("https://api.deepseek.com/v1")
                .modelName("deepseek-chat")
                .build();
    }


}
```

## 大模型的调用

```java
    @Resource(name = "Qwen")
    private ChatModel chatModelQwen;
    @Resource(name = "DeepSeek")
    private ChatModel chatModelDeepSeek;

    // http://localhost:9002/multimodel/qwen
    @GetMapping(value = "/multimodel/qwen")
    public String qwenCall(@RequestParam(value = "prompt", defaultValue = "你是谁") String prompt)
    {
        String result = chatModelQwen.chat(prompt);

        System.out.println("通过langchain4j调用模型返回结果：\n"+result);

        return result;
    }

    // http://localhost:9002/multimodel/deepseek
    @GetMapping(value = "/multimodel/deepseek")
    public String deepseekCall(@RequestParam(value = "prompt", defaultValue = "你是谁") String prompt)
    {
        String result = chatModelDeepSeek.chat(prompt);

        System.out.println("通过langchain4j调用模型返回结果：\n"+result);

        return result;
    }
```

## 整和的两种方式

![image-20250710160432208](SpringAI.assets/image-20250710160432208.png)

上面一种是带用原生的langchain4j框架的方法

下面一种是调用langchain4j的集成方法，有RAG,Tool等

## 集成SpringBoot

### 底层集成

**maven添加**

```xml
<dependency>
    <groupId>dev.langchain4j</groupId>
    <artifactId>langchain4j-open-ai-spring-boot-starter</artifactId>
    <version>1.1.0-beta7</version>
</dependency>
```

**application.yml添加**

```
langchain4j.open-ai.chat-model.api-key=${OPENAI_API_KEY}
langchain4j.open-ai.chat-model.model-name=gpt-4o
langchain4j.open-ai.chat-model.log-requests=true
langchain4j.open-ai.chat-model.log-responses=true
```

在这种情况下，将自动创建一个 `OpenAiChatModel` 的实例（`ChatModel` 的实现），你可以在需要时自动连接它：

```java
@RestController
public class ChatController {

    ChatModel chatModel;

    public ChatController(ChatModel chatModel) {
        this.chatModel = chatModel;
    }

    @GetMapping("/chat")
    public String model(@RequestParam(value = "message", defaultValue = "Hello") String message) {
        return chatModel.chat(message);
    }
}
```

### 高级集成

```xml
<dependency>
    <groupId>dev.langchain4j</groupId>
    <artifactId>langchain4j-spring-boot-starter</artifactId>
    <version>1.1.0-beta7</version>
</dependency>
```

定义ai接口并使用@AiServer来解释

```java
@AiService
interface Assistant {

    @SystemMessage("You are a polite assistant")
    String chat(String userMessage);
}
```

将其视为标准的 Spring Boot `@Service`，但具有 AI 功能。

当应用程序启动时，LangChain4j starter 将扫描 Classpath 并查找所有带有 `@AiService` 注解的接口。对于找到的每个 AI Service，它将使用应用程序上下文中可用的所有 LangChain4j 组件创建此接口的实现，并将其注册为 bean，因此您可以在需要时自动连接它：

```java
@RestController
class AssistantController {

    @Autowired
    Assistant assistant;

    @GetMapping("/chat")
    public String chat(String message) {
        return assistant.chat(message);
    }
}
```

![image-20250710161544645](SpringAI.assets/image-20250710161544645.png)

两种整合方式对比

关于**==@AiService==**他不用编写实现类

在整合了springboot框架里面使用

## 模型生成器

```java
OpenAiChatModel model = OpenAiChatModel.builder()
        .apiKey(System.getenv("OPENAI_API_KEY"))
        .modelName("gpt-4o-mini")
        .temperature(0.3)
        .timeout(ofSeconds(60))
        .logRequests(true)
        .logResponses(true)
        .build();
```

可以在 `application.properties` 文件中设置 Quarkus 应用程序中的 LangChain4j 参数，如下所示：

```text
quarkus.langchain4j.openai.api-key=${OPENAI_API_KEY}
quarkus.langchain4j.openai.chat-model.temperature=0.5
quarkus.langchain4j.openai.timeout=60s
```









==@Autowired==和==@Resource==

@Autowired是根据类型来寻找注解的，寻找不到在根据名称来找，spring框架的

@Resource是先根据名称来寻找的，然后在根据类型找的，java自带

