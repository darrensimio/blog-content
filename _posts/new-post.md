---
post_title: 'Security Vulnerability and Browser Performance Impact of target="_blank"'
layout: post
published: true
---

The anchor tag `[<a>](https://www.w3.org/MarkUp/1995-archive/Elements/A.html)` has been around in the [HTML standards](https://www.w3.org/MarkUp/1995-archive/Elements/A.html) for over 20 years, and has been the defecto standard for implementing in-page navigations, as well as navigations beyond the page. This has changed in recent years, with the popularity of javascript and single-page applications (SPAs); where page navigations are handled within onClick events of page elements instead.

Nonetheless, many of the concepts remain the same, and are handled by the browsers in almost the same way. The following code will give you the same behavior on [evergreen browsers](https://www.techopedia.com/definition/31094/evergreen-browser).

**HTML**`<a href="http://www.google.com" target="_blank">open me</a>`**JavaScript**`window.open("http://www.google.com", "_blank");`

### Opening Links in New Browser Tabs

Navigation is one of the most important aspects of any modern web page or web application. Links can be largely categorized into two categories: **part of the user flow**, and **not part of the user flow**.

Links not considered as part of the user flow are generally considered a distraction; we often apply the `target="_blank"` treatments to these links to open them in a separate browser window or browser tab. The `target="_blank"` treatment is also the defecto standard for any outbound links to an external website.

While many would expect that opening an external link in a separate browser window / tab disassociates the external link from the host page, this is **not the case**.

When a browser window (or tab) is opened from another window using the JavaScript window.open function, or from a link with its target attribute set (e.g. `<a href="site.html" target="_blank">`), it maintains a reference to that first window via the `window.opener` property in the page model.

### The Vulnerability

With a reference to the page that opens it, the _**newly opened page**_ (in a new window / tab) now has full control over the _**opening page**_'s window object. What this means is the _**newly opened page**_ can go as far as to change the location of the _**opening page**_.

While this might look relatively harmless, consider a scenario of a very popular online forum, where the newly opened page redirects users to a phishing page, asking for login credentials or other sensitive information.

![image-25](http://darrensim.io/wp-content/uploads/2017/11/target-blank-vulnerability-example-flow-1024x300.png)

I've created a [_**sample scenario**_](https://s3-ap-southeast-1.amazonaws.com/darrensimio-files/target-blank-vulnerability-example/homepage.html) available accessible via [https://s3-ap-southeast-1.amazonaws.com/darrensimio-files/target-blank-vulnerability-example/homepage.html](https://s3-ap-southeast-1.amazonaws.com/darrensimio-files/target-blank-vulnerability-example/homepage.html).

Following is the malicious code that will navigate the _**opening page**_ to a phishing page.

**The Malicious JavaScript Code**
[crayon-5ab92cc094b40192555268/]

### Resolving the Vulnerability

Fortunately, resolving this vulnerability is pretty straight forward.

In order to restrict access to the opening page from the newly opened page, all we need to do is to add the `rel="noopener"` attribute to any link that has `target="_blank"` on the opening page; this will work with the latest releases of Firefox, Chrome, Safari, and Opera. ([Click here](https://caniuse.com/#feat=rel-noopener) for list of browser supporting `rel=noopener`)

Alternatively, for older browsers, `rel="noreferrer"` which also disables the Referrer HTTP header, can be used.

Following are the two implementations; one in html and the other in JavaScript.

**HTML**
[crayon-5ab92cc094b4e184317379/]
**JavaScript**
[crayon-5ab92cc094b56089495190/]

### Performance Impacts

While most modern browsers supports multi-process and multi-threading, parsing, style calculation, layout, painting and non-worker JavaScript happens in the same thread commonly known as the "main" thread. Each browser tab runs on its own "main" thread, preventing a resource hungry site running on a tab from bringing down the entire browser with it.

However, given that cross-window access provided via `window.opener` is synchronous, windows launched through `target="_blank"` end up in the same process & thread. This applies to iframes and windows opened via `window.open` as well.

![image-24](http://darrensim.io/wp-content/uploads/2017/11/target-blank-page-freeze-example-flow-1024x300.png)

What this means is if the **newly opened page** is a resource hungry and slow performing site, it will result in performance degradation of the **opening page** (rendering the **opening page** inaccessible), and vice-versa. The introduction of new "[throttling policy](https://developers.google.com/web/updates/2017/03/background_tabs)" since [Chrome 57](https://developers.google.com/web/updates/2017/03/background_tabs) can add interesting complexities to this.

In the [sample scenario](https://s3-ap-southeast-1.amazonaws.com/darrensimio-files/target-blank-vulnerability-example/homepage.html) accessible via [https://s3-ap-southeast-1.amazonaws.com/darrensimio-files/target-blank-vulnerability-example/homepage.html](https://s3-ap-southeast-1.amazonaws.com/darrensimio-files/target-blank-vulnerability-example/homepage.html), we will observe that when we navigate to a low performing page, it causes the **opening page** to be unresponsive. Clicking, scrolling, or selecting text becomes no longer possible on the **opening page**.

This is all that the slow performing page is doing; running a while loop for 2 minutes.
[crayon-5ab92cc094b5b416152246/]

### Resolving Performance Impacts

Like the above example from _**resolving the vulnerability**_, `rel="noopener"` prevents `window.opener`, so there's no cross-window access. Like the above example from resolving the vulnerability, `rel="noopener"` prevents `window.opener`, so there's no cross-window access.

Unfortunately the JavaScript example did not work in my experiments with Chrome 61; it seems like chrome only disassociates the thread when `rel="noopener"` is implemented.

What this means is that we will now need to use the window features options in the JavaScript `window.open()` function instead. While it worked, this was not an ideal implementation for me as using window features results in the **newly opened page** to be opened in a new browser window rather than a new tab.

(**Note:** Chrome will open the new window in a new tab if the window is set to full screen on the Mac; but will open in a new window if it is not in full screen mode.)

Following are the two implementations; one in html and the other in JavaScript.

**HTML**
[crayon-5ab92cc094b5e484295992/]
**JavaScript**
[crayon-5ab92cc094b65241329680/]

### Cross Browser Support

![image-23](http://darrensim.io/wp-content/uploads/2017/11/can-i-use-no-opener-1024x367.png)

The screen capture above (extracted from [caniuse.com](https://caniuse.com/#search=noopener)), offers a nice summary of browsers that supports the `noopener` attribute. While this attribute is supported by most major evergreen browsers, it is intriguing to learn that this is not yet supported in the latest versions of Microsoft Edge.

The good news is this is not a call for concern at this point, as cross-window communication is pretty broken in Microsoft Edge. When Microsoft Edge moved to Universal Windows Platform (UWP), many cross-window communication that used to work in Win32 broke due to marshaling issues. This resulted in browser windows / tabs being forcibly isolated from each other, hence, we will not suffer from the shared main thread issue in this case.

When browsers eventually adds support for the [CSP](https://w3c.github.io/webappsec-csp/#directive-disown-opener) `disown-opener` [directive](https://w3c.github.io/webappsec-csp/#directive-disown-opener), youll be able to just use it as a meta tag in the `head` of your HTML page, or as an HTTP header. Either way the effect is to set `window.opener` to `null` for any new windows / tabs that get created from the document which includes that CSP directive.

**Meta Tag**
[crayon-5ab92cc094b69614010653/]
**HTTP Header**
[crayon-5ab92cc094b6b222696250/]

### Additional Readings

*   Source code of the [sample scenarios](https://s3-ap-southeast-1.amazonaws.com/darrensimio-files/target-blank-vulnerability-example/homepage.html) used in this article can be found on [https://github.com/darrensimsg/target-blank-vulnerability-example](https://github.com/darrensimsg/target-blank-vulnerability-example).
*   [https://jakearchibald.com/2016/performance-benefits-of-rel-noopener/](https://jakearchibald.com/2016/performance-benefits-of-rel-noopener/)
*   [https://w3c.github.io/webappsec-csp/#directive-disown-opener](https://w3c.github.io/webappsec-csp/#directive-disown-opener)
*   [https://developer.mozilla.org/en-US/docs/Web/API/Window/open](https://developer.mozilla.org/en-US/docs/Web/API/Window/open)