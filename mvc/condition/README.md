(1495 LoC)

### Overview
The package are used to handle different aspects of incoming HTTP requests (e.g., HTTP method, media types, request parameters, headers, and path patterns) based on various conditions. The goal is to determine if the incoming request matches the conditions configured in the Spring MVC framework and thus should be handled by the corresponding controller method.

### Hierarchical structure（父子關係）

- `RequestCondition.java` 是起始（介面）
    - 由 `AbstractRequestCondition.java` 實踐（是 `RequestCondition.java` 的 base class）
        - 由 `CompositeRequestCondition.java` 繼承 `AbstractRequestCondition.java`
        - 由 `ConsumesRequestCondition.java` 繼承 `AbstractRequestCondition.java`
        - 由 `ProducesRequestCondition.java` 繼承 `AbstractRequestCondition.java`


### Class 分類

- **Request condition classes**:
    - 功能：to provide a modular and extensible framework for handling different types of requests in Spring MVC. Each request condition class is responsible for evaluating a specific condition related to the incoming request.
    - 包含的 class 名稱：（他們全部主要都是要做到 parsing 跟 matching 的功能，藉此找到對應的 controller method；他們都繼承自 AbstractRequestCondition class）
        - `CompositeRequestCondition`: combines multiple request conditions into one composite condition.
        - `ConsumesRequestCondition`: handles requests based on the media type that the request can consume.
        - `ProducesRequestCondition`: handles requests based on the media type that the request can produce.
        - `RequestMethodsRequestCondition`: handles requests based on HTTP methods such as GET, POST, PUT, DELETE, etc.
        - `ParamasRequestCondition`: handles requests based on URL parameters.
        - `HeadersRequestCondition`: handles requests based on headers in the HTTP request.
        - `PatternsRequestCondition`: checks if the request matches any of the defined patterns.
        - `PathRequestCondition`: checks if the request's path matches a given pattern or path pattern.

### Detailed explanation of .java
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
