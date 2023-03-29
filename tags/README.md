# tags folder

## Reading order
1. package-info.java
2. RequestContextAwareTag.java
3. HtmlEscapingAwareTag.java
4. HtmlEscapeTag.java
5. EscapeBodyTag.java
6. EvalTag.java
7. ArgumentAware.java
8. ArgumentTag.java
9. ParamAware.java
10. ParamTag.java
11. Param.java
12. MessageTag.java
13. ThemeTag.java
14. TransformTag.java
15. UrlTag.java
16. BindTag.java
17. EditorAwareTag.java
18. NestedPathTag.java
19. form
    1. package-info.java
    2. AbstractFormTag.java
    3. FormTag.java
    4. AbstractHtmlElementTag.java
    5. AbstractHtmlInputElementTag.java
    6. TagWriter.java
    7. ValueFormatter.java
    8. SelectedValueComparator.java
    9. TagIdGenerator.java

## This Folder
    Spring provides a set of custom JSP tags that can be used to access Spring-specific features from within a JSP page.

## RequestContextAwareTag
    This tag provides access to the current RequestContext object, which represents the current request being processed by the application. The RequestContext object contains information about the request, such as the request parameters, the request headers, and the session attributes. The RequestContext object is also used by Spring to resolve message codes for internationalization and localization purposes.

## HtmlEscapingAwareTag
    HtmlEscapingAwareTag is another interface in the Spring Framework's webmvc package that allows tags to be aware of HTML escaping. It extends the BodyTag interface and adds a single method called setHtmlEscape, which takes a boolean value that indicates whether HTML escaping should be enabled or disabled for the tag.

    HTML escaping is an important feature of web applications that helps prevent cross-site scripting (XSS) attacks. When HTML is displayed on a web page, any characters that have special meaning in HTML (such as < and >) must be properly encoded to prevent them from being interpreted as HTML tags. This is typically done using the HTML entity encoding syntax, where special characters are replaced with their corresponding entity references (such as < for < and > for >).

    The setHtmlEscape method allows a tag to control whether its output should be HTML-escaped or not. If enabled, any output produced by the tag will be automatically HTML-escaped by the Spring Framework before being written to the response. If disabled, the output will be written as-is, without any HTML escaping.

