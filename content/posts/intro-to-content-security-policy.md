---
title: "Intro to Content Security Policy"
date: 2021-07-22T19:00:42+05:30
draft: false
toc: false
images:
tags:
  - web-security
---
Understanding certain web security concepts is crucial for comprehending how browsers, servers, and web applications function in specific scenarios. In this blog post, I'll provide a brief introduction to the Content Security Policy (CSP) mechanism.

{{< image src="/csp-meme.jpeg" alt="Centered Image" size="450x450" align="center" >}}

While inspecting the Request/Response cycle, you may have come across this HTTP Header in the response:

`Content-Security-Policy: default-src 'self'; script-src static.site.com`

Alternatively, during web app development, you might have encountered this error in the browser console:

`Refused to load the scripts because it violates the following content security policy directive...`

These experiences might have left you with some questions:

1. What is CSP?
2. Why is it necessary?
3. How does it work?

---

# Understanding CSP
CSP, which stands for Content Security Policy, is a mechanism that allows developers to whitelist the locations from which an application can load resources. These resources can include scripts, styles, images, media, and more. The primary objective behind implementing CSP is to mitigate content injection vulnerabilities, with cross-site scripting (XSS) being one of the most common and damaging types of such vulnerabilities. CSP is implemented by specifying the Content-Security-Policy Header followed by directives and schemes.

---

# Why CSP is Essential?
CSP is essential because, without it, malicious content, such as JavaScript code, could be injected into your website. When rendered, this can lead to XSS vulnerabilities. CSP effectively prevents these malicious codes from being executed within the web application.

---

# How CSP Works?
CSP accomplishes this by defining trusted locations in the directives from which the application can load resources. Before loading a web page, the browser checks the Content-Security-Policy Header and verifies if the requested resources originate from the allowed sources. If the resources are fetched from whitelisted locations specified in the CSP Header, the browser allows them; otherwise, it blocks the resources.

For example, consider this CSP header:

`Content-Security-Policy: default-src 'self'`

This policy contains a single directive, `default-src`, which permits resources to be loaded only from their own origin ('self').

The `self` keyword signifies "Same Protocol and Same Host", So for ex:

- `https://github.com/johndoe` & `https://github.com/test/johndoe`

are the same because they both have the same protocol and same host.

- `https://github.com/bob` & `http://github.com/bob`

are not the same because they have a different protocol.

`https://test.github.com/alice` & `https://github.com/alice`

are not the same because they have a different host.

It's important to note that the `default-src` directive serves as a fallback for other directives. So, even if you haven't explicitly set other directives, they will automatically inherit the value of the `default-src` directive. So,

`Content-Security-Policy: default-src 'self'; script-src script.site.com`

is same as:

```
Content-Security-Policy:
        connect-src 'self';
        font-src 'self';
        frame-src 'self';
        img-src 'self';
        manifest-src 'self';
        media-src 'self';
        prefetch-src 'self';
        object-src 'self';
        script-src 'script.site.com';
        script-src-elem 'self';
        script-src-attr 'self';
        style-src-elem 'self';
        style-src-attr 'self';
        worker-src 'self'
```

As you can see, although you didn’t specify the other fetch directives, by default they are set to the value of the default-src directive.

Did you notice the `script-src` directive? Its value is set to 'script.site.com,' not `self`, because explicitly defined directives will override the value of the `default-src` directive.

---

# Examples of CSP Policies

Here are some examples of CSP policies to illustrate the granularity of control CSP offers:

`Content-Security-Policy: default-src '*://*.trusted.com'`

This policy allows sources from any scheme (http/https) and any subdomain of trusted.com but not 'trusted.com' itself.

`Content-Security-Policy: script-src 'self' api.trusted.com`

This CSP policy permits loading scripts from its own origin and api.trusted.com. It does not allow executing inline-scripts, such as <script> tags, <style> tags, onclick events, etc., as CSP considers them unsafe and disables them by default. You can specify 'unsafe-inline' to enable them, but it's not recommended.

`Content-Security-Policy: default-src 'none'`

This policy blocks all resource loading because the source value is set to 'none.'

CSP allows for highly customized settings tailored to your application's needs. For instance, consider this policy:

- `default-src 'none'`: Blocks everything.
- `script-src '*.trusted.com'`: Allows scripts from any subdomain of trusted.com.
- `image-src 'image.trusted.com'`: Permits images from image.trusted.com.
- `font-src 'https://fonts.googleapis.com/' '*://fonts.trusted.com'`: Allows fonts from https://fonts.googleapis.com and fonts.trusted.com, using any scheme (http/https).
- `frame-src 'child.trusted.com'`: Allows frames to be loaded from child.trusted.com.
- `report-uri https://report.trusted.com`: The report-uri directive instructs the browser to send a POST request to the specified endpoint containing attempted CSP violations in JSON format.

You can also use `Content-Security-Policy-Report-Only` as the HTTP header name to instruct the browser to only send reports when CSP violations occur. This header doesn't block anything. It's a good practice to use this header when starting out to build up CSP rules based on violation reports.

{{< image src="/one-does-not.jpg" alt="Centered Image" size="450x450" align="center" >}}

---

# Conclusion

This overview provides a glimpse into Content Security Policy. Of course, there's much more to explore, but delving into every detail would make this blog quite extensive.

Special thanks to [Adit Chanchal](https://www.linkedin.com/in/adit-chanchal-3344b2145/) for reviewing this blog.





