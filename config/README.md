## 讀 code 順序

| 檔案名稱 | Lines of code |
| --- | --- |
| WebMvcConfigurer.java | 58 |
| WebMvcConfigurationSupport.java | 652 |
| DelegatingWebMvcConfiguration.java | 97 |
| EnableWebMvc.java | 13 |
| WebMvcConfigurerComposite.java | 147 |
| InterceptorRegistry.java | 69 |
| InterceptorRegistration.java | 36 |
| ResourceHandlerRegistry.java | 96 |
| ResourceHandlerRegistration.java | 81 |
| ResourceChainRegistration.java | 80 |
| ContentNegotiationConfigurer.java | 84 |
| CorsRegistry.java | 21 |
| CorsRegistration.java | 50 |
| ViewResolverRegistry.java | 162 |
| ViewControllerRegistry.java | 57 |
| ViewControllerRegistration.java | 31 |
| AsyncSupportConfigurer.java | 51 |
| UrlBasedViewResolverRegistration.java | 40 |
| PathMatchConfigurer.java | 134 |
| DefaultServletHandlerConfigurer.java | 36 |
| AnnotationDrivenBeanDefinitionParser.java | 521 |
| ResourcesBeanDefinitionParser.java | 306 |
| MvcNamespaceUtils.java | 197 |
| ViewResolversBeanDefinitionParser.java | 135 |
| ViewControllerBeanDefinitionParser.java | 93 |
| CorsBeanDefinitionParser.java | 62 |
| InterceptorsBeanDefinitionParser.java | 62 |
| ScriptTemplateConfigurerBeanDefinitionParser.java | 58 |
| DefaultServletHandlerBeanDefinitionParser.java | 42 |
| FreeMarkerConfigurerBeanDefinitionParser.java | 32 |
| GroovyMarkupConfigurerBeanDefinitionParser.java | 20 |
| MvcNamespaceHandler.java | 20 |
| (SUM) | 3586 |



## Overview

config package 主要處理的是 Spring 內部的各種配置，主要環繞在 WebMvcConfigurer（介面）與 WebMvcConfigurationSupport（類）。

- 在 Spring 中配置 WebMVC 時有兩種方法：

  - 實現 `WebMvcConfigurer`（重寫裡面相應的方法，但是需要在配置類上添加 `@EnableWebMvc` 註解）
  - 繼承 `WebMvcConfigurationSupport`（重寫裡面相應的方法）

- `WebMvcConfigurer` 與 `WebMvcConfigurationSupport` 的關係：
    
    `WebMvcConfigurer` 配置類是 Spring 內部的一種配置方式，採用 JavaBean 的形式來代替傳統的 xml 配置文件形式進行針對「框架個性化定制」，可以自定義一些 Handler、Interceptor、ViewResolver、MessageConverter。
    
    `WebMvcConfigurer` 只是 `WebMvcConfigurationSupport` 的一個擴展類（`WebMvcConfigurationSupport` 中那些子類可以重寫的空方法在 `WebMvcConfigurer` 都有，它並沒有擴展新功能，只是為讓用戶更方便安全的添加自定義配置）
    
- Why `WebMvcConfigurer`:
    
    安全性考量（如果直接繼承 `WebMvcConfigurationSupport`，那麽用戶可以重寫默認的配置，如果對原理不是很清楚地開發者不小心重寫了默認的配置，springmvc可能相關功能就無法生效，這是一種不安全的行為。但如果是直接實現 `WebMvcConfigurer`，那麼開發者是在默認配置的基礎上添加自定義配置，相對來說更安全一些。從這個角度來說，最佳實踐還是直接實現 `WebMvcConfigurer`）

ref: ****[WebMvcConfigurationSupport 和 WebMvcConfigurer 区别](https://blog.csdn.net/qq_51033936/article/details/128185892?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522168170915816800182770812%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fall.%2522%257D&request_id=168170915816800182770812&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~first_rank_ecpm_v1~rank_v31_ecpm-13-128185892-null-null.142^v83^insert_down38,239^v2^insert_chatgpt&utm_term=WebMvcConfigurer&spm=1018.2226.3001.4187)****

ref: [WebMvcConfigurer](https://so.csdn.net/so/search?q=WebMvcConfigurer&spm=1001.2101.3001.7020)


## WebMvcConfigurer

功能簡介：*defines callback methods to customize the Java-based configuration for Spring MVC enabled via **@EnableWebMvc** annotation* *(set up the basic support we need for an MVC project, such as registering controllers and mappings, type converters, validation support, message converters and exception handling)*

如果想要客製化 configuration，就必須實現 *WebMvcConfigurer* interface

重要元素：

- Handler
- Interceptor
- ViewResolver — 將邏輯視圖名解析為一個具體的視圖對象
- MessageConverter

Key methods（前三者最常用）:

- `addInterceptors(InterceptorRegistry registry)`: 自定義攔截器，並指定攔截路徑。Allows developers to register custom interceptors to intercept HTTP requests and responses.
    - 常見的使用場景：
        - 根據 session 判斷用戶是否登錄（用戶如果未登錄， 除了訪問登錄界面，其他的界面被訪問的時候被攔截且返回到登錄頁面）
        - 效能監控：統計 API 執行所耗費的時間
        
        ref: ****[SpringBoot快速設定攔截器並實現許可權驗證的方法，springboot許可權](https://topic.alibabacloud.com/tc/a/springboot-allows-you-to-quickly-set-the-interceptor-and-implement-permission-verification_1_27_32612904.html)****
        
        ref: ****[Spring Boot由拦截器HandlerInterceptor实现登录认真、API耗时统计和资源拦截](https://www.cnblogs.com/east7/p/10054083.html)****
        
- `addResourceHandlers(ResourceHandlerRegistry registry)`: 自定義資源映射。Allows developers to add custom handlers for serving static resources like CSS, JavaScript, and images.
- `addCorsMappings(CorsRegistry registry)`: 解決跨域問題
    - Cors 是一種機制，告訴我們的後台，哪邊 (origin) 來的請求可以訪問服務器的數據。
- `addViewControllers(ViewControllerRegistry registry)`: 自定義頁面映射，配置 view 視圖映射；實現一個路徑自動跳轉到一個頁面（只是單純的路由過程：點擊一個按鈕跳轉到一個頁面）
- `configureViewResolvers(ViewResolverRegistry registry)`: 配置視圖解析器，主要是配置視圖的前後綴
- `addFormatters(FormatterRegistry registry)`: 數據格式化配置
    - 例如：自定義日期轉換器（可通過 DateFormatter 類來實現自動轉換）

Other methods:

- addArgumentResolvers、addReturnValueHandlers、configureMessageConverters、extendMessageConverters、configureHandlerExceptionResolvers、extendHandlerExceptionResolvers

Example:

- **addInterceptors(InterceptorRegistry registry)**
    
    ```java
    public class MyInterceptor implements HandlerInterceptor {
    
    		// 在目標方法執行前執行
        @Override
        public boolean preHandle(HttpServletRequest request,
                                 HttpServletResponse response, Object handler) throws Exception {
            System.out.println("preHandle,ok,假設給true，運行去吧");
            return true;
        }
    
    		// 在目標方法執行後執行，但在請求返回前，我們仍然可以對 ModelAndView 進行修改
        @Override
        public void postHandle(HttpServletRequest request,
                               HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
            System.out.println("postHandle,ok,看看我什麼時候運行的。");
        }
    
    		// 在請求已經返回之後執行
        @Override
        public void afterCompletion(HttpServletRequest request,
                                    HttpServletResponse response, Object handler, Exception ex) throws Exception {
            System.out.println("afterCompletion,ok.");
        }
    }
    ```
    
    然后配置一下：
    
    ```java
    @Configuration
    public class MyConfigurer implements WebMvcConfigurer {
        @Bean
        public MyInterceptor getMyInterceptor(){
            return  new MyInterceptor();
        }
        @Override
        public void addInterceptors(InterceptorRegistry registry) {
            registry.addInterceptor(this.getMyInterceptor())
            .addPathPatterns("/abc","/configurePathMatch");
        }
    }
    ```
    
    可以看出 `addPathPatterns()` 裡面可以嘗試添加多個路徑，或者寫成 `”/**“`（代表對所有的路徑攔截）
    
    測試一下，輸出：
    
    ```
    preHandle,ok,假設給true，運行去吧
    ===》執行業務邏輯===》
    postHandle,ok,看看我什麼時候運行的。
    afterCompletion,ok.
    ```
    
- **addCorsMappings(CorsRegistry registry)**
    
    全局解決跨域
    
    - 寫一個配置類
    - 在 IOC 容器中創建一個 Bean **WebMvcConfigurer**
    - 重寫 addCorsMappings 方法 並設置參數
    
    ```java
    package cn.yufire.dining.back.confing;
    
    import org.springframework.context.annotation.Bean;
    import org.springframework.context.annotation.Configuration;
    import org.springframework.web.servlet.config.annotation.CorsRegistry;
    import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;
    
    /**
     *
     * <p>
     * Description:跨域請求配置
     * </p>
     *
     * @author yufire
     * @version v1.0.0
     * @since 2020-03-05 22:43:21
     * @see cn.yufire.dining.back.confing
     *
     */
    @Configuration
    public class CORSConfiguration {
    
    //addMapping：配置可以被跨域的路徑，可以任意配置，可以具體到直接請求路徑。
    //allowedMethods：允許所有的請求方法訪問該跨域資源服務器，如：POST、GET、PUT、DELETE等。
    //allowedOrigins：允許所有的請求域名訪問我們的跨域資源，可以固定單條或者多條內容，如："http://www.baidu.com"，只有百度可以訪問我們的跨域資源。
    //allowedHeaders：允許所有的請求header訪問，可以自定義設置任意請求頭信息，如："X-YAUTH-TOKEN"
    
        @Bean
        public WebMvcConfigurer corsConfigurer() {
            return new WebMvcConfigurer() {
                @Override
                public void addCorsMappings(CorsRegistry registry) {
                    registry.addMapping("/**")
                            .allowedOrigins("*")
                            .allowedMethods("GET", "POST", "DELETE", "PUT","PATCH")
                            .allowedHeaders("*")
                            .allowCredentials(true)
                            .maxAge(3600);
                }
            };
        }
    }
    ```
    
- **addFormatters(FormatterRegistry registry)**
    - 首先寫一個轉換日期的util工具類
    - 這邊可以轉換 yyyy-MM-dd      yyyy-MM-dd HH:mm:ss 兩種格式的日期
    
    ```java
    package cn.yufire.wms.util;
    
    import cn.yufire.wms.util.EmptyUtils;
    import org.springframework.core.convert.converter.Converter;
    
    import java.text.ParseException;
    import java.text.SimpleDateFormat;
    import java.util.Date;
    
    /**
     * <p>
     * Description:  自定定義日期轉換器
     * </p>
     *
     * @author yufire
     * @version v1.0.0
     * @see cn.yufire.wms.config
     * @since 2020-05-12 16:27:14
     */
    public class DateConverter implements Converter<String, Date> {
        @Override
        public Date convert(String str) {
            if (EmptyUtils.isEmpty(str) || str.length() <= 0) {
                return null;
            } else {
                Date date = null;
                try {
                    date = new SimpleDateFormat(str.length() > 11 ? "yyyy-MM-dd HH:mm:ss" : "yyyy-MM-dd").parse(str);
                } catch (ParseException e) {
                    // TODO Auto-generated catch block
                    e.printStackTrace();
                }
                return date;
            }
        }
    }
    ```
    
    - 寫一個配置類 繼承 WebMvcConfigurerAdapter
    - 重寫 addFormatters 方法
    - 配置日期轉換類
    
    ```java
    package cn.yufire.wms.config;
    
    import cn.yufire.wms.util.DateConverter;
    import org.springframework.context.annotation.Configuration;
    import org.springframework.format.FormatterRegistry;
    import org.springframework.web.servlet.config.annotation.WebMvcConfigurerAdapter;
    
    /**
     * <p>
     * Description:  配置日期轉換器
     * </p>
     *
     * @author yufire
     * @version v1.0.0
     * @see cn.yufire.wms.config
     * @since 2020-05-12 16:27:02
     */
    @Configuration
    public class WebMvcConfig extends WebMvcConfigurerAdapter {
        @Override
        public void addFormatters(FormatterRegistry registry) {
            registry.addConverter(new DateConverter());
        }
    }
    ```
    

ref: ****[SpringBoot系列——WebMvcConfigurer介绍](https://juejin.cn/post/6844903863640653832)****

ref: ****[跨域问题与SpringBoot解决方案](https://juejin.cn/post/6844903862956982279)****


## **WebMvcConfigurationSupport**

This is the main class providing the configuration behind the MVC Java config. It is typically imported by adding `@EnableWebMvc` to an application `@Configuration` class. An alternative more advanced option is to extend directly from this class and override methods as necessary, remembering to add `@Configuration` to the subclass and @Bean to overridden `@Bean` methods.

This class registers the following `HandlerMappings`:

- `RequestMappingHandlerMapping` ordered at 0 for mapping requests to annotated controller methods.
    - **requestMappingHandlerMapping**(@Qualifier("mvcContentNegotiationManager") ContentNegotiationManager contentNegotiationManager, @Qualifier("mvcConversionService") FormattingConversionService conversionService, @Qualifier("mvcResourceUrlProvider") ResourceUrlProvider resourceUrlProvider)
    - By default, the `WebMvcConfigurationSupport` class provides a `RequestMappingHandlerMapping` instance that is configured with the standard behavior for Spring MVC. This includes supporting `@RequestMapping`, `@GetMapping`, `@PostMapping`, and other HTTP method mapping annotations, as well as support for URI templates, path variables, and more.
- `HandlerMapping` ordered at 1 to map URL paths directly to view names.
    - **viewControllerHandlerMapping**(@Qualifier("mvcConversionService") FormattingConversionService conversionService, @Qualifier("mvcResourceUrlProvider") ResourceUrlProvider resourceUrlProvider)
    - By default, the `WebMvcConfigurationSupport` class provides a `viewControllerHandlerMapping()` instance that is configured with the standard behavior for Spring MVC. This includes support for mapping URLs to view names based on convention (e.g., "/about" maps to the "about" view), as well as the ability to customize the behavior of the handler mapping by overriding the `addViewControllers()` method.
- `BeanNameUrlHandlerMapping` ordered at 2 to map URL paths to controller bean names.
    - **beanNameHandlerMapping**(@Qualifier("mvcConversionService") FormattingConversionService conversionService, @Qualifier("mvcResourceUrlProvider") ResourceUrlProvider resourceUrlProvider)
- `RouterFunctionMapping` ordered at 3 to map router functions.
    - **routerFunctionMapping**(@Qualifier("mvcConversionService") FormattingConversionService conversionService,      @Qualifier("mvcResourceUrlProvider") ResourceUrlProvider resourceUrlProvider)
- `HandlerMapping` ordered at `Integer.MAX_VALUE-1` to serve static resource requests.
    - **resourceHandlerMapping**(@Qualifier("mvcContentNegotiationManager") ContentNegotiationManager contentNegotiationManager,      @Qualifier("mvcConversionService") FormattingConversionService conversionService, @Qualifier("mvcResourceUrlProvider") ResourceUrlProvider resourceUrlProvider)
- `HandlerMapping` ordered at `Integer.MAX_VALUE` to forward requests to the default servlet.
    - **defaultServletHandlerMapping**()

Registers these `HandlerAdapters`:

- `RequestMappingHandlerAdapter` for processing requests with annotated controller methods.
    - **requestMappingHandlerAdapter**(@Qualifier("mvcContentNegotiationManager") ContentNegotiationManager contentNegotiationManager,      @Qualifier("mvcConversionService") FormattingConversionService conversionService, @Qualifier("mvcValidator") Validator validator)
- `HttpRequestHandlerAdapter` for processing requests with `HttpRequestHandlers`.
    - **httpRequestHandlerAdapter**()
- `SimpleControllerHandlerAdapter` for processing requests with interface-based `Controllers`.
    - **simpleControllerHandlerAdapter**()
- `HandlerFunctionAdapter` for processing requests with router functions.

Registers a `HandlerExceptionResolverComposite` with this chain of exception resolvers:

- `ExceptionHandlerExceptionResolver` for handling exceptions through `ExceptionHandler` methods.
- `ResponseStatusExceptionResolver` for exceptions annotated with `ResponseStatus`.
- `DefaultHandlerExceptionResolver` for resolving known Spring exception types

Registers an `AntPathMatcher` and a `UrlPathHelper` to be used by:

- the `RequestMappingHandlerMapping`,
- the `HandlerMapping` for ViewControllers
- and the `HandlerMapping` for serving resources

Note that those beans can be configured with a `PathMatchConfigurer`.

Both the `RequestMappingHandlerAdapter` and the `ExceptionHandlerExceptionResolver` are configured with default instances of the following by default:

- a `ContentNegotiationManager`
- a `DefaultFormattingConversionService`
- an `OptionalValidatorFactoryBean` if a JSR-303 implementation is available on the classpath
- a range of `HttpMessageConverters` depending on the third-party libraries available on the classpath.

## EnableWebMvc

實現原理：實際上是導入了 `DelegatingWebMvcConfiguration` 配置類，等價於 `@Configuration` + 繼承該類
