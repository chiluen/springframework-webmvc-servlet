## *Under Support folder*

### HandlerFunctionAdapter
為一個class，implement handleradapter，並且該adapter服務的對象為HandlerFunction（在function資料夾中）。其主要做的事情為：
- handle一個handlerfunction，根據request吐回一個ModelAndView
- maintain該handlerfunction的order（在Dispatchservlet 排序的位置）
- 同時也會處理async等的問題




## *Under root folder*