# JWT Authorizator usage scenarios
In this section you will find different typical scenarios where Oberkorn can be used. Please review them and check if any of them matches your needs.

If you have a scenario that is not covered here, please contact us in order to review it and help you on how to configure Oberkorn to protect your applications.

## Basic scenario: a static HTML application
The simplest web application we can find is a classical 90-like web application, that is, a static HTML application containing static resources only. In such an application we may find the need for protecting some parts of the application, for example restrict the administrators area. Let's work on a simple sample.

We have developed a simple application that contains only HTML, JS, CSS and images. For managing the contents of the application there exist some pages that allow administrators of the application to upload new content (new images, for example), this administration area has been created under the '/admin' path. On the other side, public access is done via '/public'.

>Diagram of the architecture of the application.

For this application to be protected using Oberkorn we should create an Oberkorn authorizator like this:

```yaml
apiVersion: jfvilas.at.outlook.com/v1
kind: JwtAuthorizator
metadata:
  name: spa-authorizator
  namespace: test
spec:
  ingress:
    name: sample-nginx-ingress
    class: nginx
  validators:
    - cognito:
        name: cognito-validator
        region: eu-west-1
        userpool: eu-west-1_abcdefg
        iss: https://cognito-idp.eu-west-1.amazonaws.com/eu-west-1_abcdefg
  ruleset:
    # token must exist and be valid
    - uri: "/admin/"
      uritype: "prefix"
      type: "valid"
    # unrestricted
    - uri: "/public/"
      uritype: "prefix"
      type: "unrestricted"
```

As you may suppose, every resource whose path starts with "/admin/..." needs a valid JWT token to be accessed, and every resource under "/public/..." can be accessed freely. All other resource paths cannot be accessed.


## SPA scenario
One of the most useful scenarios is the one that cover the authorization needs of an SPA application. A Single Page Application (SPA) is a special architecture of web applications where:

  - Front application (the one that is loaded into the browser) is built as a static web application by using a framework like Angular, React of Vue, for example.
  - When users access the home page of the application, several resources are downloaded to the browser, static resources like 'index.html', CSS, javasctipt, svg, png, etc.
  - The idea behind an SPA is that the index.html is loaded only once and there is no more HTML pages (thus the name Single Page Application), and there is no navigation to other pages. In fact, a typical SPA has only one html file, the 'index.html'.
  - When the front application has been loaded in the browser, the rest of the communication with the backend is based on REST API, that is, no more visual objects will be downloaded, only data will come-from or send-to the backend.

In an application architecture like this we tipically let the users access freely the static resources (index.html and other static files) and protect the calls to the application APIS's using a JWT token.

>diagram

For this application to be protected using Oberkorn we should create a Oberkorn authorizator like this:

```yaml
apiVersion: jfvilas.at.outlook.com/v1
kind: JwtAuthorizator
metadata:
  name: spa-authorizator
  namespace: test
spec:
  ingress:
    name: sample-nginx-ingress
    class: nginx
  validators:
    - cognito:
        name: cognito-validator
        region: eu-west-1
        userpool: eu-west-1_abcdefg
        iss: https://cognito-idp.eu-west-1.amazonaws.com/eu-west-1_abcdefg
  ruleset:
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
  1. When the Nginx Ingress Controller named 'sample-nginx-ingress' (in namespace 'test') receives an HTTP request, it routes the request to the Oberkorn authorizator.
  2. The authorizator checks all the rules in the ruleset.
  3. If any rule evaluates to true, the authorizator answers the ingress with a positive response (HTTP 200).
  4. The ingress then re-routes the request to the appropriate backend.

If the response from the authorizator where negative, an HTTP 401 status code would be sent back to the ingress, so the ingress would send a 403 back to the user.

The YAML file we've just see has only 2 rules in the ruleset:
  - The first one evaluates to true if a request sent by a user (that matches the URI '/api/') contains a valid JWT token in the 'Authorization' HTTP header, since the rule type has been set to 'valid'. If the rule matches the URI, but the evaluation of the policy (to have a valid token) evaluates to false, the request is rejected and no more evaluation is performed.
  - The second rule evaluates to true whenever a request that starts with "/" is received, since the type of rule has been set to 'unrestricted'.

## ASP/JSP (or whatever variant) scenario
Securing classic transactional web applications is somehow similar to classic static HTML applications, since there is no differnce between front and APIS (as it occurs in SPA). Let's define a simple example application:
  1. Application front page is index.html, which shows a login form.
  2. The rest of the aplication is made up of ASP pages.
  3. Static resurces like images, styles and javascript is served from: /media, /css and /js respectively.
  4. Let's suppose there is an Azure B2C in place for protecting this applications.

In order to build a protection layer based on Oberkorn we should deploy an Oberkorn authorizator like this one:

```yaml
apiVersion: jfvilas.at.outlook.com/v1
kind: JwtAuthorizator
metadata:
  name: asp-authorizator
  namespace: test
spec:
  ingress:
    name: sample-nginx-ingress
    class: nginx
  validators:
    - azure-b2c:
        name: my-b2c-tenant
        tenat: mytenant
        userflow: B2C_1_ropc
  ruleset:
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

You ca configure your permalinks using a predefined URI structure, like:

  - Sanitized post name.
  - Year, month and name.
  - Year, month, day and name.
  - ...

Suppose you want to have public access for recent content and protect access to old content for suscriptiors. You could use an authorizator like this:

```yaml
apiVersion: jfvilas.at.outlook.com/v1
kind: JwtAuthorizator
metadata:
  name: spa-authorizator
  namespace: test
spec:
  ingress:
    name: sample-nginx-ingress
    class: nginx
  validators:
    - cognito:
        name: cognito-validator
        region: eu-west-1
        userpool: eu-west-1_abcdefg
        iss: https://cognito-idp.eu-west-1.amazonaws.com/eu-west-1_abcdefg
  ruleset:
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
Please be aware of the order of processing: second rule is less restrictive, but it is executed only in the case the first rule did not match the requested URI. If requested URI matches 2023, first rule would be the last rule to evaluate, since 'ontrue' is 'accept' (default behaviour) and 'onfalse' has been set to 'reject'.

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

