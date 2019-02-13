# Free Code Camp - Applied InfoSec Challenges
=============================================

# fCC Challenges: Information Security with HelmetJS  

1. **Install and require Helmet**. [Helmet](https://github.com/helmetjs/helmet) helps you secure your Express apps by setting various HTTP headers. Install the package, then require it.  
2. **Hide Potentially Dangerous Information Using `helmet.hidePoweredBy()`**.  Hackers can exploit known vulnerabilities in Express/Node if they see that your site is powered by Express. `X-Powered-By: Express` is sent in every request coming from Express by default. The `hidePoweredBy` middleware will remove the `X-Powered-By` header. You can also explicitly set the header to something else, to throw people off. e.g. `helmet.hidePoweredBy({ setTo: 'PHP 4.2.0' })`.  
3. **Mitigate the Risk of Clickjacking with `helmet.frameguard()`**. Your page could be put in a `<frame>` or `<iframe>` without your consent. This can result in [clickjacking attacks](https://en.wikipedia.org/wiki/Clickjacking), among other things. Clickjacking is a technique of tricking a user into interacting with a page different from what the user thinks it is. Often this happens using another page put over the framed original, in a transparent layer. The `X-Frame-Options` header set by this middleware restricts who can put your site in a frame. It has three modes: DENY, SAMEORIGIN, and ALLOW-FROM. We don't need our app to be framed, so you should use `helmet.frameguard()` passing to it the configuration object `{action: 'deny'}`.  
4. **Mitigate the Risk of Cross Site Scripting (XSS) Attacks with helmet.xssFilter()**. Cross-site scripting (XSS) is a very frequent type of attack where malicious script are injected into vulnerable pages, on the purpous of stealing sensitive data like session cookies, or passwords. The basic rule to lower the risk of an XSS attack is simple: **"Never trust user's input"**. As a developer, you should always *sanitize* all input coming from the outside. This includes data coming from forms, GET query urls, and even from POST bodies. Sanitizing means that you should find and encode the characters that may be dangerous e.g. `<`, `>`. More info [here](https://www.owasp.org/index.php/XSS_(Cross_Site_Scripting)_Prevention_Cheat_Sheet). Modern browsers can help mitigate XSS risk by adoptiong better software strategies. Often these are configurable via http headers. The `X-XSS-Protection` HTTP header is a basic protection. The browser detects a potential injected script using an heuristic filter. If the header is enabled, the browser changes the script code, neutralizing it. It still has limited support. Use `helmet.xssFilter()`. 
5. **Avoid Inferring the Response MIME Type with `helmet.noSniff()`**. Browsers can use content or MIME sniffing to adapt to different datatypes coming from a response. They override the `Content-Type` headers to guess and process the data. While this can be convenient in some scenarios, it can also lead to some dangerous attacks. This middleware sets the `X-Content-Type-Options` header to nosniff. This instructs the browser to not bypass the provided Content-Type. Use `helmet.noSniff()`.
6. **Prevent IE from Opening Untrusted HTML with `helmet.ieNoOpen()`**. Some web applications will serve untrusted HTML for download. Some versions of Internet Explorer by default open those HTML files in the context of your site. This means that an untrusted HTML page could start doing bad things in the context of your pages. This middleware sets the `X-Download-Options` header to `noopen`. This will prevent IE users from executing downloads in the trusted site’s context. 
