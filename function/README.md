
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
- DefaultServerResponseBuilder: 實際創立server response的builder

## ErrorHandlingServerResponse.java
為一個abstract，implement ServerResponse。幫ServerResponse實現了error handling，包含回傳的ModelAndView和serverResponse的error handle
** 一個abstract implement interface時，不需要全部methods都implement

## AbstractServerResponse.java
為一個abstract，extend from ErrorHandlingServerResponse（其實就是實踐ServerResponse）。

底下為該class具體實踐ServerResponse的method（這個class只有實現ServerResponse部分的東西）：
- writeTo: 把一段response的header, status, cookiews寫進回覆當中

## AsyncServerResponse.java
為一個interface創建一個async的serverresponse，會透過DefaultAsyncServerResponse創建一個response

## DefaultAsyncServerResponse.java
為一個class，繼承ErrorHandlingServerResponse並implementAsyncServerResponse。

裡面包含methods for創建ServerResponse，並且同時使用block method處理async的狀況

幾個重要的methods:
- writeTo: 由於是async版本，因此會額外呼叫writeAsync去控管非同步問題
- createDeferredResult: 根據async的狀況，創建一個response

## DefaultServerResponseBuilder.java

為一個class，implement ServerResponse.java中的BodyBuilder interface。其作用為將一個response的body建構出來，包含content, cookies, header等

## SseServerResponse.java

為一個class，implement AbstractServerResponse。

sse的意思為server-sent-event，這概念為server主動通知client(瀏覽器)有新的資料，例如股市頁面等。這個功能與websocket相似，但websocket是client與server雙向，而sse則是server to client單向。

此class定義了如何create一個response，其中包含底下幾個class
- DefaultSseBuilder: 去實踐ServerResponse裡面的SseBuilder，為定義Sse的行為與資料，例如send, data等

## EntityResponse.java

為一個interface，implement ServerResponse。

這個class與ServerResponse相似，但他回傳的東西會被封裝為一個entity，此entity可能是任何的type。舉個例子，若想從database retrieve data，就可以根據data的屬性包成一個entity，並且當作server的response回傳給client。（一般來說ServerResponse就只回傳一些基本的content和ModelAndView，不一定會回傳一些object）

## DefaultEntityResponseBuilder.java

為一個class，implement EntityResponse之中的Builder。

這個Builder便是把資料封裝成一個entity，並且回傳給client

其中重要的method or attributes:
- this.entity: 將entity存於該attribute之中
- DefaultEntityResponse: 這是一個private class，當builder build時，會回傳這個class，裡面會設定header以及entity等attributes

## RenderingResponse.java

為一個interface，implement ServerResponse。

這是特別針對render的response，此response會封裝Model的attribute，以及header等等

## DefaultRenderingResponseBuilder.java

為一個class，implement RenderingResponse中的builder class。

實際定義了如何將model透過build method包到response中，同時還會build與cookies, status相關的資訊

## HandlerFilterFunction.java
為一個interface，用來filter 'request'與'response'，可以對他們做pre-processing後，再送至對應的handler

ChatGPT上的解釋：
“The primary function of the HandlerFilterFunction is to intercept and modify the request and response objects. It provides a way to perform pre-processing or post-processing logic, such as authentication, authorization, request/response logging, or modifying the request/response headers.”

根據該解釋，這個function並非用來filter handler function，而是pre/post-processed進入handler function的request/response

底下為重要的function
- ofRequestProcessor: 處理request
- ofResponseProcessor: 處理response

















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