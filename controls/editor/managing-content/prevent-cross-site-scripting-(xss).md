---
title: Prevent Cross-site Scripting (XSS)
page_title: Prevent Cross-site Scripting (XSS) | UI for ASP.NET AJAX Documentation
description: Prevent Cross-site Scripting (XSS)
slug: editor/managing-content/prevent-cross-site-scripting-(xss)
tags: prevent,cross-site,scripting,(xss)
published: True
position: 4
---

# Prevent Cross-site Scripting (XSS)



This article examines the built-in protection of the __RadEditor__ control against __Cross Site Scripting (XSS)__ attacks and explains	how you can handle specific security needs via a custom solution.

## RadEditor and XSS

XSS is a class of attacks where malicious scripts can be injected in the web applicationand submitted to the server. If no validation or protective measures are undertaken,the injected script is executed on the page, loaded by an unsuspecting user. Subsequently,any sensitive information could be successfully hijacked to a location, known by the attacker.

These type of attacks are popular, therefore, this matter is a commonly discussedtopic in various public articles. You can learn more about how to protect your applications against XSS attacks by following these materials:

* MSDN: [How To: Prevent Cross-Site Scripting in ASP.NET](http://msdn.microsoft.com/en-us/library/ff649310.aspx)

* OWASP: [Cross-site Scripting (XSS)](https://www.owasp.org/index.php/Cross-site_Scripting_%28XSS%29)

* MDN: [Cross-site scripting](https://developer.mozilla.org/en-US/docs/Glossary/Cross-site_scripting)

__RadEditor__ provides built-in options for XSS protection of the processed content. Nevertheless,it should not be underestimated that it is still the developer’s responsibility to take care ofthe application security and validate the submitted content to prevent possible attacks as per to the guidelines of the materials provided.

The built-in __RadEditor__ features to prevent harmful content are:

* [RemoveScripts and EncodeScripts filters](#removescripts-and-encodescripts)—__RadEditor__ strips and/or encodes the script elements in order to prevent script loading/execution;

* [CSS expression stripping](#css-expressions)—__RadEditor__ sanitizes the content from possible script injections via CSS;

* [Removing attribute DOM events](#attribute-dom-events)—__RadEditor__ removes the DOM attributes that expose an option to add inline JavaScript code (e.g., onclick, onmouseover);

* [Custom content filters](#custom-content-filters)—__RadEditor__ lets you implement your own logic to strip or encode specific tags or expressions.

## RemoveScripts and EncodeScripts

The __RemoveScripts__ and __EncodeScripts__ filters are[content filters]({%slug editor/managing-content/content-filters%}) that handle <script> tags inserted in the editor’s content.

RemoveScripts filter strips all <script> elements and all JavaScript logic enclosed between its tags.This filter is designed to work on both client and server side to protect from possible execution of an already injected malicious code.For example, this content:

````HTML
			<p>paragraph</p>
			<script>
				alert("XSS")
			</script>
````



Will become this:

````HTML
			<p>paragraph</p>
````



__EncodeScripts__ enables end-users to enter a <script> element, although no code from it will be executed,because the script tags will be transformed to an HTML comment. For example, this content:

````HTML
			<p>paragraph</p>
			<script>
				alert("XSS")
			</script>
````



In Design mode will be transformed to:

````HTML
			<p>paragraph</p>
			<!--RADEDITORSAVEDTAG_script>
				alert("XSS")
			</script-->
````



And in HTML will be decoded, so that user can continue working on it.

This filter is intended only to encode and decode scripts, so JavaScript code will not be executed while edited in the __RadEditor__.Also, the submitted content will be decoded on the server (i.e., the server-side __RadEditor.Content__ property will return content with fully functional script logic).

Note that if the __RemoveScripts__ filter is enabled, the __EncodeScripts__ one will be of no value.Therefore, if you need to let users edit JavaScript in the __RadEditor__ you should disable the __RemoveScripts__ filter.For that you can use the server-side[DisableFilter()](http://www.telerik.com/help/aspnet-ajax/m_telerik_web_ui_radeditor_disablefilter.html) method.

## CSS Expressions

[CSS expressions](http://msdn.microsoft.com/en-us/library/ie/ms537634%28v=vs.85%29.aspx) were first introduced in Internet Explorer 5.5 and later deprecated with the release of Internet Explorer 8.They were designed to provide more flexible CSS stylization by incorporating JavaScript logic directly in CSS properties. This feature,again, leads to the possibility of XSS attacks by injecting malicious script in the expressions.

Since its __Q3 2014__ version, __RadEditor__ provides built-in protection against this kind of attack.With the __StripCssExpressions__ filter, CSS expressions are automatically stripped from the content. For example, the following HTML:

````HTML
			<style type="text/css">
				#container {
					width: expression(document.body.offsetWidth / 4 + 30 + "px");
				}
			</style>
			<div id="container">
				some text
			</div>
````



Will be modified by this filter to the following:

````HTML
			<style type="text/css">
				#container {
			
				}
			</style>
			<div id="container">
				some text
			</div>	
````



The __StripCssExpressions__ filter runs not only on the client, but also on the server and removes any CSS expressionsfrom the content to protect from possible execution of an already injected malicious code. Note that if this filter is disabled, both client-sideand server-side logic will not be executed, and CSS expressions will not be removed.

## Attribute DOM Events

The well-known[DOM event attributes](http://en.wikipedia.org/wiki/DOM_events)like onclick, onmouseover, etc. can also create an XSS vulnerability. Since their value is a JavaScript string, injection of scripts is possible which can be categorized as an XSS attack.

Since the __Q3 2014__ version, the __StripDomEventAttributes__ filter removes the attribute handler and all the inline code. For example, this content:

````HTML
			<p onclick="alert('XSS')">paragraph</p>
````



Will be changed to this one:

````HTML
			<p>paragraph</p>
````



__StripDomEventAttributes__, just like the __StripCssExpressions__ and __RemoveScripts__ filters,runs both on the client and on the server side to protect from possible execution of an already injected malicious code.Disabling it will prevent all such attributes from being removed.

## Custom Content Filters

Upon further security testing or according to additional custom needs, there may be the need to remove, strip or modify content based on specific requirements that are not covered out of the box.

Since the creation of the __RadEditor__ control, building a[Custom Content Filter](http://demos.telerik.com/aspnet-ajax/editor/examples/contentfilters/defaultcs.aspx) has been a well-known approach to prevent malicious code from being submitted and eventually executed on a page.

__Example 1__ demonstrates a simple encoding approach, where the script tags are encoded with the corresponding HTML entities. This prevents the script logic from being executed and the script element is rendered as plain text on the page for the reader to see.

>important The custom filters process the content only on the client. In cases where the content should be sanitized on the server, further logic should be implemented in the code behind when content is saved to the data base and/or when requested. See __Example 2__ .
>


__Example 1__: Custom content filter that shows script tags as text to the user.



````ASPNET
			<telerik:RadEditor runat="server" ID="RadEditor1" OnClientLoad="OnClientLoad">
			</telerik:RadEditor>
	
			<script type="text/javascript">
				function OnClientLoad(editor, args) {
					editor.get_filtersManager().add(new MyCustomScriptEncoder());
				}
	
				function MyCustomScriptEncoder() {
					MyCustomScriptEncoder.initializeBase(this);
					this.set_isDom(false);
					this.set_enabled(true);
					this.set_name("MyCustomScriptEncoderFilter");
					this.set_description("Encodes all <script> tags and allows the browser to render them as plain text.");
				}
	
				MyCustomScriptEncoder.prototype = {
					encodeScripts: function (contentToEncode) {
						var encodedContent = contentToEncode.replace(/<(\/*)script([^>]*)>/gi, "&lt;$1script$2&gt;");
						return encodedContent
					},
	
					getDesignContent: function (content) {
						var newContent = content;
						return this.encodeScripts(newContent);
					},
	
					// The code below is the same because it needs to be applied when switching to HTML mode and also when content is submitted. 
					getHtmlContent: function (content) {
						var newContent = content;
						return this.encodeScripts(newContent);
					}
				}
	
				MyCustomScriptEncoder.registerClass('MyCustomScriptEncoderFilter', Telerik.Web.UI.Editor.Filter);
			</script>
````
````C#
			/* These filters are disabled only for demonstration purpose.
			   Creating a custom content filter does not involve disabling any of the default filters.*/
	
			RadEditor1.DisableFilter(EditorFilters.RemoveScripts);
			RadEditor1.DisableFilter(EditorFilters.EncodeScripts);
````
````VB
			' These filters are disabled only for demonstration purpose.
			' Creating a custom content filter does not involve disabling any of the default filters.
			RadEditor1.DisableFilter(EditorFilters.RemoveScripts)
			RadEditor1.DisableFilter(EditorFilters.EncodeScripts)
	
````


__Example 2__: Custom code behind logic to remove scripts from the content before loading it in the __RadEditor__.



````C#
	using System;
	using System.Text.RegularExpressions;
	using Telerik.Web.UI;
	
	public partial class DefaultCS : System.Web.UI.Page
	{
		protected void Page_Load(object sender, EventArgs e)
		{
			// Dummy content from data base.
			string htmlContent = "some text<script>alert('XSS')</script> some text"; 
	
			RadEditor1.Content = sanitizeScripts(htmlContent);
		}
	
		private string sanitizeScripts(string htmlContent)
		{
			Regex regex = new Regex(@"<script[^>]*>[^>]*>");
	
			string sanitizedContent = regex.Replace(htmlContent, "");
	
			return sanitizedContent;
		}
	}
	
````
````VB
	Imports System.Text.RegularExpressions
	Imports Telerik.Web.UI
	
	Partial Public Class DefaultVB
		Inherits System.Web.UI.Page
		Protected Sub Page_Load(sender As Object, e As EventArgs)
			' Dummy content from data base.
			Dim htmlContent As String = "some text <script>alert('XSS')</script> some text"
	
			RadEditor1.Content = sanitizeScripts(htmlContent)
		End Sub
	
		Private Function sanitizeScripts(htmlContent As String) As String
			Dim regex As New Regex("<script[^>]*>[^>]*>")
	
			Dim sanitizedContent As String = regex.Replace(htmlContent, "")
	
			Return sanitizedContent
		End Function
	End Class
````


# See Also

 * [Content Filters]({%slug editor/managing-content/content-filters%})

 * [Securing RadEditor Content and Preventing XSS Attacks](http://blogs.telerik.com/blogs/14-09-24/securing-radeditor-content-and-preventing-xss-attacks)

 * [Demo: Built-in Content Filters](http://demos.telerik.com/aspnet-ajax/editor/examples/builtincontentfilters/defaultcs.aspx)

 * [Demo: Custom Content Filters](http://demos.telerik.com/aspnet-ajax/editor/examples/contentfilters/defaultcs.aspx)