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
    2. AbstractFormTag.java (38)
    3. FormTag.java (303)
    4. AbstractDataBoundFormElementTag.java (110)
    5. AbstractHtmlElementTag.java (238)
    6. AbstractHtmlElementBodyTag.java (75)
    7. ButtonTag.java (66)
    8. AbstractCheckedElementTag.java (39)
    9. AbstractSingleCheckedElementTag.java (45)
    10. AbstractMultiCheckedElementTag.java (167)
    11. AbstractHtmlInputElementTag.java (76) 
    12. TagWriter.java (139) 
    13. CheckboxesTag.java (23)
    14. CheckboxTag.java (45) 
    15. ErrorsTag.java (91) (Done)
    15. ValueFormatter.java
    16. SelectedValueComparator.java
    17. TagIdGenerator.java

## This Folder
Spring provides a set of custom JSP tags that can be used to access Spring-specific features from 
within a JSP page.

## JSP

| SKIP_BODY        | EVAL_BODY_INCLUDE | EVAL_BODY_BUFFERED | SKIP_PAGE | EVAL_PAGE |
| ------------- | ------------- | ----- | ----- | -----|
| The body of the tag should be skipped and not evaluated.| The body of the tag should be evaluated.| The body of the tag should be buffered and evaluated.|The remainder of the JSP page should be skipped and not evaluated.|The remainder of the JSP page should be evaluated.|


## RequestContextAwareTag
This tag is an abstract class which provides access to the current RequestContext object, which 
represents the current request being processed by the application. The RequestContext object contains 
information about the request, such as the request parameters, the request headers, and the session 
attributes. The RequestContext object is also used by Spring to resolve message codes for 
internationalization and localization purposes.

## XSS attack

## HtmlEscapingAwareTag
HtmlEscapingAwareTag is another interface in the Spring Framework's webmvc package that allows tags to
be aware of HTML escaping. It extends the BodyTag interface and adds a single method called 
setHtmlEscape, which takes a boolean value that indicates whether HTML escaping should be enabled or 
disabled for the tag.

HTML escaping is an important feature of web applications that helps prevent cross-site scripting (XSS) 
attacks. When HTML is displayed on a web page, any characters that have special meaning in HTML (such 
as < and >) must be properly encoded to prevent them from being interpreted as HTML tags. This is 
typically done using the HTML entity encoding syntax, where special characters are replaced with their 
corresponding entity references (such as < for < and > for >).

The setHtmlEscape method allows a tag to control whether its output should be HTML-escaped or not. If 
enabled, any output produced by the tag will be automatically HTML-escaped by the Spring Framework 
before being written to the response. If disabled, the output will be written as-is, without any HTML 
escaping.
## HtmlEscapeTag
    
`<spring:htmlEscape>`
Change the default html escape setting in the body.
    
## EscapeBodyTag
    
`<spring:escapeBody>` 

Is used to escape HTML characters in the body of an HTML document. This tag is 
typically used to ensure that user-supplied content, such as form input or comments, is properly 
sanitized and does not contain any malicious scripts or code that could harm the system or other users.

## EvalTag
`<c:set>`
`<c:out>`
It allows you to evaluate an expression and store its result in a variable. The tag evaluates the 
expression at runtime and the result can be used in subsequent process.

## ArgumentAware

## ArgumentTag
`<spring:argument>`

## ParamAware

## ParamTag
`<spring:param>`

## Param

## MessageTag
`<spring:message>` is a custom tag provided by the Spring Framework that allows you to easily access and display messages from message resource bundles. Message resource bundles are typically used to store application messages and other text strings that need to be internationalized or localized.

Here's a brief overview of how `<spring:message>` works:

*   The tag takes a `code` attribute that specifies the message code to look up in the message resource bundle.
*   The tag can also take additional attributes to customize the behavior of the tag, such as a `var` attribute to store the message in a variable, a `text` attribute to specify a default message if the code is not found, and so on.
*   The tag evaluates the message code using the `MessageSource` configured in the Spring application context.
*   The `MessageSource` looks up the message code in the configured message resource bundles (typically properties files), using the current `Locale` to select the appropriate bundle.
*   If the message is found, the tag displays it; otherwise, it falls back to the default message specified by the text attribute, or an empty string if no default message is specified.

## ThemeTag
`<spring:theme>` is a custom tag provided by the Spring Framework that allows you to easily apply theme-specific styles to your web pages. A theme is a collection of related resources such as CSS stylesheets, images, and other resources that define the look and feel of your web application.

Here's a brief overview of how `<spring:theme>` works:

*   The tag takes a `code` attribute that specifies the name of the theme to apply. Themes are typically defined in configuration files, such as XML or properties files.
*   The tag can also take additional attributes to specify the resource type (type), resource name (name), and other parameters needed to retrieve the appropriate theme resources.
*   The tag generates HTML code that includes links to the specified theme resources, such as CSS stylesheets and images.
*   The generated HTML code can be included in your JSP pages to apply the specified theme to the page.

## TransformTag
`<spring:transform>` is a custom tag provided by the Spring Framework that allows you to transform the output of other tags using a specified transformation script or expression language. This can be useful for customizing the output of other tags, such as formatting dates or numbers, or converting HTML markup to another format.
*   The tag takes a `var` attribute that specifies a variable name to store the transformed output.
*   The tag also takes a `script` or `expression` attribute that specifies the transformation to apply to the output.
*   The tag delegates the rendering of its body content to another tag or JSP element, such as `<c:out>` or `<spring:message>`.
*   After the body content is rendered, the tag applies the specified transformation script or expression to the output.
*   The transformed output is stored in the specified variable for further processing or display.

Using `<spring:transform>` can provide a powerful way to customize the output of your JSP pages by applying arbitrary transformations to the output of other tags. The tag supports a wide range of transformation options, including script-based transformations using various scripting languages, as well as expression-based transformations using SpEL (Spring Expression Language).

## UrlTag
`<spring:url>` is a custom tag provided by the Spring Framework that allows you to generate URLs for various resources in your web application, such as controller actions, static resources, or other web pages. The tag provides a consistent way to generate URLs that are compatible with Spring's URL mapping and handling mechanisms.

Here's a brief overview of how `<spring:url>` works:

*   The tag takes a `value` attribute that specifies the target URL, using a Spring-specific syntax that includes placeholders for dynamic parts of the URL.
*   The tag can also take additional attributes to customize the behavior of the tag, such as a `var` attribute to store the generated URL in a variable, or attributes to specify query parameters or fragment identifiers.
*   The tag generates a URL based on the configured URL path prefix and suffix, and the target URL specified in the `value` attribute.
*   The generated URL can be used as a hyperlink or other link within your JSP page, or can be stored in a variable for further processing or display.

## BindErrorsTag
`<spring:hasBindErrors>` is a custom tag provided by the Spring Framework that allows you to check if there are any binding or validation errors associated with a specific form field. When using form binding and validation in Spring, you typically use the `<form:form>` and `<form:errors>` tags to display validation errors that occur during form submission. `<spring:hasBindErrors>` provides an alternative way to check for binding and validation errors, with some additional flexibility and control.

## BindTag
`<spring:bind>` is a custom tag provided by the Spring Framework that allows you to bind form fields to model attributes in your JSP pages. When using form binding in Spring, you typically use the `<form:form>` tag to generate an HTML form that includes form fields, and then use the `<form:input>` or `<form:select>` tags to generate form fields that are bound to specific model attributes. `<spring:bind>` provides an alternative way to bind form fields to model attributes, with some additional features and flexibility.

## NestedPathTag
`<spring:nestedPath>` is a custom tag provided by the Spring Framework that allows you to specify a nested path for form binding in your JSP pages. When using form binding in Spring, you may need to bind form fields to properties of nested objects, and `<spring:nestedPath>` provides a way to specify the nested path in a concise and consistent way.

Here's a brief overview of how `<spring:nestedPath>` works:

*   The tag takes a `path` attribute that specifies the nested path to be used for form binding.
*   The tag sets the specified nested path on the `HttpServletRequest` object, so that any form binding that occurs within the tag's body content will use the specified nested path.
*   The tag also restores the original nested path after the tag's body content is rendered, so that subsequent form binding will use the original nested path.
