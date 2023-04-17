## AbstractThemeResolver.java
為一個abstract，extend ThemeResolver.java(at remain files)，主要多了設定default theme name的功能

## CookieThemeResolver.java
為一個class，implement ThemeResolver。將ThemeName添加到Cookie當中(setThemeName)，或者從cookies拿出theme name(resolveThemeName)。

## FixedThemeResolver.java
為一個class，extend from AbstractThemeResolver，使用"theme"作為themeName，並且不能set新的theme name

## SessionThemeResolver.java
為一個class，extend from AbstractThemeResolver，透過Session（Client與Server的對話）中的attribute來設定Theme以及resolve theme

## ThemeChangeInterceptor.java
為一個class，implement HandlerInterceptor，並且實作preHandle function，其作用為可以在得到新的request後，對當前的theme做改變