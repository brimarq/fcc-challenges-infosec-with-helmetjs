# Free Code Camp - Applied InfoSec Challenges
=============================================

# fCC Challenges: Information Security with HelmetJS  

1. **Install and require Helmet**. [Helmet](https://github.com/helmetjs/helmet) helps you secure your Express apps by setting various HTTP headers. Install the package, then require it.  
2. **Hide Potentially Dangerous Information Using `helmet.hidePoweredBy()`**.  Hackers can exploit known vulnerabilities in Express/Node if they see that your site is powered by Express. `X-Powered-By: Express` is sent in every request coming from Express by default. The `hidePoweredBy` middleware will remove the `X-Powered-By` header. You can also explicitly set the header to something else, to throw people off. e.g. `helmet.hidePoweredBy({ setTo: 'PHP 4.2.0' })`.