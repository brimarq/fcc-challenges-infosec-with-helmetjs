# fCC Challenges: Information Security with HelmetJS  

https://learn.freecodecamp.org/information-security-and-quality-assurance/information-security-with-helmetjs

## NOTE:  
This set of challenges uses two *different* starter/boilerplate repos. Starting with challenge #12, the second one is expected, even though NO NOTICE is given in the instructions to that fact. So, this is my attempt to keep both in this repo. Challenges #1-11 use the `helmet-challenges` branch, while the remainder use the `bcrypt-challenges` branch.  

## CHALLENGES:

Helmet docs are [here](https://helmetjs.github.io/docs/).
### 1. Install and require Helmet.  
[Helmet](https://github.com/helmetjs/helmet) helps you secure your Express apps by setting various HTTP headers. Install the package, then require it.  

```bash
npm install helmet
```
```js
const helmet = require('helmet');
```
### 2. Hide Potentially Dangerous Information Using `helmet.hidePoweredBy()`.  
Hackers can exploit known vulnerabilities in Express/Node if they see that your site is powered by Express. `X-Powered-By: Express` is sent in every request coming from Express by default. The `hidePoweredBy` middleware will remove the `X-Powered-By` header. You can also explicitly set the header to something else, to throw people off. e.g. `helmet.hidePoweredBy({ setTo: 'PHP 4.2.0' })`.  
```js
app.use(helmet.hidePoweredBy({ setTo: 'PHP 4.2.0' }));
```
### 3. Mitigate the Risk of Clickjacking with `helmet.frameguard()`.  
Your page could be put in a `<frame>` or `<iframe>` without your consent. This can result in [clickjacking attacks](https://en.wikipedia.org/wiki/Clickjacking), among other things. Clickjacking is a technique of tricking a user into interacting with a page different from what the user thinks it is. Often this happens using another page put over the framed original, in a transparent layer. The `X-Frame-Options` header set by this middleware restricts who can put your site in a frame. It has three modes: DENY, SAMEORIGIN, and ALLOW-FROM. We don't need our app to be framed, so you should use `helmet.frameguard()` passing to it the configuration object `{action: 'deny'}`.  
```js
app.use(helmet.frameguard({action: 'deny'}));
```
### 4. Mitigate the Risk of Cross Site Scripting (XSS) Attacks with helmet.xssFilter().  
Cross-site scripting (XSS) is a very frequent type of attack where malicious script are injected into vulnerable pages, on the purpous of stealing sensitive data like session cookies, or passwords. The basic rule to lower the risk of an XSS attack is simple: **"Never trust user's input"**. As a developer, you should always *sanitize* all input coming from the outside. This includes data coming from forms, GET query urls, and even from POST bodies. Sanitizing means that you should find and encode the characters that may be dangerous e.g. `<`, `>`. More info [here](https://www.owasp.org/index.php/XSS_(Cross_Site_Scripting)_Prevention_Cheat_Sheet).  

Modern browsers can help mitigate XSS risk by adoptiong better software strategies. Often these are configurable via http headers. The `X-XSS-Protection` HTTP header is a basic protection. The browser detects a potential injected script using an heuristic filter. If the header is enabled, the browser changes the script code, neutralizing it. It still has limited support.  

Use `helmet.xssFilter()`.  
```js
app.use(helmet.xssFilter());
```
### 5. Avoid Inferring the Response MIME Type with `helmet.noSniff()`.  
Browsers can use content or MIME sniffing to adapt to different datatypes coming from a response. They override the `Content-Type` headers to guess and process the data. While this can be convenient in some scenarios, it can also lead to some dangerous attacks. This middleware sets the `X-Content-Type-Options` header to nosniff. This instructs the browser to not bypass the provided Content-Type. Use `helmet.noSniff()`.  
```js
app.use(helmet.noSniff());
```
### 6. Prevent IE from Opening Untrusted HTML with `helmet.ieNoOpen()`.  
Some web applications will serve untrusted HTML for download. Some versions of Internet Explorer by default open those HTML files in the context of your site. This means that an untrusted HTML page could start doing bad things in the context of your pages. This middleware sets the `X-Download-Options` header to `noopen`. This will prevent IE users from executing downloads in the trusted site’s context.  
```js
app.use(helmet.ieNoOpen());
```
### 7. Ask Browsers to Access Your Site via HTTPS Only with `helmet.hsts()`.  
HTTP Strict Transport Security (HSTS) is a web security policy which helps to protect websites against protocol downgrade attacks and cookie hijacking. If your website can be accessed via HTTPS you can ask user’s browsers to avoid using insecure HTTP. By setting the header `Strict-Transport-Security`, you tell the browsers to use HTTPS for the future requests in a specified amount of time. This will work for the requests coming after the initial request. Configure `helmet.hsts()` to use HTTPS for the next 90 days. Pass the config object `{maxAge: timeInMilliseconds, force: true}`. Glitch already has hsts enabled. To override its settings you need to set the field `"force"` to `true` in the config object. We will intercept and restore the Glitch header, after inspecting it for testing.  

NOTE: Configuring HTTPS on a custom website requires the acquisition of a domain and an SSL/TSL Certificate.  
```js
const ninetyDaysInMilliseconds = 90*24*60*60*1000;
app.use(helmet.hsts({maxAge: ninetyDaysInMilliseconds, force: true}));
```

### 8. Disable DNS Prefetching with `helmet.dnsPrefetchControl()`.  
To improve performance, most browsers prefetch DNS records for the links in a page. In that way the destination ip is already known when the user clicks on a link. This may lead to over-use of the DNS service (if you own a big website, visited by millions people…), privacy issues (one eavesdropper could infer that you are on a certain page), or page statistics alteration (some links may appear visited even if they are not). If you have high security needs you can disable DNS prefetching, at the cost of a performance penalty.  
```js
app.use(helmet.dnsPrefetchControl());
```

### 9. Disable Client-Side Caching with `helmet.noCache()`.  
If you are releasing an update for your website, and you want the users to always download the newer version, you can (try to) disable caching on client’s browser. It can be useful in development too. Caching has performance benefits, which you will lose, so only use this option when there is a real need.  
```js
app.use(helmet.noCache());
```

### 10. Set a Content Security Policy with `helmet.contentSecurityPolicy()`.  
This challenge highlights one promising new defense that can significantly reduce the risk and impact of many type of attacks in modern browsers. By setting and configuring a Content Security Policy you can prevent the injection of anything unintended into your page. This will protect your app from XSS vulnerabilities, undesired tracking, malicious frames, and much more. CSP works by defining a whitelist of content sources which are trusted. You can configure them for each kind of resource a web page may need (scripts, stylesheets, fonts, frames, media, and so on…). There are multiple directives available, so a website owner can have a granular control. See [HTML 5 Rocks](http://www.html5rocks.com/en/tutorials/security/content-security-policy/) and [KeyCDN](https://www.keycdn.com/support/content-security-policy/) for more details. Unfortunately CSP is unsupported by older browsers.  

By default, directives are wide open, so it’s important to set the `defaultSrc` directive as a fallback. Helmet supports both `defaultSrc` and `default-src` naming styles. The fallback applies for most of the unspecified directives. In this exercise, use `helmet.contentSecurityPolicy()`, with the `defaultSrc` directive configured to `["'self'"]` (the list of allowed sources must be in an array), in order to trust only your website address by default. Set also the `scriptSrc` directive so that you will allow scripts to be downloaded from your website, and from the domain `'trusted-cdn.com'`.  

HINT: in the `"'self'"` keyword, the single quotes are part of the keyword itself, so it needs to be enclosed in double quotes to be working.  
```js
app.use(helmet.contentSecurityPolicy({
  directives: {
    defaultSrc: ["'self'"],
    scriptSrc: ["'self'", 'trusted-cdn.com']
  }
}));
```

### 11. Configure Helmet Using the ‘parent’ `helmet()` Middleware.  
`app.use(helmet())` will automatically include all the middleware introduced above, except `noCache()`, and `contentSecurityPolicy()`, but these can be enabled if necessary. You can also disable or configure any other middleware individually, using a configuration object.  

Example:  

```js
app.use(helmet({
  frameguard: {              // configure
    action: 'deny'
  },
  contentSecurityPolicy: {   // enable and configure
   directives: {
     defaultSrc: ["'self'"],
     styleSrc: ['style.com'],
   }
  },
 dnsPrefetchControl: false   // disable
}))
```

We introduced each middleware separately for teaching purpose, and for ease of testing. Using the ‘parent’ `helmet()` middleware is easiest, and cleaner, for a real project.  

### 12. Understand BCrypt Hashes.  
BCrypt hashes are very secure. A hash is basically a fingerprint of the original data- always unique. This is accomplished by feeding the original data into a algorithm and having returned a fixed length result. To further complicate this process and make it more secure, you can also salt your hash. Salting your hash involves adding random data to the original data before the hashing process which makes it even harder to crack the hash.  

BCrypt hashes will always look like this: `$2a$13$ZyprE5MRw2Q3WpNOGZWGbeG7ADUre1Q8QO.uUUtcbqloU0yvzavOm`, which does have a structure. The first small bit of data `$2a` is defining what kind of hash algorithm was used. The next portion `$13` defines the cost. Cost is about how much power it takes to compute the hash. It is on a logarithmic scale of 2^cost and determines how many times the data is put through the hashing algorithm. For example, at a cost of 10 you are able to hash 10 passwords a second on an average computer, however at a cost of 15 it takes 3 seconds per hash... and to take it further, at a cost of 31 it would takes multiple days to complete a hash. A cost of 12 is considered very secure at this time. The last portion of your hash `$ZyprE5MRw2Q3WpNOGZWGbeG7ADUre1Q8QO.uUUtcbqloU0yvzavOm`, looks like 1 large string of numbers, periods, and letters but it is actually 2 separate pieces of information. The first 22 characters is the salt in plain text, and the rest is the hashed password!  

To begin using BCrypt, add it as a dependency in your project and require it as 'bcrypt' in your server.  

Submit your page when you think you've got it right.  

```bash
npm install bcrypt
```
```js
const bcrypt = require('bcrypt');
```

### 13. Hash and Compare Passwords Asynchronously.  
As hashing is designed to be computationally intensive, it is recommended to do so asyncronously on your server to avoid blocking incoming connections while you hash. All you have to do to hash a password asynchronously is call:  
```js
bcrypt.hash(myPlaintextPassword, saltRounds, (err, hash) => { /*Store hash in your db*/ });
``` 

Add this hashing function to your server (we've already defined the variables used in the function for you to use) and log it to the console for you to see! At this point you would normally save the hash to your database.  

Now when you need to figure out if a new input is the same data as the hash you would just use the compare function `bcrypt.compare(myPlaintextPassword, hash, (err, res) => { /*res == true or false*/ });`. Add this into your existing hash function (since you need to wait for the hash to complete before calling the compare function) after you log the completed hash and log `res` to the console within the compare. You should see in the console a hash, then `true` is printed! If you change `myPlaintextPassword` in the compare function to `someOtherPlaintextPassword` then it should say `false`. 

```js
bcrypt.hash('passw0rd!', 13, (err, hash) => {
  console.log(hash); //$2a$12$Y.PHPE15wR25qrrtgGkiYe2sXo98cjuMCG1YwSI5rJW1DSJp0gEYS
  bcrypt.compare('passw0rd!', hash, (err, res) => {
      console.log(res); //true
  });
});
```
Submit your page when you think you've got it right.  

```js
bcrypt.hash(myPlaintextPassword, saltRounds, (err, hash) => {
  if (err) return console.log(err);
  console.log(hash); 
  bcrypt.compare(myPlaintextPassword, hash, (err, res) => {
    if (err) return console.log(err);
    console.log(res);
  });
});
```

### 14. Hash and Compare Passwords Synchronously.  
Hashing synchronously is just as easy to do but can cause lag if using it server side with a high cost or with hashing done very often. Hashing with this method is as easy as calling:  
```js
var hash = bcrypt.hashSync(myPlaintextPassword, saltRounds);
```
Add this method of hashing to your code and then log the result to the console. Again, the variables used are already defined in the server so you wont need to adjust them. You may notice even though you are hashing the same password as in the async function, the result in the console is different- this is due to the salt being randomly generated each time as seen by the first 22 characters in the third string of the hash.  

Now to compare a password input with the new sync hash, you would use the compareSync method:  
```js
var result = bcrypt.compareSync(myPlaintextPassword, hash);
```
with the result being a boolean `true` or `false`. Add this function in and log to the console the result to see it working.  
```js
let hash = bcrypt.hashSync(myPlaintextPassword, saltRounds);
console.log(hash);
let result = bcrypt.compareSync(myPlaintextPassword, hash);
console.log(result); 
```
