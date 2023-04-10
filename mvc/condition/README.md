(1495 LoC)

## 讀 code 順序

- `RequestCondition`（介面）
    
    由 `AbstractRequestCondition` 實踐（是 `RequestCondition` 的 base class）
    
    - `RequestCondition` 的具體實現都繼承自 `AbstractRequestCondition`，都是針對請求匹配的某一個方面：請求路徑，請求頭部，請求方法，請求参數，可消費`MIME`
    ，可生成`MIME`等等。包括：
        - `CompositeRequestCondition`
        - `ConsumesRequestCondition`
        - `ProducesRequestCondition`
        - `RequestMethodsRequestCondition`
        - `ParamasRequestCondition`
        - `HeadersRequestCondition`
        - `PatternsRequestCondition`
        - `PathRequestCondition`
    - 框架還提供了另一個實現類 `RequestConditionHolder`（繼承自 `AbstractRequestCondition`），是一個匹配條件持有器，用於持有某個 `RequestCondition` 對象。如果我們想持有一個 RequestCondition 對象，但事先不知道它的類型，在這種情況下這種工具就很有用
- `NameValueExpression` （介面）
    - `AbstractNameValueExpression` （實踐）
- `MediaTypeExpression`（介面）
    - `AbstractMediaTypeExpression`（實踐）


## Overview

The package are used to handle different aspects of incoming **HTTP requests** (e.g., HTTP method, media types, request parameters, headers, and path patterns) based on various **conditions**. The goal is to determine if the incoming request matches the conditions configured in the Spring MVC framework and thus should be handled by the corresponding controller method.

- 如何將請求映射到對應的控制器方法中是 Spring MVC 框架最重要的任務之一，這項任務由 `@RequestMapping` 負責
- Dispatcher Servlet 接獲請求後，就會透過控制器上 `@RequestMapping` 提供的映射訊息確定請求所對應的處理方法
- 將請求映射到控制器處理方法的工作包含一系列的映射規則，這些規則則是根據請求中的各種訊息制定的，具體包括請求 URL、請求參數、請求方法、請求頭這四個方面的訊息項


## Spring MVC 概述與大致流程（condition package 位在 HTTP 請求處理階段）

Spring MVC 框架圍繞著 `DispatcherServlet` 這個前端 Servlet 展開，它是 Spring MVC 的核心，負責截獲請求並將其分派給相應的 handler 處理，協調和組織不同組建以完成請求並返回響應的工作。

> 在瀏覽器-伺服器的互動過程中，Spring MVC 起著「郵局」的作用。它一方面會從瀏覽器接收各種各樣的「來信」（HTTP 請求），並把不同的請求分發給對應的服務層進行業務處理；另一方面會傳送「回信」（HTTP 響應），將伺服器處理後的結果迴應給瀏覽器。
> 

因此，開發人員就像是「郵遞員」，主要需要完成三方面工作：

- **指定分發地址**：使用`@RequestMapping`等註解指定不同業務邏輯對應的URL。
- 接收請求資料：使用`@RequestParam`等註解接收不同型別的請求資料。
- 傳送響應資料：使用`@ResponseBody`等註解傳送不同型別的響應資料。

而 condition package 的角色大概是對應到「**指定分發地址**」

當一個請求發出來時，首先由 `DispatcherServlet` 接收，根據請求的訊息（包括 URL、HTTP 方法、請求 header、請求參數等）及 `HandlerMapping` 的配置找到處理請求的 handler。

- `HandlerMapping` 是一個 interface，定義一些策略以把 HTTP 請求 map 到 handler，它的其中一個具體實踐是 `RequestMappingHandlerMapping`。
- `RequestMappingHandlerMapping` 負責決定哪一個 handler method 適合 handle 這個請求，它主要針對控制器類（帶有註解 `@Controller`）中類級別或者方法級別的註解 `@RequestMapping` 創建 `RequestMappingInfo` 並管理。
    - 在 `RequestMappingHandlerMapping` 的 `match()` 中，會間接使用 `RequestMappingInfo` 的 `getMatchingCondition()`，透過`RequestMappingInfo` 自身保存的各個 class 的條件訊息對當前請求進行匹配，如果其中有一個條件不滿足，直接 return null，不會加入 match 集合中。
    - `RequestMappingInfo` 的 constructor（可從中看出它與 condition folder 下的那些 XXXcondition class 的關係）
        
        ```java
        public RequestMappingInfo(@Nullable String name, @Nullable PatternsRequestCondition patterns,
        			@Nullable RequestMethodsRequestCondition methods, @Nullable ParamsRequestCondition params,
        			@Nullable HeadersRequestCondition headers, @Nullable ConsumesRequestCondition consumes,
        			@Nullable ProducesRequestCondition produces, @Nullable RequestCondition<?> custom) {
        
        		this(name, null,
        				(patterns != null ? patterns : EMPTY_PATTERNS),
        				(methods != null ? methods : EMPTY_REQUEST_METHODS),
        				(params != null ? params : EMPTY_PARAMS),
        				(headers != null ? headers : EMPTY_HEADERS),
        				(consumes != null ? consumes : EMPTY_CONSUMES),
        				(produces != null ? produces : EMPTY_PRODUCES),
        				(custom != null ? new RequestConditionHolder(custom) : EMPTY_CUSTOM),
        				new BuilderConfiguration());
        	}
        ```


## Class 分類與功能簡介

- **Request condition classes**:
    - 功能：to provide a modular and extensible framework for handling different types of requests in Spring MVC. Each request condition class is responsible for evaluating a specific condition related to the incoming request.
    - 包含的 class 名稱：（他們全部主要都是要做到 **parsing** 跟 **matching** 的功能，藉此再經由 `@RequestMapping` annotations — on the controller methods — 找到對應的 controller method；他們都繼承自 `AbstractRequestCondition` class）
        - `CompositeRequestCondition`: combines multiple request conditions into one composite condition.
        - `ConsumesRequestCondition`: handles requests based on the media type that the handler method can accept or consume a specific type of content (media type) as part of an HTTP request.
        - `ProducesRequestCondition`: handles requests based on the media type that the handler method can generate a specific type of content (media type) in response to an HTTP request.
        - `RequestMethodsRequestCondition`: handles requests based on HTTP methods such as GET, POST, PUT, DELETE, etc.
        - `ParamasRequestCondition`: handles requests based on URL parameters.
        - `HeadersRequestCondition`: handles requests based on headers in the HTTP request.
        - `PatternsRequestCondition`: match a handler method to a request based on the URL pattern(s) specified in the `@RequestMapping` annotation.
        - `PathRequestCondition`: match a handler method to a request based on the URI path of the request.
    
    Note: `@RequestMapping` is an annotation used to map HTTP requests to controller methods. You can find it in the `org.springframework.web.bind.annotation` package.
    
    [https://github.com/spring-projects/spring-framework/tree/main/spring-web/src/main/java/org/springframework/web/bind/annotation](https://github.com/spring-projects/spring-framework/tree/main/spring-web/src/main/java/org/springframework/web/bind/annotation)
    

| 實現類 (class) | 簡介 |
| --- | --- |
| PatternsRequestCondition | 路徑匹配條件 |
| RequestMethodsRequestCondition | 請求方法匹配條件 |
| ParamsRequestCondition | 請求參數匹配條件 |
| HeadersRequestCondition | 頭部信息匹配條件 |
| ConsumesRequestCondition | 可消費 MIME 匹配條件 |
| ProducesRequestCondition | 可生成 MIME 匹配條件 |

**Example**:

- "`pattern`":
    - if a handler method has a `@RequestMapping` annotation with a `value` attribute that specifies `/users/{id}`, the `PatternsRequestCondition` will match any request whose URL starts with `/users/` and has a path variable named `id`. The `PatternsRequestCondition` can also handle more complex patterns using wildcards, regex expressions, or custom path matchers.
- "`request method`":
    - if a handler method has a `@RequestMapping` annotation with a `method` attribute that specifies `RequestMethod.POST`, the `RequestMethodsRequestCondition` will match only requests that use the `POST` method. The `RequestMethodsRequestCondition` can also handle multiple HTTP methods using comma-separated values or `RequestMethod` enums.
- "`parameter`":
    - if a handler method has a `@RequestMapping` annotation with a `params` attribute that specifies `foo=bar`, the `ParamsRequestCondition` will match any request that includes a parameter named `foo` with a value of `bar`. If the request does not include this parameter or has a different value, the `ParamsRequestCondition` will not match the request to the handler method.
- "`headers`":
    - if a handler method has a `@RequestMapping` annotation with a `headers` attribute that specifies `Accept=application/json`, the `HeadersRequestCondition` will match only requests that include an `Accept` header with a value of `application/json`. The `HeadersRequestCondition` can also handle multiple headers using logical operators such as AND and OR.
- "`consume`":
    - if a handler method consumes `application/json` and `application/xml` content types, and a client sends a request with a `Content-Type` header of `application/json`, the server can use the `ConsumesRequestCondition` to determine that the handler method can consume the requested content type and invoke the appropriate method to handle the request.
- "`produce`":
    - if a handler method produces `application/json` and `application/xml` content types, and a client sends a request with an `Accept` header of `application/json`, the server can use the `ProducesRequestCondition` to determine that the handler method can produce the requested content type and generate an appropriate response.

**補充**：

- 當我們要發 request 時候，我們需要在 header 中設定 Content Type ，告訴對方說「嘿！我送給你的資料是這種類型」，讓對方可以用適合的方式解析處理你所提供的資料。Content Type 中有個屬性 `media type`，讓我們填寫所傳送的資料類型，而我們必須根據 `MIME` 格式填寫，不可以亂填。
- `MIME` 是一種標準，用來表示文件、檔案、各種位元組 ex. image/jpg
    
    主要類別 ( type )：
    
    - application 適用多數 二進制資料 (binary data) ex. application/pdf
    - audio 適用音訊類資料 ex. audio/mpeg
    - font 適用各類文字型檔案 ex. font/ttf
    - image 適用各種圖片行檔案 ex. image/jpg
    - model 適用各種模型累資料, 3D object ex. model/3mf
    - text 適用各種人類可閱讀之文字資料 ex. text/html
    - video 適用各種影像資料ex. video/mp4
    
    ****[Spring MVC 概念模型 : 接口 RequestCondition](https://blog.csdn.net/andy_zhang2007/article/details/88913776)****
    
- The `Accept` request HTTP header indicates which content types, expressed as `MIME` types, the client is able to understand.


## Detailed explanation of .java
- `RequestCondition.java` (*interface*) *介面(Interface)主要表示抽象的行為，不能初始化屬性和方法，當類別實踐(implement)介面時就可以具體化行為了。*
    - represents a mapping between a request path, method, and other conditions, and a controller method that should handle requests that match these criteria. This is the key class in the MVC request mapping infrastructure, and is used extensively throughout the Spring Web MVC framework.
        - Request conditions are represented by instances of classes in this package, such as `ConsumesRequestCondition`, `HeadersRequestCondition`, `ParamsRequestCondition`, etc. These conditions can be combined with each other to create more complex conditions using the `combine()` method.
        - Once a set of request conditions has been created, it can be matched against an incoming HTTP request to determine if the request meets the conditions. This is done using the `getMatchingCondition(HttpServletRequest)` method, which takes the incoming request as a parameter and returns a matching request condition if one is found.
        - When multiple request conditions are applied to a request, it's possible that more than one condition will match. In this case, it's necessary to determine which condition is the best match for the request. This is done using the `compareTo(Object, HttpServletRequest)` method, which takes two request conditions as parameters and compares them to each other to determine which is a closer match for the given request.
- `AbstractRequestCondition.java` (abstract class)
    - *A base class for `RequestCondition` types providing implementations of `equals(Object)`, `#hashCode()`, and `#toString()`.*
    - represent different types of conditions that can be used to match incoming requests to controller methods. These include `HeadersRequestCondition`, `ProducesRequestCondition`, and `RequestMethodRequestCondition`
    - Method:
        - `getToStringInfix()`: an abstract method that must be implemented by all subclasses of `AbstractRequestCondition`. Its purpose is to provide a string that will be used to join the string representations of the different attributes of the request condition.
            
            The implementation of the `getToString()` method in the `AbstractRequestCondition` class uses the value returned by `getToStringInfix()` to join the string representations of the different attributes. Subclasses of `AbstractRequestCondition` can override `getToStringInfix()` to provide their own custom infix string that is used to join the string representations of their attributes.
            
            The notation to use when printing discrete items of content.
            
            For example `" || "` for URL patterns or `" && "` for param expressions.
            
- `CompositeRequestCondition.java`
    - *Implements the `RequestCondition` contract by delegating to multiple `RequestCondition` types and using a logical conjunction (`&&`) to ensure **all** conditions match a given request.*
    - represents a combination of multiple request conditions. This class is used to create complex request mapping rules that take into account multiple conditions, such as a combination of request method, headers, and media types.
- Other classes: These classes are used to define and evaluate request conditions, which are used by Spring MVC to determine which controller method or handler mapping should handle an incoming request.
    - `ConsumesRequestCondition`:
        - *A logical disjunction (`||`) request condition to match a request's 'Content-Type' header to a list of media type expressions. Two kinds of media type expressions are supported, which are described in `RequestMapping#consumes()` and `RequestMapping#headers()` where the header name is 'Content-Type'. Regardless of which syntax is used, the semantics are the same.*
        - Nested class:
            - `ConsumeMediaTypeExpression`: It represents an individual media type expression within the `ConsumesRequestCondition`. The `ConsumesRequestCondition` class itself manages a list of `ConsumeMediaTypeExpression` instances to represent the media types that a request can consume. So, `ConsumeMediaTypeExpression` is used as a building block within `ConsumesRequestCondition` to build up the list of acceptable media types.
        - Methods:
            - `parseExpressions()`: to parse a list of media type expressions and convert them into `MediaTypeExpression` objects, so that the `ConsumesRequestCondition` is able to easily match incoming requests against a list of supported media types, taking into account any wildcards or other conditions that may be specified in the expressions.
            - `getExpressions()`: return the contained MediaType expressions
            - `getConsumableMediaTypes()`: Returns the media types for this condition excluding negated expressions.
                
                "negated expressions" refer to media type expressions that have been negated using the `!` operator.
                
                For example, consider the following `@RequestMapping` annotation:
                
                ```kotlin
                @RequestMapping(value = "/example", consumes = {"text/plain", "!application/json"})
                ```
                
                This specifies that the `"/example"` endpoint can only consume requests with a media type of `text/plain`, but not `application/json`. The `!application/json` expression is negated, indicating that it is not an acceptable media type for the endpoint.
                
                In the `ConsumesRequestCondition` class, negated expressions are handled by checking for the presence of the `!` operator when parsing the list of media type expressions using the `parseExpressions()` method. If a negated expression is detected, it is added to a separate list of "not consumable" media types.
                
                The `getConsumableMediaTypes()` method then returns a list of media types that are consumable by the endpoint, excluding any media types that have been negated. This allows the `ConsumesRequestCondition` to correctly match incoming requests against a list of supported media types, taking into account any negated expressions that may be specified in the endpoint's `@RequestMapping` annotation.
                
            - `setBodyRequired()`: to set a flag indicating whether a request body is required for the condition to be a match.
                
                Whether this condition should expect requests to have a body. By default this is set to `true` in which case it is assumed a request body is required and this condition matches to the "Content-Type" header or falls back on "Content-Type: application/octet-stream".
                
                If set to `false`, and the request does not have a body, then this condition matches automatically, i.e. without checking expressions.
                
            - `combine()`: combine two `ConsumesRequestCondition` objects into a single `ConsumesRequestCondition` object. This method is used when merging the `consumes` conditions of two `RequestMappingInfo` objects, for example when a handler method is inherited from a superclass and has its own set of `consumes` conditions.
                
                The `combine()` method returns a new `ConsumesRequestCondition` object that represents the intersection of the `mediaTypes` of the two input objects. If either of the input objects has a `isWildcard()` flag set to `true`, then the resulting `ConsumesRequestCondition` will also have this flag set to `true`. If both input objects have the `isWildcard()` flag set to `true`, then the resulting `ConsumesRequestCondition` will have a `mediaTypes` list that contains a single `MediaType.ALL` element.
                
                Here is an example usage of `combine()`:
                
                ```java
                ConsumesRequestCondition cond1 = new ConsumesRequestCondition("application/json");
                ConsumesRequestCondition cond2 = new ConsumesRequestCondition("application/xml");
                
                // Combine the two conditions
                ConsumesRequestCondition combined = cond1.combine(cond2);
                
                // Resulting condition will have the mediaTypes ["application/json", "application/xml"]
                ```
                
                In this example, `cond1` and `cond2` represent `ConsumesRequestCondition`  objects that each specify a single media type. The `combine()` method is used to create a new `ConsumesRequestCondition` object that represents a request condition that matches if the request's content type is either `application/json` or `application/xml`.
                
            - `getMatchingCondition()`: Checks if any of the contained media type expressions match the given request 'Content-Type' header and returns an instance that is guaranteed to contain matching expressions only.
    - `HeadersRequestCondition`: A request condition that checks whether the request headers match a set of header expressions.
    - `AbstractMediaTypeExpression` is an abstract base class for media type expressions, such as the `Accept` header in an HTTP request.
        - The class provides some common functionality for subclasses, such as parsing the media type from a string representation, generating a hash code based on the media type, and providing a human-readable representation of the expression.
        - `AbstractMediaTypeExpression` defines two abstract methods that subclasses must implement: `getMediaType()` and `getQualityValue()`. These methods return the media type and quality value of the expression, respectively.
    - `AbstractNameValueExpression` is an abstract base class for expressions that consist of a name and a value, such as request headers, request parameters, or request attributes.
        - The class provides some common functionality for subclasses, such as parsing the name and value from a string representation, generating a hash code based on the name and value, and providing a human-readable representation of the expression.
        - `AbstractNameValueExpression` defines two abstract methods that subclasses must implement: `getName()` and `getValue()`. These methods return the name and value of the expression, respectively.
    - `MediaTypeExpression` is a subclass of `NameValueExpression` that represents a media type expression, such as the `Accept` header in an HTTP request.
        - The `MediaTypeExpression` class defines a `getMediaType()` method that returns a `MediaType` object representing the media type specified in the expression. It also defines a `compareTo()` method that allows media type expressions to be sorted based on their quality values.
        - The `MediaTypeExpression` class is used by Spring MVC to determine the best match between the media types specified in an incoming request and the media types supported by a controller method or handler mapping.
    - `NameValueExpression` is an abstract base class for expressions that consist of a name and a value, such as request headers, request parameters, or request attributes.
        - The `NameValueExpression` class has two subclasses: `NameValueMatchExpression` and `NameValueRegexExpression`. The former represents an exact match between the expression value and a given string, while the latter represents a regular expression match.
        - The `NameValueExpression` class defines a `match()` method that takes a `Map<String, String>` and checks whether the expression matches any of the entries in the map. If the expression matches, the `match()` method returns a `NameValueExpression.MatchResult` object that contains the matched name and value.
    - `ParamsRequestCondition`: A request condition that checks whether the request parameters match a set of parameter expressions.
    - `PathPatternsRequestCondition`: a request condition that checks whether the request path matches a set of URL path patterns.
        - The class is used by Spring MVC to determine whether a controller method or handler mapping should handle an incoming request based on its URL path.
        - The `PathPatternsRequestCondition` class extends the `AbstractRequestCondition<PathPatternsRequestCondition>` class, which provides some common functionality for request conditions that check URL paths.
        - The `PathPatternsRequestCondition` class has a constructor that takes a variable number of URL path patterns as arguments. The class stores these patterns in a `LinkedHashSet` for fast lookup.
        - The `PathPatternsRequestCondition` class overrides several methods of the `AbstractRequestCondition` class, including `getMatchingCondition()` and `combine()`. These methods are used by Spring MVC to compare different request conditions and choose the best match for an incoming request.
        - The `PathPatternsRequestCondition` class also has a `toString()` method that returns a human-readable representation of the request condition.
    - `PatternsRequestCondition`: A request condition that checks whether the request path matches a set of URL patterns.
    - `ProducesRequestCondition`: A request condition that checks whether the request accepts a given list of media types.
        - to represent the "produces" condition for a request mapping. It is responsible for parsing and evaluating the "Accept" header in the HTTP request, to determine whether the request can be handled by the given controller method.
            
            The class contains a list of media type expressions that are declared in the `@RequestMapping` annotation or in the `@RestController` annotation at the controller level. It then compares the media types specified in the `Accept` header of the request with the list of media types declared in the controller method.
            
            Based on this comparison, the class provides methods to determine whether a request is a match for a particular controller method or not. The class also provides a `combine()` method to combine multiple `ProducesRequestCondition` objects into a single composite condition.
            
    - `RequestConditionHolder`: A container for a request condition and associated metadata, such as a controller method or handler mapping.
        - The `RequestConditionHolder` class is used to associate a `RequestMappingInfo`
         with a specific controller method or handler mapping, and the `RequestMappingInfoHandlerMapping` class is used to map incoming requests to the appropriate `RequestMappingInfo`.
    - `RequestCondition`: A marker interface for classes that define request conditions.
    - `RequestMethodsRequestCondition`: A request condition that checks whether the request method matches a set of HTTP methods.
    - `RequestMappingInfo`: A class that encapsulates the mapping information for a controller method or handler mapping, including the request path patterns, HTTP methods, request parameters, headers, and content type.
        - The `RequestMappingInfo` class is particularly important, as it encapsulates all of the mapping information for a single controller method or handler mapping. The other classes in the package provide various types of request conditions that can be used to construct a `RequestMappingInfo` instance. (located in the `handler` package of the `org.springframework.web.servlet.mvc.method` package)
    - `RequestMappingInfoHandlerMapping`: A handler mapping that maps requests to controller methods based on the `RequestMappingInfo` associated with each method. (located in the `org.springframework.web.servlet.mvc.method.annotation`
     package)
