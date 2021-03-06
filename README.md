luminateExtend.js
=================

Version: 1.2 (22-OCT-2012)  
Requires: jQuery v1.6.4+

luminateExtend.js is a JavaScript library for use with 
[Luminate Online](http://www.convio.com/our-products/luminate/fundraising.html), a product of Blackbaud. 
Built on top of jQuery, it provides a JavaScript wrapper around the 
[Luminate Online REST API](http://open.convio.com/api), with some helper functions and other magic 
sprinkled in. The library includes support for [all major browsers](#libBrowsers), and can be used both 
within and outside of Luminate Online.

**TL;DR:** you can add donation forms and stuff to your organization's Drupal or Wordpress (or whatever) 
website!

<a name="libToc"></a>
Table of contents
-----------------

[Basic setup](#libSetup)  
[Including the library](#includingLib)  
[The luminateExtend object](#luminateExtendObj)  
[luminateExtend.library](#libraryObj)  
[luminateExtend.init](#initObj)  
[luminateExtend.global](#globalObj)  
[luminateExtend.api](#apiObj)  
[luminateExtend.tags](#tagsObj)  
[luminateExtend.sessionVars](#sessionVarsObj)  
[luminateExtend.utils](#utilsObj)  
[Browser support](#libBrowsers)  
[Reporting issues](#libIssues)

<a name="libSetup"></a>
Basic setup
-----------

Before using luminateExtend.js, there are a few basic steps you must follow:

 * Create an API Key
   
   In order to use the API, you must define an API Key for your organization's Luminate Online website. 
   If you haven't already done so, go to Setup -> Site Options -> Open API Configuration, and click 
   "Edit API Keys". For the purposes of using this library, the only option you need to worry about on 
   this page is **1. Convio API Key**. This key is an arbitrary value which you will use when calling 
   [luminateExtend.init](#initObj).
 
 * Whitelist your domain
   
   For security reasons, the API and this library limit requests to a list of domains whitelisted by 
   your organization. If you haven't already done so, go to Setup -> Site Options -> Open API 
   Configuration, and click "Edit Javascript/Flash configuration". For the purposes of using this 
   library, the only option you need to worry about on this page is **2. Trust JavaScript/Flash API from 
   these domains**. Add any domains where you will use this library to the list. As noted on the page, 
   you can use an asterisk as a wildcard if your website has multiple subdomains, e.g. 
   "\*.myorganization.com".
 
 * Create luminateExtend_server PageBuilder page 
   
   Go to Content -> PageBuilder and create a new page named luminateExtend_server. If you wish to 
   restrict access to this page so that others in your organization cannot edit it, set the Security 
   Category to the appropriate value. Though there will be cases where this page will need to be secure, 
   ensure that the option "This is a secure Page that will be encrypted via SSL" is *not* checked. 
   The other configuration options are not relevant for this page and can be skipped. In an HTML Content 
   component, copy and paste the code found in the attached file, 
   [luminateExtend_server.html](https://github.com/noahcooper/luminateExtend/blob/master/luminateExtend_server.html).
   
   The library uses a hidden request to this PageBuilder page to handle cross-domain communication in 
   older browsers (and, in some cases, all browsers). See the description of 
   [luminateExtend.api.request](#apiObj) for more details.
 
 * Upload luminateExtend_client.html to your external website
   
   On the website where you will be using the library (e.g. your organization's Drupal or Wordpress 
   site), upload the attached file, 
   [luminateExtend_client.html](https://github.com/noahcooper/luminateExtend/blob/master/luminateExtend_client.html). 
   The file should be placed in the root of your website.
   
   The library uses a hidden request to this page to handle cross-domain communication in IE7. See the 
   description of [luminateExtend.api](#apiObj) for more details.

<a name="includingLib"></a>
Including the library
---------------------

Once you've uploaded 
[luminateExtend.js](https://github.com/noahcooper/luminateExtend/blob/master/luminateExtend.js) to your 
website, including the library on a page is easy &mdash; simply add the following script tag to the head, 
somewhere below where jQuery is included. Change out the file path as needed, depending on where you 
uploaded the file.

```  html
<script src="../js/luminateExtend.js"></script>
```

If you aren't yet including jQuery on your website, use the Google CDN.

```  html
<script src="//ajax.googleapis.com/ajax/libs/jquery/1/jquery.min.js"></script>
<script src="../js/luminateExtend.js"></script>
```

<a name="luminateExtendObj"></a>
The luminateExtend object
-------------------------

Once you've included the library on a page, the global object `luminateExtend` will be available for 
use. All of the methods and data used by the library are contained within this object.

<a name="libraryObj"></a>
luminateExtend.library
----------------------

`luminateExtend.library` contains information about the library.

```  js
console.log(luminateExtend.library.version); // logs a value like "1.0"
```

The library object contains the following:

`version`

The version number of the library loaded on the page.

<a name="initObj"></a>
luminateExtend.init
-------------------

`luminateExtend.init` is a method used to define global settings used by the library. It needs only be 
called once per page, e.g. immediately below where the library is included in the head.

```  js
$(function() {
  luminateExtend.init({
    apiKey: '123456789', 
    path: {
      nonsecure: 'http://www.myorganization.com/site/', 
      secure: 'https://secure2.convio.net/myorg/site/'
    }
  });
});
```

luminateExtend is itself an alias for the init method when called directly.

```  js
$(function() {
  luminateExtend({
    apiKey: '123456789', 
    path: {
      nonsecure: 'http://www.myorganization.com/site/', 
      secure: 'https://secure2.convio.net/myorg/site/'
    }
  });
});
```

The init method accepts an object containing the following:

`apiCommon`

An object containing the [common parameters](http://open.convio.com/api/#main.common_parameters.html) 
to use with all API requests, including **categoryId**, **centerId**, **source**, and **subSource**.

`apiKey`

The API Key for your organization's Luminate Online website. When using the library within Luminate 
Online, this can be defined dynamically using "[[S0:CONVIO_API_KEY]]".
 
`auth`

An object containing [authentication](http://open.convio.com/api/#main.auth_token.html) information for 
the current user. This object contains **token**, the authentication token string, and **type**, either 
"auth" or "sso_auth_token", depending on how the token was obtained. If you don't have an authentication 
token for the user, not a problem &mdash; [luminateExtend.api](#apiObj) will automatically obtain a token 
when it needs one.
 
`path`

An object containing the URL path to your organization's Luminate Online website. **nonsecure** is the path 
for requests made over HTTP, e.g. "http://www.myorganization.com/site/". **secure** is the path for 
requests made over HTTPS, e.g. "https://secure2.convio.net/myorg/site/".

<a name="globalObj"></a>
luminateExtend.global
---------------------

The init method stores the data passed to it in the `luminateExtend.global` object.

```  js
$('#profileContainer').append('<a href="' + luminateExtend.global.path.nonsecure + 
                              'ConsProfileUser">Edit your profile</a>');
```

To update any of the data stored in the global object after calling init, use the update method.

```  js
luminateExtend.global.update('apiKey', '123456789');
```

The update method accepts either a single name/value pair or an object.

```  js
luminateExtend.global.update({
  auth: {
    token: myAuthToken, 
    type: 'sso_auth_token'
  }
});
```

Note that the global object can be extended to store custom data as needed.

```  js
luminateExtend.global.update('myCustomFn', function() {
  alert('success!');
});

luminateExtend.global.myCustomFn(); // alerts "success!"
```

<a name="apiObj"></a>
luminateExtend.api
------------------

`luminateExtend.api` is the heart of the library. It provides methods for making AJAX requests to the 
Luminate Online REST API, with a built-in polyfill for handling the idiosyncrasies of older, less 
compliant browsers.

The api object contains the following:

`request`

With the request method, calling the API becomes as simple as writing a few lines of code.

```  js
var myLoginTestCallback = function(data) {
  console.log(data);
};

luminateExtend.api.request({
  api: 'cons', 
  callback: myLoginTestCallback, 
  data: 'method=loginTest'
});
```

The API supports the [Cross-Origin Resource Sharing (CORS)](http://www.w3.org/TR/cors/) spec. With CORS, 
modern browsers including Firefox 3.5+, Chrome 2+, Safari 4+, Opera 12+, and IE10+ allow for natively 
making cross-domain XMLHttpRequests thanks to the Access-Control-Allow-Origin HTTP response header. By 
default, the request method uses XHR in these browsers. However, as currently the API does not respond 
with the Access-Control-Allow-Credentials header needed to send cookies with cross-domain requests, for 
any request that requires cookies (i.e. any request that requires authentication), 
[window.postMessage](https://developer.mozilla.org/en-US/docs/DOM/window.postMessage) is used instead. 
For older browsers which do not support CORS, namely IE8 and IE9, window.postMessage is *always* used. 
(Note that the Microsoft-proprietary XDomainRequest is not used because of its limitations, most 
importantly, the inability to set the Content-Type request header and the same-scheme policy.) For even 
*older* browsers such as IE7 which do not support window.postMessage, this method falls back to using a 
hash change transport.

The request method accepts one argument, an options object containing the following:

**api:** The specific API servlet to call. Can be provided as either the full, case-sensitive servlet 
name, e.g. "CRConsAPI", or a case-insensitive shorthand with "CR" and "API" removed, e.g. "cons".

**callback:** The callback to be used after the request is complete. The JSON response object is passed 
as the sole argument to the callback. Can be provided as either a function, or an object containing 
distinct `success` and `error` functions. With the former, the function is called regardless of whether 
or not the API responds with an error message. The latter simplifies error-detection by pre-screening the 
response.

**contentType:** The Content-Type for the request, either "application/x-www-form-urlencoded", the 
default, or "multipart/form-data". Note that due to browser limitations, it is not currently possible to 
use the library with API methods which involve a file upload, e.g. the TeamRaiser 
[uploadPersonalPhoto](http://open.convio.com/api/#teamraiser_api.uploadPersonalPhoto_method.html) method.

**data:** The data string to be sent with the request. The api_key, response_format, 
suppress_response_codes, and v parameters are automatically appended to this string, along with any 
common parameters defined in luminateExtend.global.apiCommon.

**form:** A selector for a form to be serialized with the request. The result is appended to the data 
string above.

**requestType:** The type of HTTP request, either "GET", the default, or "POST".

**requiresAuth:** A boolean indicating whether or not the API method being called requires 
authentication. If true, an auth token is automatically appended to the request data string.

**useHTTPS:** A boolean indicating whether or not to use HTTPS when making the request. Some API servlets 
(namely CRDonationAPI and CRTeamraiserAPI) must always be called over a secure channel, in which case 
this setting is ignored. Otherwise, the default depends on the protocol of the requesting page, meaning 
false for pages served over HTTP or true for pages served over HTTPS.

Note that as of v1.1, luminateExtend.api is an alias for the request method when called directly.

```  js
var myLoginTestCallback = function(data) {
  console.log(data);
};

luminateExtend.api({
  api: 'cons', 
  callback: myLoginTestCallback, 
  data: 'method=loginTest'
});
```

`getAuth`

The getAuth method is used to retrieve an auth token for the current user. The resulting token is stored 
in luminateExtend.global. This method is automatically called by the request method if requiresAuth is 
true and no auth token has been defined.

``` js
luminateExtend.api.getAuth();
```

The getAuth method accepts one argument, an options object containing the following:

**callback:** The callback to be used after the auth token is retrieved. Note that unlike the request 
method, the callback is passed no data.

**useHTTPS:** A boolean indicating whether or not to use HTTPS when making the request.
 
`bind`

The bind method is used to simplify the process of authoring HTML forms that are intended to call the API 
upon submission. The method accepts one argument, a selector for the forms to be bound. If no selector is 
provided, the default is "form.luminateApi". Once bind has been called, the specified forms will have an 
onsubmit event attached to automatically call the request method.

``` js
luminateExtend.api.bind();
```

When the request method is called ...

 * The value for `api` is determined by the form action, which can be either an absolute reference to the 
 full API URL or a relative reference to the API servlet.
 * The value for `contentType` is set to the value of the enctype attribute if defined.
 * Any query strings included in the form action are passed as `data`.
 * The form's ID is passed as the value for `form`, and if the form has no ID, one is added.
 * The value for `requestType` is determined by the form method.
 * The value for `useHTTP` is determined by the protocol of the form action, or if the form action is 
 relative, by the protocol of the requesting page.
 * The values for `callback` and `requiresAuth` are set using an HTML5 data- attribute, data-luminateApi. 
 This attribute should be an object. Note that as the callback is provided as a string, it must be 
 defined in the global scope.

Here is an example of a fully functional login form, which will call the globally-defined loginCallback 
after login:

``` html
<form method="POST" action="http://www.myorganization.com/site/CRConsAPI" class="luminateApi" 
data-luminateApi='{"callback": "loginCallback"}'>
  <input type="hidden" name="method" value="login" />
  <p><label for="login_username">Username:</label><br />
  <input id="login_username" name="user_name" /></p>
  <p><label for="login_password">Password:</label><br />
  <input type="password" id="login_password" name="password" /></p>
  <p><input type="submit" value="Log In" /></p>
</form>
```

<a name="tagsObj"></a>
luminateExtend.tags
-------------------

As of v1.2, `luminateExtend.tags` allows for using "luminate tags" which are evaluated client-side. 
The purpose of these tags is to simplify the process of embedding dynamic content from Luminate Online. 
Currently this is limited to presenting logged in users with information from their constituent record 
using `luminate:cons` tags.

``` html
Welcome back, <luminate:cons field="name.first"></luminate:cons>!
```

The tags object contains the following:

`parse`

This method is used to parse all luminate tags of the specified type within a given selector.

``` js
luminateExtend.tags.parse();
```

luminateExtend.tags is an alias for the parse method when called directly.

``` js
luminateExtend.tags();
```

The parse method accepts two arguments:

**tagType:** The types of luminate tags to be evaluated, provided either as a string for a single type, or 
an array for multiple types. The string "all" can be used to evaluate all luminate tags. Currently the only 
supported tag type is "cons". If no value is provided, the default is "all".

luminate:cons tags are evaluated using the [getUser](http://open.convio.com/api/#constituent_api) API 
method. The field attribute is used to indicate which constituent field should be rendered.

``` html
Welcome back, <luminate:cons field="name.first"></luminate:cons>!
```

If the field specified does not exist, or if the constituent is not logged in, the parse method will 
fail silently.

**selector:** A selector for an element in which to limit the search for luminate tags. If no value is 
provided, the default is "body".

<a name="sessionVarsObj"></a>
luminateExtend.sessionVars
--------------------------

`luminateExtend.sessionVars` is used to manage Luminate Online session variables.

The sessionVars object contains the following:

`set`

This method is used to define a session variable. It accepts three arguments:

**varName:** The name of the session variable to be set.

**varValue:** The value to be set.

**callback:** The callback to be used after the session variable is set. Note that the callback is passed 
no data.

``` js
luminateExtend.sessionVars.set('src', 'mySourceCode');
```

<a name="utilsObj"></a>
luminateExtend.utils
--------------------

`luminateExtend.utils` provides helper functions that come in handy when working with Luminate Online and 
the API.

The utils object contains the following:

`ensureArray`

When the API returns a list of objects, the response format varies depending on the length of the list. 
If there is more than one of an object, an array is used, while if there is exactly one of an object, an 
array is *not* used. For example, when using the TeamRaiser 
[getTeamraisersByDistance](http://open.convio.com/api/#teamraiser_api.getTeamraisersByDistance_method.html) 
method, the response for two events might look like this:

``` js
{"teamraiser": [
  {
    "region": "Hill Country",
    "state": "TX", 
    "city": "Austin", 
    "id": "12000", 
    etc.
  }, 
  {
    "region": "Hill Country",
    "state": "TX", 
    "city": "Austin", 
    "id": "12001", 
    etc.
  }
]}
```

However, if event 12001 were later unpublished, the response would become:

``` js
{"teamraiser": 
  {
    "region": "Hill Country",
    "state": "TX", 
    "city": "Austin", 
    "id": "12000", 
    etc.
  }
}
```

The ensureArray method is used to guarantee that a given object is an array, regardless of length. This 
is necessary when iterating over a list, i.e. using [$.each()](http://api.jquery.com/jQuery.each/). The 
method accepts one argument, an object which may or may not be an array.

``` js
var teamraisers = luminateExtend.utils.ensureArray(data.getTeamraisersResponse.teamraiser);

$.each(teamraisers, function() {
  // do something with each TeamRaiser
});
```

`ping`

As its name implies, this method is used to ping the Luminate Online server. This is useful, for example, 
to keep a logged in user's session from timing out.

```  js
// when someone's session is about to expire, show them a lightbox that prompts them to click a 
// "Continue Working" button to stay logged in
// when the button is clicked, ping Luminate Online
$('#continueWorking').on('click', function(e) {
  luminateExtend.utils.ping();
});
```

The ping method accepts one optional argument, an options object containing the following:

**callback:** The callback to be used after pinging the server. Note that the callback is passed no 
data.

`simpleDateFormat`

Many API methods return dates and times formatted in ISO-8601 format. The library includes 
[SimpleDateFormatJS](https://github.com/noahcooper/SimpleDateFormatJS) in order to help format these 
dates in a human-readable way.

``` js
$.each(teamraisers, function() {
  // format dates following a pattern such as "September 21, 2012"
  var friendlyDate = luminateExtend.utils.simpleDateFormat(this.event_date, 'MMMM d, yyyy');
});
```

The simpleDateFormat method accepts two arguments:

**unformattedDate:** The date to be formatted. Can be provided as either an ISO-8601 string, or as a 
JavaScript Date object.

**pattern:** The pattern to use when formatting the date.

See [SimpleDateFormatJS](https://github.com/noahcooper/SimpleDateFormatJS) for additional documentation.

<a name="libBrowsers">
Browser support
---------------

luminateExtend.js includes support for the following browsers:

 * IE7+
 * Firefox 3.6+
 * Chrome 2+
 * Safari 4+
 * Opera 9+
 * Android

<a name="libIssues">
Reporting issues
----------------

Should you encounter any issues when using this library, please report them here, using the 
["Issues"](https://github.com/noahcooper/luminateExtend/issues) tab above. If you have general 
questions about the library, or about the API in general, the fastest way to get answers is to use 
the [Open APIs](http://community.convio.com/t5/Open-APIs/ct-p/OpenAPIs) section on 
http://community.convio.com.

Enjoy!