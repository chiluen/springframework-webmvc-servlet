## ModelAndView.java
ModelAndView提供一個容器，可以同時包含Model以及View，且這個檔案可由View or Model去Construct，並且還提供修改Model的method等等。之所以要把Model與View綁在通一個Class是因為controller可以同時return Model與View

## ModelAndViewDefiningException.java
提供一個class，使用ModelAndView object進行init，用來檢驗該ModelAndView object是否為Null

## View.java
提供View的interface，最重要的為其中的render function，用來定義如何與Model配合並且render當前畫面

## ViewResolver.java
解析當前View的name屬性，當controller回傳多個view時，就是靠resolver去辨別當前的view所對應的名稱

ref:https://www.cntofu.com/book/84/spring/spring-view-resolver.md

## HandlerAdapter.java
為一個interface，提供service provider interface（調用Handler）給MVC framework。這個檔案有三個主要function：
- boolean supports(Object handler): 根據handler的類別，確認handleradapter是否可以處理該handler
- ModelAndView handle(...)：根據request, response和handler，回傳對應的Model與View
- long getLastModified(HttpServletRequest request, Object handler)：回傳該handler是否有modified過

## HandlerExceptionResolver.java
為一個interface，處理handler在運行時是否有exception，會回傳ModelandView，若有錯誤則會回傳一個error page，若回傳empty則代表該ModelandView不需更改


## HandlerExecutionChain.java
結合Handler以及HandlerInterceptor，檢查是否有HandlerInterceptor的存在以及其屬性（為preHanle or postHanle or afterCompletion），並照順序形成一個執行順序，裡面幾個重要function:

- applyPreHandle():針對handler執行prehandle
- applyPostHandle(): 針對handler執行posthandle
- triggerAfterCompletion():針對handler執行afterCompletion

## HandlerInterceptor.java
為一個interface，對Handler做預處理或後處理，因此無須對Handler本身做更改就可以透過這一個物件做一些想要的處理。主要有以下三個function
- boolean preHandle(): 在HandlerMapping成功找出特定的handler後，在handler執行之前對handler做預處理
- void postHandle():在handeler執行之後，對handler做後處理（時間點在rendering之前）
- void afterCompletion(): 在DispatcherServlet render之後，對handler做額外處理

ref: https://blog.csdn.net/securitit/article/details/111189938

## HandlerMapping.java
為一個interface，根據request去尋找對應的handler，重要的function:
- HandlerExecutionChain getHandler(HttpServletRequest request)：根據request，回傳一個handler執行順序（此執行順序包含handlerinterceptor）

## HttpServletBean.java
這一個檔案被FrameworkServlet.java繼承，而DispatcherServlet.java繼承自FrameworkServlet.java

Servlet這個詞代表對client所發出的Request提供Response的一個容器（ref:[link1](https://worktile.com/kb/ask/30170.html), [link2](https://zh.wikipedia.org/zh-tw/Java_Servlet)）

HttpServletBean是abstract class，並且implement EnvironmentCapable和EnvironmentAware，這兩個的作用為設定web環境的properties

- setEnvironment/getEnvironment: 實現它所implement的兩個interface，目的為設定enviroment
- init: 初始化Servlet，若Servlet的Config不為空，則根據Config的properties設定Beanwrapper等
- initBeanWrapper: 可自定義的init method，BeanWrapper本身為JavaBean的延伸概念，其概念為將不同物件封裝到一個物件中
- ServletConfigPropertyValues: 為一個private的class，用來存放properties，這一個class有在init時被用到

ref: [link](https://zhuanlan.zhihu.com/p/33372365)

## NoHandlerFoundException.java
在DispatcherServlet找不到request所對應的handler時會有兩個處理方式，第一個為回傳404 not found，第二個則為調用此class(若DispatcherServlet有設定 throw exception的話)

這個class會根據目前的statu以及錯誤的訊息內容（by getBody method）回傳較豐富的資訊給client

## ThemeResolver.java
為一個interface，根據request resolve對應的theme名字，同時也可以根據request改變theme名字

## LocaleResolver.java
Locale的意思為區域，由於不同國家區域可能有不同的限制，因此LocaleResolver通過解析client所在的區域並給出對應的View

LocaleResolver為一個interface
- resolveLocale: 根據client的request判定所在區域
- setLocale: 判定Locale後，針對Locale給予不同的View

## LocaleContextResolver
為Interface，extend from LocaleResolver，為LocaleResolver的延伸interface，有更具體的說怎麼判別Locale，例如使用Timezone去辨別Locale
(ex: TimeZoneAwareLocaleContext)，並且這個interface有具體實現resolveLocale和setLocale(透過Java的default關鍵字)

## SmartView.java
繼承自View.java的interface，相較於View.java多了一個函式判斷，該View是否為redirect過來的

## RequestToViewNameTranslator.java
為一個interface，作用為根據Http request將其轉換為對應的View的名稱

## FrameworkServlet.java
為Abstract，extend from HttpServletBean.java

此class重點為Host一個WebApplicationContext，並且依據Client所傳入的request做出對應的處理，底下為FrameworkServlet重要的functions
- FrameworkServlet(WebApplicationContext): init一個FrameworkServlet
- service: 處理Client請求的入口
- processRequest: 真正處理Client請求的method，會透過內部的doService method處理請求


底下講解上述三個methods做的事情：
- FrameworkServlet: init FrameworkServlet的method，有不同種的init方式，主要是透過傳入WebApplicationContext進行init，而這個WebApplication是這個servelet主要host的物件
接下來要把每個function做解析
- service: 根據request，丟到processRequest或
super.service處理，具體的處理方法在FrameworkServlet有實現，例如doGet, doPost, doDelete等等，然而這一些doGet與doPost仍會在method之中將請求丟給processRequest處理，因此無論是super.service或processRequest處理，都會丟到processRequest辨別request的類型並且處理請求
- processRequest: 此method為FrameworkServlet最重要的method，一共有三個部分。
    - 第一個部分為對LocaleContext 和 RequestAttributes做處理，其中LocaleContext是獲取request的locale(地區)資訊，而RequestAttributes則是或許該request的不同訊息(如用戶id等)
    - 第二個部分為doService，會根據不同的httprequest做不同的事情，而其具體的實現在DispatcherServlet
    - 第三個部分(在finally的區塊)為根據request，發布一個事件，說明Servelet有針對request做處理

ref: 
- [link1](https://blog.csdn.net/u012702547/article/details/115108949)
- [link2](https://blog.csdn.net/f641385712/article/details/87982095)

## AsyncHandlerInterceptor.java
為interface，擴展HandlerInterceptor至非同步的應用

## FlashMap.java
為一個Class。當網頁重定向(redirect)時，通常現有網站的參數(attributes)沒辦法簡單的跟著傳過去（例如User ID等等），常見的做法是將attributes作為URL的參數存下來，但這樣可能會有secruity等issue，因此FlashMap幫助網站在重定向時，儲存想要的參數至Target URL

## FlashMapManager.java
為一個interface，作用為儲存和查找對應的FlashMap