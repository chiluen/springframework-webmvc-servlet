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