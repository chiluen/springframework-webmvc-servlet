
# *Under root folder*

## HandlerFunction.java
為一個interface，為透過'function形式'表達的handler，裡面也有**handle** function

## RouterFunction.java
為一個class，routerfunction的目的為透過該function映射到不同的handler function，可以想像成這是一種不同的handler mapping的形式，是透過routerfunction進行mapping
- route: 主要的function，可route到不同的handler function

## ServerRequest.java
為一個interface，hold by HandlerFunction，主要的作用為若有一個http request，會有這個ServerRequest去解析該request並傳送到server，裡面提供對http的head, body訪問的method。
- body(Class<T> bodyType): 取得http request的body內容

該份code裡有數個關於http request的interface:
- Headers
- Builder

** builder為一個design pattern，目的是把class的創建過程（包含創建params）使用另外一個class操作，而這另外一個class即為builder。例如class A創建過程過於複雜，我們將他的constuctor設為private並且只讓builder class去access，就可以讓builder進行操作


ref: [link](https://blog.csdn.net/qq_43605444/article/details/122148706)

## ServerResponse.java
為一個interface，hold by handlerfunction，主要作用為幫助server將需要回傳的response打包，裡面包含不同種的http status，還有response的body與header。
該interface包含了以下的builder幫助其contruct一個response:
- HeadersBuilder
- BodyBuilder
- SseBuilder: server side event，代表server自動傳送給client的資訊，可以想像成是某個網站對你的推播
- Context: server回傳response時，同時也可能會渲染畫面，因此有'writeTo'這個function回傳ModelAndView，而該context就是ModelAndView所需的原料之一


# *Under Support folder*

## HandlerFunctionAdapter.java
為一個class，implement handleradapter，並且該adapter服務的對象為HandlerFunction（在function資料夾中）。其主要做的事情為：
- handle一個handlerfunction，根據request吐回一個ModelAndView
- maintain該handlerfunction的order（在Dispatchservlet 排序的位置）
- 同時也會處理async等的問題

## RouterFunctionMapping.java
為一個class，是handler mapping的一種，但是是透過routerfunction mapping到對應的handler

會根據(or 偵測)目前context有的routerfunction
- initRouterFunctions: 偵測當前application有的routerfunction