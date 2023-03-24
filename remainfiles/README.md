## ModelAndView.java
ModelAndView提供一個容器，可以同時包含Model以及View，且這個檔案可由View or Model去Construct，並且還提供修改Model的method等等。之所以要把Model與View綁在通一個Class是因為controller可以同時return Model與View

## ModelAndViewDefiningException.java
提供一個class，使用ModelAndView object進行init，用來檢驗該ModelAndView object是否為Null

## View.java
提供View的interface，最重要的為其中的render function，用來定義如何與Model配合並且render當前畫面

## ViewResolver.java
解析當前View的name屬性，當controller回傳多個view時，就是靠resolver去辨別當前的view所對應的名稱

ref:https://www.cntofu.com/book/84/spring/spring-view-resolver.md