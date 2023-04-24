## *Under Support folder*

### HandlerFunctionAdapter.java
為一個class，implement handleradapter，並且該adapter服務的對象為HandlerFunction（在function資料夾中）。其主要做的事情為：
- handle一個handlerfunction，根據request吐回一個ModelAndView
- maintain該handlerfunction的order（在Dispatchservlet 排序的位置）
- 同時也會處理async等的問題

### RouterFunctionMapping.java
為一個class，是handler mapping的一種，但是是透過routerfunction mapping到對應的handler

會根據(or 偵測)目前context有的routerfunction
- initRouterFunctions: 偵測當前application有的routerfunction





## *Under root folder*

### HandlerFunction.java
為一個interface，為透過'function形式'表達的handler，裡面也有**handle** function

### RouterFunction.java
為一個class，routerfunction的目的為透過該function映射到不同的handler function，可以想像成這是一種不同的handler mapping的形式，是透過routerfunction進行mapping
- route: 主要的function，可route到不同的handler function