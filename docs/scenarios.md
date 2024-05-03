# Oberkorn Authorizator usage scenarios
In this section you will find different typical scenarios where Oberkorn can be used. Please review them and check if any of them matches your needs.

If you have a scenario that is not covered here, please contact us in order to review it and help you on configuring Oberkorn to protect the applications under your scenario.

## Basic scenario: a static HTML application
The simplest web application we can find is a classical 90-like web application, that is, a static HTML application containing static resources only. In such an application we may find the need for protecting some parts of the application, for example restrict the administrators area. Let's work on a simple sample.

We have developed a simple application that contains only HTML, JS, CSS and images. For managing the contents of the application there exist some pages that allow administrators of the application to upload new content (new images, for example). This administration area has been created under the '/admin' path. On the other side, public access is done via '/public'.

![Basic scenario](/_media/scenarios/scenario-basic.png)

For this application to be protected using Oberkorn we should create an Oberkorn authorizator like this:

```yaml
apiVersion: jfvilas.at.outlook.com/v1
kind: ObkAuthorizator
metadata:
  name: spa-authorizator
  namespace: test
spec:
  ingress:
    name: my-ingress
    provider: nginx-ingress
    class: nginx
  validators:
    - cognito:
        name: cognito-validator
        region: eu-west-1
        userpool: eu-west-1_abcdefg
        iss: https://cognito-idp.eu-west-1.amazonaws.com/eu-west-1_abcdefg
  rulesets:
    - name: general
      uriPrefix: [ '' ]
      # token must exist and be valid
      rules:
        - uri: "/admin/"
          uritype: "prefix"
          type: "valid"
        # unrestricted
        - uri: "/public/"
          uritype: "prefix"
          type: "unrestricted"
```

As you have guessed, every resource whose path starts with "/admin/..." needs a valid JWT token emited by our Cognito service to be accessed, and every resource under "/public/..." can be accessed freely. All other resource paths cannot be accessed.


## SPA scenario
One of the most useful scenarios is the one that cover the authorization needs of an SPA application. A Single Page Application (SPA) is a special architecture of web applications where:

  - Front application (the one that is loaded into the browser) is built as a static web application by using a framework like Angular, React or Vue, for example.
  - When users access the homepage of the application, several resources are downloaded to the browser, static resources like 'index.html', CSS, javasctipt, svg, png, etc.
  - The idea behind an SPA is that the index.html is loaded only once and there is no more HTML pages (thus the name Single Page Application), and there is no navigation to other pages. In fact, a typical SPA has only one HYML file, the 'index.html'.
  - When the front application has been loaded into the browser, the rest of the communication with the backend is based on REST API, that is, no more visual objects will be downloaded, only data will come-from or will be send-to the backend.

In an application architecture like this we tipically let the users access freely the static resources (index.html and other static files) and protect the calls to the application APIS's using a JWT token (or any other token type).

![SPA scenario](/_media/scenarios/scenario-spa.png)

For this application to be protected using Oberkorn we should create a Oberkorn authorizator like this:

```yaml
apiVersion: jfvilas.at.outlook.com/v1
kind: ObkAuthorizator
metadata:
  name: spa-authorizator
  namespace: test
spec:
  ingress:
    name: my-ingress
    provider: nginx-ingress
    class: nginx
  validators:
    - cognito:
        name: cognito-validator
        region: eu-west-1
        userpool: eu-west-1_abcdefg
        iss: https://cognito-idp.eu-west-1.amazonaws.com/eu-west-1_abcdefg
  rulesets:
    - name: general
      uriPrefix: [ '' ]
      rules:
        # token must exist and be valid
        - uri: "/api/"
          uritype: "prefix"
          type: "valid"
          onfalse: reject
        # unrestricted
        - uri: "/"
          uritype: "prefix"
          type: "unrestricted"
```
This YAML will create an Oberkorn authorizator which works like this:
  1. When the Nginx Ingress Controller named 'my-ingress' (in namespace 'test') receives an HTTP request, it routes the request to the Oberkorn authorizator.
  2. The authorizator checks all applicable the rules in the ruleset.
  3. If any rule evaluates to true, the authorizator answers the ingress with a positive response (HTTP 200).
  4. The ingress then re-routes the request to the appropriate backend.

If the response from the authorizator where negative, an HTTP 4xx status code would be sent back to the ingress, so the ingress would send a 4xx back to the user.

The YAML file we've just see has only 2 rules in the ruleset:
  - The first one evaluates to true if a request sent by a user (that matches the URI '/api/') contains a valid JWT token in the 'Authorization' HTTP header, since the rule type has been set to 'valid'. If the rule matches the URI, but the evaluation of the policy (to have a valid token) evaluates to false, the request is rejected and no more evaluation is performed (because of the 'onfalse' behaviour).
  - The second rule evaluates to true whenever a request that starts with "/" is received, since the type of rule has been set to 'unrestricted'.


## ASP/JSP (or whatever variant) scenario
Securing classic transactional web applications is somehow similar to classic static HTML applications, since there is no differnce between front and APIS (as it occurs in SPA). Let's define a simple example application:
  1. Application front page is index.html, which shows a login form.
  2. The rest of the aplication is made up of ASP pages.
  3. Static resources like images, styles and javascript is served from: /media, /css and /js respectively.
  4. Let's suppose there is an Azure B2C in place for protecting this applications.

In order to build a protection layer based on Oberkorn we should deploy an Oberkorn authorizator like this one:

```yaml
apiVersion: jfvilas.at.outlook.com/v1
kind: ObkAuthorizator
metadata:
  name: asp-authorizator
  namespace: test
spec:
  ingress:
    name: my-ingress
    provider: nginx-ingress
    class: nginx
  validators:
    - azure-b2c:
        name: my-b2c-tenant
        tenat: mytenant
        userflow: B2C_1_ropc
  rulesets:
    - name: general
      uriPrefix: [ '' ]
      rules:
        # if requested uri starts with /media/ or with /css/ or with /js, access is granted, the rule will be unrestricted 
        - uri: "^\/media\/|^\/css\/|^\/js\/"
          uritype: "regex"
          type: "unrestricted"
        # if the requested uri as an ASP page (i.e., URI ends with '.asp'), there must exist a valid token
        - uri: ".asp$"
          uritype: "regex"
          type: "valid"
          validators:
            - my-b2c-tenant
```
*Easy*, isn't it?


## WordPress scenario
How to secure a Wordpress application

In WordPress links are called **permalinks** (i.e. **perma**nent **links**). First thing you should do is check the permalinks settings in your WordPress admin area (click on Settings link in the admin menu and then click on Permalinks). You should see a configuration page like this one:

![Permalink settings](/_media/scenarios/wpsettings.png)

You can configure your permalinks using a predefined URI structure, like:

  - Sanitized post name.
  - Year, month and name.
  - Year, month, day and name.
  - ...

Suppose you want to have public access for recent content and protect access to old content for suscriptors. You could use an authorizator like this:

```yaml
apiVersion: jfvilas.at.outlook.com/v1
kind: ObkAuthorizator
metadata:
  name: spa-authorizator
  namespace: test
spec:
  ingress:
    name: my-ingress
    provider: nginx-ingress
    class: nginx
  validators:
    - cognito:
        name: cognito-validator
        region: eu-west-1
        userpool: eu-west-1_abcdefg
        iss: https://cognito-idp.eu-west-1.amazonaws.com/eu-west-1_abcdefg
  rulesets:
    - name: general
      uriPrefix: [ '' ]
      rules:
        # token must exist and be valid
        - uri: "/2023/"
          uritype: "prefix"
          type: "valid"
          onfalse: reject
        # unrestricted
        - uri: "/"
          uritype: "prefix"
          type: "unrestricted"
```
Please be aware of the order of processing: second rule is less restrictive, but it is executed only in the case the first rule did not match the requested URI. If requested URI matches 2023, first rule would be the last rule to evaluate, since 'ontrue' is 'accept' (default behaviour, so we don't specify it in the YAML) and 'onfalse' has been set to 'reject'.

Another way to configure protection for WordPress applications is to use custom permalinks and configure the auhtorizator ruleset accordingly. When you create a custom permalink format you can use this variables:

| Variable | Meaning |
|--|--|
|%year% | The year of the post (four digits) |
|%monthnum% | Month of the year (2 digits) |
|%day% | Day of the month (2 digits) |
|%hour% | Hour (2 digits) |
|%minute% | Minute (2 digits) |
|%second% | Second (2 digits) |
|%postname% | A sanitized version of the title of the post |
|%post_id% | The ID of the post (a number) |
|%category% | A sanitized version of the category name (you can see them in New/Edit Category panel). Nested sub-categories appear as nested directories in the URI |
|%author% | A sanitized version of the author name |

In the case of using custom permalinks, the protection mechanism is the same of a static HTML web application.

(Thanks to [WordPress Beginner](https://www.wpbeginner.com/wp-tutorials/seo-friendly-url-structure-for-wordpress))


## Classic Web authentication
Maybe you don't need something as conmplex as an IdM (Identity Manager), IdP (Identity Provider) and owrking with weird stuff lile OAtuh, JSON Web tokens, cypher, etc...
Maybe you just need to protect a web resource (a static HTML web application, for example) by using classic web authentication mechanisms. If this is your case, Basic Authentication is what you are looking for.

Oberkorn provides a special type of Validator (not OAuth-oriented) qich can help you protecting web resources using [Basic Authentication](https://en.wikipedia.org/wiki/Basic_access_authentication), that is, the one in which the browser asks the user for its credentials.

Suppose yo have a simple web application whose root path is "/home", and other stuff served on other URL paths that do not need to be protected. In order to build a protection layer based on Oberkorn we should deploy an Oberkorn authorizator like this one:

```yaml
apiVersion: jfvilas.at.outlook.com/v1
kind: ObkAuthorizator
metadata:
  name: basic-authentication-test
  namespace: test
spec:
  ingress:
    name: my-ingress
    provider: ingress-nginx
    class: nginx
  validators:
    - basic-auth-list:
        name: testBasicAuth
        realm: "Access to home site"
        users: 
          - name: u1
            password: p1
          - name: u2
            password: p2
  rulesets:
    - name: general
      uriPrefix: [ '' ]
      rules:
        # if requested uri starts with /home/ user must authenticate 
        - uri: "/home/"
          uritype: "prefix"
          validators:
            - testBasicAuth
```
*Easy* and **classic**, isn't it?

Please, take into account that the user list (and the passwords) are in fact a **static list**, this way of protecting resources has several specific use cases, like and administrator website, an operation website or simple applications like those. And, as you have guessed, users cannot change passwords.


## Using Google (SSO)
If you want your application to work with Google users you need to use a Google validator, what is really simple: it has no configuration parameters. The configuration must in fact be done on the login process and not in the token validation process.

To cover this scenario we will show you how to configure the login process and how not to configure the Google validator.

### 1. Login
You first need to create a clientid for this purpose in your Google cloud console. Follow this steps:

  1. Select your google cloud project in the console and navigate to 'APIs & Services'
  2. Select 'Credentials' on the left menu.
  3. Click 'Create credentials' and select the option 'OAuth client ID'. Fill-in the parameters:
     - **Application type** will be 'Web Application'.
     - **Name**: the name of your web application (feel free).
     - **Authorized JavaScript Origins**: the origin URL's that will invoke the login process, for example 'http://localhost'.
     - **Autorized redirect URIs**: the URI/URIs where Google redirect the user to after login (for example http://localhost).
  4. You will receive a client ID and a client secret (write it down immediately).

Now, on your web application you must configure the magic Google login button. Just paste this code (adding your client id in the corresponding parameter):

```html
<script src="https://accounts.google.com/gsi/client" async defer></script>
<div id="g_id_onload"
      data-client_id="ENTER-HERE-YOUR-CLIENT-ID"
      data-ux_mode="redirect"
      data-login_uri="http://localhost">
</div>
<div class="g_id_signin" data-type="standard"></div>
```

When a user succesfully signs in with this magic Google button, Google will redirect him to your redirect URI, but, instead of getting an access token in the query string (as you could expect..., a typical "http://localhost#acces_token=eyfhsjetutlkjsg...."), you'll get a POST to your redirect URI with the token in the body, in a parameter named 'credential'.

### 2. Protect your application with Google tokens
You have just performed the hardest part. Now protecting your endpoints by requesting users to present Google access tokens is as simple as defining a Google validator and adding it to your ruleset.

```yaml
apiVersion: jfvilas.at.outlook.com/v1
kind: ObkAuthorizator
metadata:
  name: ja-jfvilas
  namespace: dev
spec:
  config:
    replicas: 1
  ingress:
    name: ingress-jfvilas
    provider: ingress-nginx
    class: nginx
  validators:
    - google:
        name: testgoogle
        audience: ENTER-HERE-YOUR-CLIENT-ID
  rulesets:
    - name: general
      uriPrefix: [ '' ]
      rules:
        # unrestricted
        - uri: "/"
          uritype: "exact"
          type: "unrestricted"
        # user must present a valid (not expired, not corrupt) token
        - uri: "/protect/"
          uritype: "prefix"
          type: "valid"
          validators: [ testgoogle ]
```
This authorizator:
  1. Allows free access to your website root ("/").
  2. Requires a valid token emitted specifically to your application in order to let the users access the path "/protect/...". This job is done thanks to the use of the **audience** property when defining the validator, where you must enter your client id. If you don't specify and audience, any Google user could use your application.

*Easy* and **googlefied**, isn't it?
