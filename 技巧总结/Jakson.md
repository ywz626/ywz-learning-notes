# Jackson 快速入门：常用 API 大全

Jackson 是 Java 中最强大、最灵活的 JSON 处理库之一。下面是 Jackson 常用 API 的全面指南，涵盖从基础操作到高级功能。

## 一、核心类与初始化

### 1. ObjectMapper - 核心处理器
```java
import com.fasterxml.jackson.databind.ObjectMapper;

// 创建 ObjectMapper 实例（线程安全，建议重用）
ObjectMapper mapper = new ObjectMapper();

// 常用配置
mapper.configure(SerializationFeature.INDENT_OUTPUT, true); // 美化输出
mapper.disable(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES); // 忽略未知属性
mapper.setSerializationInclusion(JsonInclude.Include.NON_NULL); // 忽略 null 值
mapper.registerModule(new JavaTimeModule()); // 支持 Java 8 时间类型
```

## 二、基本序列化与反序列化

### 1. 简单对象转换
```java
// Java对象 → JSON字符串
String json = mapper.writeValueAsString(user);

// JSON字符串 → Java对象
User user = mapper.readValue(json, User.class);

// 文件操作
mapper.writeValue(new File("user.json"), user); // 写入文件
User user = mapper.readValue(new File("user.json"), User.class); // 从文件读取
```

### 2. 集合类型处理
```java
// List 序列化/反序列化
List<User> users = Arrays.asList(user1, user2);
String jsonList = mapper.writeValueAsString(users);
List<User> parsedList = mapper.readValue(jsonList, new TypeReference<List<User>>() {});

// Map 处理
Map<String, User> userMap = Map.of("user1", user1, "user2", user2);
String jsonMap = mapper.writeValueAsString(userMap);
Map<String, User> parsedMap = mapper.readValue(jsonMap, new TypeReference<Map<String, User>>() {});
```

## 三、树模型（Tree Model）API

当没有对应 Java 类时，使用树模型处理 JSON

### 1. 读取树模型
```java
String json = "{\"name\":\"John\",\"age\":30,\"cars\":[\"Ford\",\"BMW\"]}";

// 解析为树
JsonNode rootNode = mapper.readTree(json);

// 获取字段值
String name = rootNode.get("name").asText(); // "John"
int age = rootNode.get("age").asInt(); // 30

// 处理数组
JsonNode carsNode = rootNode.get("cars");
carsNode.forEach(car -> System.out.println(car.asText()));
```

### 2. 构建树模型
```java
ObjectNode rootNode = mapper.createObjectNode();
rootNode.put("name", "John");
rootNode.put("age", 30);

ArrayNode carsNode = rootNode.putArray("cars");
carsNode.add("Ford");
carsNode.add("BMW");

String json = mapper.writeValueAsString(rootNode);
```

## 四、流式 API（高性能处理）

适用于大文件或高性能场景

### 1. 生成 JSON
```java
StringWriter sw = new StringWriter();
JsonGenerator generator = mapper.createGenerator(sw);

generator.writeStartObject();
generator.writeStringField("name", "John");
generator.writeNumberField("age", 30);

generator.writeArrayFieldStart("cars");
generator.writeString("Ford");
generator.writeString("BMW");
generator.writeEndArray();

generator.writeEndObject();
generator.close();

String json = sw.toString();
```

### 2. 解析 JSON
```java
JsonFactory factory = mapper.getFactory();
JsonParser parser = factory.createParser(json);

while (parser.nextToken() != null) {
    String fieldName = parser.getCurrentName();
    
    if ("name".equals(fieldName)) {
        parser.nextToken();
        String name = parser.getText();
    } 
    else if ("age".equals(fieldName)) {
        parser.nextToken();
        int age = parser.getIntValue();
    }
    // 其他字段处理...
}
parser.close();
```

## 五、注解使用

Jackson 的强大之处在于注解驱动

### 1. 常用注解
```java
@JsonProperty("user_name")  // 自定义字段名
private String name;

@JsonIgnore                 // 忽略字段
private String password;

@JsonFormat(pattern = "yyyy-MM-dd HH:mm:ss") // 日期格式化
private LocalDateTime createTime;

@JsonInclude(Include.NON_EMPTY) // 空值不序列化
private String description;

@JsonSerialize(using = CustomSerializer.class) // 自定义序列化
private Money price;

@JsonDeserialize(using = CustomDeserializer.class) // 自定义反序列化
private Money price;

@JsonView(Views.Public.class) // 视图控制
private String publicField;
```

### 2. 多态类型处理
```java
@JsonTypeInfo(
    use = JsonTypeInfo.Id.NAME,
    include = JsonTypeInfo.As.PROPERTY,
    property = "type"
)
@JsonSubTypes({
    @JsonSubTypes.Type(value = Car.class, name = "car"),
    @JsonSubTypes.Type(value = Truck.class, name = "truck")
})
public abstract class Vehicle {}
```

## 六、高级功能

### 1. 自定义序列化/反序列化
```java
public class MoneySerializer extends StdSerializer<Money> {
    public MoneySerializer() {
        super(Money.class);
    }

    @Override
    public void serialize(Money value, JsonGenerator gen, SerializerProvider provider) {
        gen.writeString(value.getAmount() + " " + value.getCurrency());
    }
}

public class MoneyDeserializer extends StdDeserializer<Money> {
    public MoneyDeserializer() {
        super(Money.class);
    }

    @Override
    public Money deserialize(JsonParser p, DeserializationContext ctxt) {
        String[] parts = p.getText().split(" ");
        return new Money(Double.parseDouble(parts[0]), parts[1]);
    }
}
```

### 2. 过滤字段
```java
// 动态过滤
SimpleFilterProvider filters = new SimpleFilterProvider()
    .addFilter("userFilter", 
        SimpleBeanPropertyFilter.filterOutAllExcept("name", "email"));

mapper.setFilterProvider(filters);
String json = mapper.writeValueAsString(user);
```

### 3. JSON Schema
```java
JsonSchemaGenerator generator = new JsonSchemaGenerator(mapper);
JsonSchema schema = generator.generateSchema(User.class);
String schemaJson = mapper.writerWithDefaultPrettyPrinter()
    .writeValueAsString(schema);
```

## 七、实用工具方法

### 1. 类型转换
```java
// JSON 转 Map
Map<String, Object> map = mapper.readValue(json, new TypeReference<>() {});

// JSON 转 List
List<Map<String, Object>> list = mapper.readValue(jsonArray, new TypeReference<>() {});
```

### 2. JSON 操作
```java
// 合并两个 JSON
JsonNode node1 = mapper.readTree(json1);
JsonNode node2 = mapper.readTree(json2);
JsonNode merged = JsonNodeFactory.instance.objectNode();
((ObjectNode) merged).setAll((ObjectNode) node1);
((ObjectNode) merged).setAll((ObjectNode) node2);

// 比较 JSON
boolean isEqual = node1.equals(node2);
```

### 3. 异常处理
```java
try {
    User user = mapper.readValue(json, User.class);
} catch (JsonParseException e) {
    // JSON 格式错误
} catch (JsonMappingException e) {
    // 映射错误
} catch (IOException e) {
    // IO 错误
}
```

## 八、性能优化技巧

1. **重用 ObjectMapper**：创建成本高，应全局共享
2. **使用流式 API**：处理大文件时内存占用更少
3. **预编译类型**：避免重复类型解析
   ```java
   JavaType userListType = mapper.getTypeFactory()
       .constructCollectionType(List.class, User.class);
   List<User> users = mapper.readValue(json, userListType);
   ```
4. **禁用不需要的特性**：减少处理开销
   ```java
   mapper.disable(MapperFeature.USE_ANNOTATIONS);
   mapper.disable(SerializationFeature.FAIL_ON_EMPTY_BEANS);
   ```

## 九、Spring Boot 集成

在 Spring Boot 中自动配置：

```java
@RestController
public class UserController {

    // 自动注入的 ObjectMapper
    @Autowired
    private ObjectMapper objectMapper;

    @GetMapping("/user")
    public User getUser() {
        return new User("John", "john@example.com");
    }
    
    // 自定义配置
    @Bean
    public ObjectMapper objectMapper() {
        return new ObjectMapper()
            .registerModule(new JavaTimeModule())
            .disable(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS);
    }
}
```

## 十、常见问题解决

### 1. 处理日期时间
```java
// 全局配置
mapper.registerModule(new JavaTimeModule());
mapper.disable(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS);

// 单个字段配置
@JsonFormat(pattern = "yyyy-MM-dd HH:mm:ss")
private LocalDateTime createTime;
```

### 2. 处理枚举
```java
public enum Status {
    @JsonProperty("active")
    ACTIVE,
    
    @JsonProperty("inactive")
    INACTIVE
}

// 或全局配置
mapper.enable(SerializationFeature.WRITE_ENUMS_USING_TO_STRING);
mapper.enable(DeserializationFeature.READ_ENUMS_USING_TO_STRING);
```

### 3. 循环引用问题
```java
// 在双向关系中
@JsonIdentityInfo(
    generator = ObjectIdGenerators.PropertyGenerator.class,
    property = "id")
public class User {
    private Long id;
    private List<Order> orders;
}

public class Order {
    private Long id;
    private User user; // 循环引用
}
```

## 总结

Jackson 提供了全方位的 JSON 处理能力：
- 简单对象转换：`writeValueAsString()` 和 `readValue()`
- 树模型：动态处理 JSON 结构
- 流式 API：高性能大数据处理
- 丰富注解：精细控制序列化行为
- 强大扩展：自定义序列化/反序列化

掌握这些 API 后，你可以高效处理任何 JSON 相关任务。对于更高级的功能，参考 [Jackson 官方文档](https://github.com/FasterXML/jackson-docs)。