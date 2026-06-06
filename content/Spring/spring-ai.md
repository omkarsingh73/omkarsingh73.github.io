- [[#Maven|Maven]]
- [[#Configuration|Configuration]]
- [[#Service|Service]]
- [[#Controller|Controller]]
- [[#Controller#DTO|DTO]]
- [[#Controller#Call|Call]]
- [[#Controller#Service|Service]]
- [[#Controller#Register Tool|Register Tool]]
- [[#Controller#Ask AI|Ask AI]]
- [[#Controller#Use System Prompts|Use System Prompts]]
- [[#Controller#Use Structured Output|Use Structured Output]]
- [[#Controller#Add Retry|Add Retry]]
- [[#Controller#Log Tokens|Log Tokens]]
- [[#Controller#Keep Temperature Low|Keep Temperature Low]]
- [[#Learning Roadmap|Learning Roadmap]]

---

# 1. What is Spring AI?

Spring AI is a framework that brings AI capabilities into Spring applications using familiar Spring patterns.

Instead of directly calling the OpenAI REST API, Spring AI provides:

- Auto-configuration
    
- Dependency Injection
    
- Unified API across LLM providers
    
- Prompt templates
    
- RAG support
    
- Function/tool calling
    
- Chat memory
    
- Structured output
    

Without Spring AI:

```java
RestTemplate/WebClient
      ↓
OpenAI REST API
      ↓
JSON parsing
```

With Spring AI:

```java
ChatClient
      ↓
Spring AI
      ↓
OpenAI
```

This reduces boilerplate significantly.

---

# 2. Architecture

```text
User
  ↓
Controller
  ↓
ChatClient
  ↓
OpenAI Chat Model
  ↓
GPT Response
```

Core components:

|Component|Purpose|
|---|---|
|ChatClient|Main API for prompting|
|OpenAiChatModel|OpenAI model wrapper|
|Prompt|Prompt object|
|ChatResponse|Full response|
|Advisor|Memory/RAG support|
|Tool Calling|Call Java methods from AI|

---

# 3. Create a Spring Boot Project

Dependencies:

### Maven

```xml
<dependencies>

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

    <dependency>
        <groupId>org.springframework.ai</groupId>
        <artifactId>spring-ai-starter-model-openai</artifactId>
    </dependency>

</dependencies>
```

Also add Spring AI BOM:

```xml
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

---

# 4. Configure OpenAI

`application.yml`

```yaml
spring:
  ai:
    openai:
      api-key: ${OPENAI_API_KEY}

      chat:
        options:
          model: gpt-4.1-mini
          temperature: 0.7
```

Environment variable:

```bash
export OPENAI_API_KEY=your-key
```

---

# 5. First Chat Example

## Configuration

```java
@Configuration
public class AIConfig {

    @Bean
    ChatClient chatClient(ChatClient.Builder builder) {
        return builder.build();
    }
}
```

---

## Service

```java
@Service
public class AIService {

    private final ChatClient chatClient;

    public AIService(ChatClient chatClient) {
        this.chatClient = chatClient;
    }

    public String ask(String question) {

        return chatClient.prompt()
                .user(question)
                .call()
                .content();
    }
}
```

---

## Controller

```java
@RestController
@RequestMapping("/ai")
public class AIController {

    private final AIService aiService;

    public AIController(AIService aiService) {
        this.aiService = aiService;
    }

    @GetMapping("/ask")
    public String ask(@RequestParam String q) {
        return aiService.ask(q);
    }
}
```

Request:

```http
GET /ai/ask?q=What is Spring AI?
```

Response:

```text
Spring AI is a framework...
```

---

# 6. Understanding ChatClient

Most commonly used API:

```java
chatClient.prompt()
          .user("Explain Java Streams")
          .call()
          .content();
```

Flow:

```text
prompt()
   ↓
user()
   ↓
call()
   ↓
content()
```

---

# 7. System Prompt

System prompts define behavior.

```java
String response = chatClient.prompt()
        .system("""
            You are a senior Java architect.
            Explain concepts with examples.
        """)
        .user("Explain Dependency Injection")
        .call()
        .content();
```

The model will answer like an architect rather than a generic assistant.

---

# 8. Prompt Templates

Instead of building strings manually:

```java
String response = chatClient.prompt()
        .user(u -> u.text("""
            Explain {topic} in simple terms.
            """)
            .param("topic", "Spring Security"))
        .call()
        .content();
```

Generated prompt:

```text
Explain Spring Security in simple terms.
```

---

# 9. Structured Output

One of Spring AI's strongest features.

Suppose AI returns a Java object.

### DTO

```java
public record Book(
        String title,
        String author,
        String summary
) {}
```

### Call

```java
Book book = chatClient.prompt()
        .user("Recommend a Spring Boot book")
        .call()
        .entity(Book.class);
```

Spring AI automatically maps the AI response into the object.

Result:

```java
Book(
  title="Spring Start Here",
  author="Laurentiu Spilca",
  summary="..."
)
```

Very useful for APIs.

---

# 10. Chat Memory

Without memory:

```text
User: My name is John.
AI: Nice to meet you.

User: What is my name?
AI: I don't know.
```

With memory:

```java
chatClient.prompt()
    .advisors(new MessageChatMemoryAdvisor(memory))
    .user("My name is John")
    .call();
```

Later:

```java
chatClient.prompt()
    .advisors(new MessageChatMemoryAdvisor(memory))
    .user("What is my name?")
    .call();
```

AI remembers previous messages.

---

# 11. Streaming Responses

Useful for ChatGPT-like UI.

```java
@GetMapping(value="/stream",
        produces = MediaType.TEXT_EVENT_STREAM_VALUE)
public Flux<String> stream(String question) {

    return chatClient.prompt()
            .user(question)
            .stream()
            .content();
}
```

Frontend receives tokens continuously.

```text
Spring...
AI...
is...
a...
framework...
```

---

# 12. Tool Calling (Function Calling)

This is where things become powerful.

AI can invoke Java methods.

### Service

```java
@Service
public class WeatherService {

    public String getWeather(String city) {
        return city + " is 28°C";
    }
}
```

---

### Register Tool

```java
@Bean
ToolCallbackProvider tools(
        WeatherService weatherService) {

    return MethodToolCallbackProvider.builder()
            .toolObjects(weatherService)
            .build();
}
```

---

### Ask AI

```java
String response = chatClient.prompt()
        .user("What is the weather in Pune?")
        .call()
        .content();
```

Flow:

```text
User
 ↓
LLM
 ↓
Tool call detected
 ↓
WeatherService
 ↓
Result
 ↓
Final response
```

The model automatically decides when to call the tool.

---

# 13. RAG (Retrieval Augmented Generation)

Problem:

```text
OpenAI doesn't know your company's
private documents.
```

Solution:

```text
PDF
 ↓
Vector Store
 ↓
Similarity Search
 ↓
Relevant Chunks
 ↓
Prompt
 ↓
OpenAI
```

Spring AI supports vector stores such as:

- PostgreSQL (pgvector)
    
- Redis
    
- Elasticsearch
    
- MongoDB
    

Example:

```java
List<Document> docs =
        vectorStore.similaritySearch(question);
```

Those documents are added to the prompt automatically.

---

# 14. Embeddings

Embeddings convert text into vectors.

```text
"Spring Boot"
    ↓
[0.123, 0.881, ...]
```

Used for:

- Semantic search
    
- RAG
    
- Similarity matching
    
- Recommendations
    

Example:

```java
EmbeddingModel embeddingModel;
```

Generate embedding:

```java
EmbeddingResponse response =
        embeddingModel.embedForResponse(
            List.of("Spring AI")
        );
```

---

# 15. Production Best Practices

### Use System Prompts

```java
.system("""
You are an expert software engineer.
Return concise answers.
""")
```

---

### Use Structured Output

Prefer:

```java
.entity(Book.class)
```

Instead of:

```java
String parsing
```

---

### Add Retry

```java
spring:
  retry:
    enabled: true
```

---

### Log Tokens

Track:

- Input tokens
    
- Output tokens
    
- Cost
    

For monitoring.

---

### Keep Temperature Low

For business apps:

```yaml
temperature: 0.2
```

For creativity:

```yaml
temperature: 0.8
```

---

# Complete Mini Project

```text
spring-ai-demo
│
├── Controller
│    └── AIController
│
├── Service
│    └── AIService
│
├── Config
│    └── AIConfig
│
└── application.yml
```

Service:

```java
@Service
public class AIService {

    private final ChatClient chatClient;

    public AIService(ChatClient chatClient) {
        this.chatClient = chatClient;
    }

    public String explain(String topic) {

        return chatClient.prompt()
                .system("You are a senior Java trainer.")
                .user("Explain " + topic)
                .call()
                .content();
    }
}
```

Controller:

```java
@RestController
@RequestMapping("/api/ai")
public class AIController {

    private final AIService service;

    public AIController(AIService service) {
        this.service = service;
    }

    @GetMapping("/explain")
    public String explain(
            @RequestParam String topic) {

        return service.explain(topic);
    }
}
```

Call:

```http
GET /api/ai/explain?topic=Spring Security
```

---

## Learning Roadmap

1. Learn basic `ChatClient`
    
2. Learn prompt templates
    
3. Learn system prompts
    
4. Learn structured output (`entity(Class)`)
    
5. Learn streaming (`Flux<String>`)
    
6. Learn tool/function calling
    
7. Learn embeddings
    
8. Learn vector databases
    
9. Learn RAG
    
10. Build an AI chatbot for your Spring Boot application
    

A great practice project is: **Build a Spring Boot customer-support chatbot using Spring AI + OpenAI + PostgreSQL(pgvector) + RAG**, because it covers nearly every important Spring AI concept used in production.