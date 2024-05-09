# Reference
Here you can find reference information for creating and operating an Oberkorn authorizator. This reference page documents all the parameters you can/must code in a YAML file in order to create or modify an Oberkorn authorizator via 'kubectl apply -f'.

## YAML
To create an Oberkorn authorizator you must create a YAML file containing all the configuration that defines your needs to protect your web resources running inside your Kubernetes cluster. what follows is a basic (minimal, with no rule restrictions) Oberkorn authorizator:

```yaml
apiVersion: jfvilas.at.outlook.com/v1
kind: ObkAuthorizator
metadata:
  name: sample-authorizator
  namespace: test
spec:
  config:
    replicas: 2
  ingress:
    name: sample-nginx-ingress
    class: nginx
  validators:
    - cognito:
        name: cognito-validator
        region: eu-west-1
        userpool: eu-west-1_abcdefg
        iss: https://cognito-idp.eu-west-1.amazonaws.com/eu-west-1_abcdefg
  rulesets:
    # unrestricted
    - uriPrefix: [ '' ]
      name: global-ruleset
      - uri: "/"
        uritype: "prefix"
        type: "unrestricted"
```

Let's explain the content. As any usual kubernetes resource, you must specify the 'apiVersion' and the 'kind' of the object you are defining. In this case, we are creating an Oberkorn Authorizator, so the kind is 'ObkAuthorizator' (this kind is created when you install the Oberkorn controller, in the CRD file).

The 'metadata' section works as usual, you must define the name of the authorizator you are creating and the 'namespace' it belongs to.

Next section of the YAML is were the magic is built: **the *spec***. The 'spec' section is divided into 4 sections:
  - *config*. The 'config' section defines general behaviour of the authorizator, like the number of replicas you want to have.
  - *ingress*. The 'ingress' section is where you can correlate an authorizator with an ingress.
  - *validators*. In 'validators' section you can define all the validator services you will use all along your ruleset to protect your applications.
  - *rulesets*. Here you define all the rulesets that protect your applications. Each ruleset is a set of rules.


## **config**
'config' is a general section where you define how the authorizator will behave regarding the performance or monitoring, that is, execution related aspects. These are the 'config' parameters.

##### replicas [number] [optional] [default: 1]
You can specify how many containers will be running in parallel in order to fulfill request validations from users. The default value is 1, but this value can grow according to your applications needs.

##### api [boolean] [optional] [default: false]
If you want to manage the authorizator via API or you want to use the controller web console to manage the authorizator, you must enable the API interface (and do some addiotional configuration on services and ingresses in order to make the API reachable from outside the cluster).

API must be enabled if you plan to manage your authorizator via the Oberkorn web console. Enabling API enables then endpoint, but it will be only accesible inside the cluster, which is engough for using the web console. But, if you plan to access authorizator API from outside the cluser (for example, for managing your authorizator from your operations tools, like Ansible, puppet, VROps, etc.), you must publish the endpint via an ingress.

When you create an authorizator, the controller creates a service to encapsulate access to all replicas of the authorizator. Such service is named:

```yaml
obk-authorizator-YOURAUTHORIZTORNAME-svc
```

That is, if you create an authorizator named 'auth1', the service would be named: obk-authorizator-auth1-svc.

So, in order to access this authorizator form outside the cluster, you would create an ingress rule like this inside your ingress:

```yaml
rules:
- host: localhost
  http:
    paths:
      # access to authrizator API (not needed by the console, but useful if you to manage authorizator externally)
      - path: /obk-authorizator/dev/auth1
        pathType: Prefix
        backend:
          service:
            name: obk-authorizator-auth1-svc
            port:
              number: 3882
```
Please pay attention to the path, which is formed by:

  - A fixed prefix, '**obk-authorizator**'.
  - The authorizator namespace, '**dev**' in this sampe.
  - The authorizator name, '**auth1**' in thsi sample.

As you can see, request received at that path would be routed to the service named '**obk-authorizator-auth1-svc**'.

##### logLevel [number] [optional] [default: 0]
If your want to have more info on what is doing the authorizator you can increase the log level. Default value (0), only shows authorizator starting info and severe errors.

##### prometheus [boolean] [optional] [default: false]
If you plan to monitor your authorizator performance by using Prometheus, the only thing you need to do is set this parameter to 'true' and configure a target in your Prometheus cluster. Please take into account that if your Promethus is running outside the kubernetes cluster, you must add an ingress to allow access to '/metrics' endpoint of your authorizators.

Following you can find a sample config section:

```yaml
config:
  replicas: 3
  api: true
  prometheus: true
```

## **ingress**
The ingess section is used to establish a permanent relation between an ingress controller and an Oberkorn authorizator. You only need to set the name of the ingress (namespace must be the same as the one specified in the authorizator).

##### name [string] [mandatory]
The name of the ingress that will send authorization requests to this authorizator.

##### provider [string] [mandatory]
The provider of the ingress controller. Currently these are the supported providers:

| Provider | Value | Notes |
|-|-|-|
| Ingress Nginx| ingress-nginx | This is the community maintained open source ingress based on Nginx. |
| NGINX Ingress| nginx-ingress | |
| HAProxy | haproxy | It's still under testing. |
| Traefik | traefik | |

##### class [string] [mandatory]
The ingress class of the ingress.

Following you can find a sample config section:

```yaml
ingress:
  name: sample-ingress
  provider: traefik
  class: nginx
```

## **validators**
The 'validators' section contains a set of validators of different types that you will use for protecting your applications.

Each validator has its own type, and thus its own specific paramters to configure it. Current version of Oberkorn supports following validators:

  - Azure B2C
  - Google
  - AWS Cognito
  - Azure AD (Entra ID)
  - KeyCloak
  - Basic Auth (classical web authentication)
  - Custom

All validators include, aside from its specific configuration parameters, this 3 properties:

##### name [string] [mandatory]
The name of the validator, that you will use later to link a specific rule to a validator. Please note that the name must be unique, but you can create different validators of the same type.

##### iss [string] [optional]
You can define a validator just defining its name, its tenant and its userflow (for instance). Later, when creating the rules, you can add conditions to the tokens emitted by a specific validator. Anyway, if you want to enforce a specific 'issuer' for the JWT tokens you plan to use, you can specify here a issuer value in such a way that every token to be validated against this validator must have an 'iss' value that matches the one specified here.

##### aud [string] [optional]
In a similar way, when creating rules you can add conditions to the tokens emitted by a specific validator. If you want to enforce a specific 'audience', you can specify here an audience value in such a way that every token to be validated against this validator must have an 'aud' value that matches the one specified here.

##### verify [boolean] [optional]
When a request needs to be validated, the authorizator will:
  1. Decode the token.
  2. Verify the signature of the token.

This is the default behaviour, but, when working with corporate applications that are not exposed to internet, that is, runnning in a trusty environment, the verification of the token may not be needed, so you can disable the verification by setting this parameter to 'false'. The JWT tokens will be decoded and its content will be checked if a rule requires doing it, but the signatures will not be verified.

**NOTE:** Validators that do not use cryptographic systems to cypher and sign tokens do ignore the 'verify' property, they do just decoding and validation.


### Azure B2C
These are the properties that define an Azure B2C validator.

##### tenant [string] [mandatory]
The tentant name of your Azure B2C service.

##### userflow [string] [mandatory]
The name of the user flow to use.


### AWS Cognito
These are the properties that define an AWS Cognito validator.

##### region [string] [mandatory]
The region where your AWS Cognito service has been deployed, something like "eu-west-1" or "us-east-2".

##### userpool [string] [mandatory]
The name of the userpool of your AWS Cognito service, something like "eu-west-1_up4ur74up" or " us-east-2_47h3hkgmu"


### Google
These are the properties that define an Google validator. Please keep in mind you need a client-id for user sign-in, but you don't need it to use, decode or verify tokens. this validator can be used as an **SSO mechanism for Google IdP-managed users**.

There is no configuration needed for using Google tokens, since this validator uses Google users (there is no tenant, the tenant so Google itself).

#### Example
Following you can find a sample 'validators' section including a Google validator.
```yaml
validators:
  - google:
      name: test-google-validator
```

Please refer to scenarios page to review some samples on how to use this SSO validator. 


### Azure AD
These are the properties that define an Azure AD validator.

##### tenant [string] [mandatory]
The name of the tentant of your Entra ID service.


### Keycloak
These are the properties that define a KeyCloak validator.

##### url [string] [mandatory]
The base URL where the KeyCloak cab be accessed. Typucally it would conntain a vlaue like "https://my.keycloak.deployment.com", but, if the KeyCloak you want to use has been deloyed the tha same Kubernetes cluster where Oberkorn runs, then you should use internal DNS name, like "http://keycloak.dev.svc.cluster.local:8080", for example, using standard DNS names inside Kubernetes (service name, namespace and the "cluster.local" TLD).

##### realm [string] [mandatory]
The name of the realm you want to use.

#### Examples
Following you can find a sample 'validators' section including two validators. First one nly checks that a token is present, and the second onw validates de token and the audience.
```yaml
validators:
  - keycloak:
      name: coporate-kc
      realm: company-one
      url: http://keycloak.dev.svc.cluster.local
      verify: false
  - keycloak:
      name: external-keycloak
      url: https://sample.keycloak.server
      aud: aa1be673-598c-4712-a211-69abcde6786d
      verify: true
```

### Basic Auth
A Basic Authentication works like classical web authentication, what follows is a typical scenario:

  1. The user tries to access a resource, an URI.
  2. The browser doesn't send any kind of authentication info.
  3. The server answers the browser by sending a 401 and adding a 'WWW-Authenticate' header.
  4. The user enters its credentilas (user and password).
  5. The browser re-sends the original request adding an Authorization header.

Basic Auth validator can work in two different modes:

  - **Static user list**. The validator uses a static list of users and passwords that are read from the YAML definition.
  - **Secret user list**. The validator uses a Kubernetes secret to read/write users database (initial values can be read from the YAML definition).

These are the properties that define a Basic Auth validator.

##### realm [string] [mandatory]
The name of the realm you want to use.

##### storeType [string] [mandatory]
Type of storage used for storing users and passwords. Two values are possible:

  - '**inline**', users are read from the validator configuration YAML.
  - '**secret**', users are read from the validator YAML if the exist, and will be stored and retrieved in a Kubernetes secret.

'inline' users database is just the YAML, while 'secret' users database is a Kuberntes secret. This implies two main differences:

  - Users database (users and also its passwords) are static in 'inline' store type, they com form the authorizator YAML and they are always the same.
  - Users database is stored on a Kubernetes secret on 'secret' store type, so passwords are cyphered (at least they are protected since they are stored outside the YAML), and thus **they are variable**. I mean, if you recreate a type 'secret' authorizator, the users/passwords database will be read from the kubernetes secret, no matter which users and passwords are present in the authorizator YAML.

But there is a third difference very important: in 'secret' store type, users **can change their passwords**, so...
  1. You initialize the passwords when you create the authorizator.
  2. Users and passwords are cpied (or merged) with the ones present in the Kubernetes secret.
  3. User can change their passwords when signing in, as we explain below.

###### Change password
For a user to change its password with Basic Auth authrizators of 'secret' type, the must follow this procedure:
  1. Try to access the protected resource (a web page or whatever).
  2. Your browser will ask you to enter a user and a password. At this moment enter your user and for the password field enter: your **current password** followed by **one blank space** followed by the **new password**.
  3. The browser will ask you to enter your credentials again. At this moment you must enter your user and the new password (this is the confirmation step).
  4. From now on, you have a new password. Enjoy it and use it responsibily!

##### storeSecret [string] [optional]
Yhe name of a Kubernetes secret where users and passwords will be stored.

##### storeKey [string] [optional]
The key inside the secret that will hold the users and passwords. Users and passwords are stored as a JSON object with this format:

{ 'user1':'password1', 'user2':'password2' }

##### users [array] [optional]
You must add here an array of users and passwords (See sample for correct syntax).


#### Examples
Following you can find a sample Basic Auth validator of type "inline", that is, a *static list* of users (and their passwords).

```yaml
validators:
  - basicAuth:
      name: testBasicList
      realm: testrealm
      storeType: inline
      users: 
        - name: julio
          password: angel
        - name: u2
          password: p2
```

What follows is a Basic Auth validator that stores users and passwords in a *Kubernetes secret*.

```yaml
validators:
  - basicAuth:
      name: testSecret
      realm: testrealm
      storeType: secret
      storeSecret: users
      storeKey: db
      users: 
        - name: u1
          password: p1333
        - name: u2
          password: p2333
```

You can add initial users/passwords by using 'users' property or you can just rely on your secret. The content of the secret must be a JSON string with this format:

```JSON
{ 'user': 'password', 'user2': 'password2'... }
```

When the authorizator starts, it reads all users existing in the YAML file and loads them into the secret key **IF THEY DO NOT ALREADY EXIST**. If a user exists in the YAML and in the secret key at the same time, the authorizator will do noting with the user (keeping his password unchanged).


### Custom
The Custom validator allow you to define your own validation mechanism via a JavaScript function. This is how it works:

  1. The end user sends a request with authorization info (via Authorization header).
  2. The Custom validator extracts data from the header and sends it to your function.
  3. Your function will evaluate the request according to your own rules, and you decide what to do:
      - If you want to authorize the request your function must send back a string with access information (or whatever you want, but not **undefined**).
      - If you want to reject the request, the ***function must return undefined***.

How do you configure a validator like this? These are the basic steps:

  1. You must create an authorization function like this:
     ```javascript
        function (context) {
          if (context.token!==null && context.token.toLowercase()==='enjoy')
            return "Ok";
          else
            return undefined;
        }
     ```
  2. You must store the function in a config map, and give access to the authorizator so it can read the config map (using a kubernetes role and a kubernetes role binding). You can find an example in the samples folder (named 'sample-custom.yaml').
  3. You configure the validator adding the needed properties to use the validator.

what follow are the parameters needed to configure a Custom validator.

##### configMap [string] [mandatory]
The name of the config map that holds the JavaScript function.

##### key [string] [mandatory]
The name of the key (inside the config map) that holds the JavaScript function

#### Examples
Following you can find a sample Custom validator (please refer to samples directoy inside obk-authorizator project for a complete sample).

```yaml
  validators:
    - custom:
        name: testcustom
        configMap: test-custom-validator-cm
        key: authorize-function
```


## **rulesets**
This is where you must define what rules will drive the security of your applications. As the name states, the content of this section is a set of rulesets (each ruleset is an array of rules, in fact).
When a user request needs to be validated, all the rulesets uri prefixes are evalauated against the request URI in order to decide what ruleset should be used for evaluating the request. When a ruleset has been selected, all the rules in the ruleset are evaluated unless one of them evaulates to 'true' (or a behaviour indicates thet processing must stop). In this case, no more rule evaluation is performed, and a response is sent back to the ingress controller.

This default way of evaluating rules can be modified by changing the behaviours (as explained downwards). These are the properties that define a ruleset:

##### name [string] [mandatory]
This property defines the name of a ruleset. A ruleset is typically a set of rules created for protecting one application, so it would be a good idea to correlate the ruleset name with the application name.

##### uriPrefix [array of string] [mandatory]
This is a list of URI prefixes that will be used to match user requests for deciding what ruleset to use when a request is received. For example, let's assume we have created two rulestes like this:

```yaml
rulesets:
  - name: general
    uriPrefix: [ '/favicon.ico'  ]
    rules: 
      # unrestricted
      - uris: [ '' ]
        uritype: exact
        type: unrestricted
  - name: app1
    uriPrefix: [ '/app1' ]
    rules: 
      # unrestricted
      - uri: ''
        uritype: prefix
        type: valid
        validators: [ 'val1' ]
```

When the user requests access to a URI like 'http://your.dns.name/app1/index.html' the authorizator will select the second ruleset, and after that, all the rules in the ruleset named **'app1'** will be evaluated.

Imagine tha 'index.html' page has a referen to the *favicon* in the root directory, that is, something like 'http://your.dns.name/favicon.ico'. In such a case the authorizator would select the ruleset named 'general' and it would concede access, since there is an 'unrestricted' rule that matches the user request.

##### rules
The 'rules' parameter is an array of rules. These are the properties that each rule must/can contain:

##### uri [string] [optional]
This is a uri specification that must match user request in order to be evaluated. If the URI the user wants to access does not match this parameter, this rule will not be evaluated. URI's inside rules are relative, that is, do not include the hostname nor the ruleset uri prefix, they are referred to **just the local path**.

##### uris [array of string] [optional]
If you face the need of creating a rule that is applicable to more than one uri, you can use the 'uris' parameter instead of a single 'uri' parameter. In this case you can provide an array of uris like in this example:

```yaml
rules: 
  # unrestricted
  - uris: [ '/static', '/js', '/css' ]
    uritype: exact
    type: unrestricted
  # protected
  - uri: '/api'
    uritype: exact
    type: unrestricted
```

Parameters 'uri' and 'uris' do exist at the same time for your convenience, you can use one of them, just the other one or even both of them. When evaluating rules the authorizator will check all of them. So, if you create a rule like this one:

```yaml
rules:
  - uri: '/static'
    uris: [ '/js', '/css' ]
    uritype: exact
    type: unrestricted
```

In this case, the authorizator will evaluate all three paths (static, js and css) against user request.

##### uritype [string] [mandatory]
Every URI in a rule can be one of this types:

  - 'exact'. The URI requested by the user must be exactly equal to the URI of this rule.
  - 'prefix'. The URI requested by the user must have its first characters exactly equal to the characters in this rule.
  - 'regex'. The URI requested by the user must match the regex specified in this rule URI.

Example:
```yaml
  - uri: '/public'
    uritype: 'exact'
  - uri: '/api/'
    uritype: 'prefix'
  - uri: '\/media\/\d\d\d\.jpeg'
    uritype: 'regex'
```
First rule will be considered to be evaluated if the URI requested by the user is excatley '/public' (case sensitive).
Second rule whill be considered to be evaluated if the URI requested by the user does start with '/api/'. For example, if the request URI is '/api/customer' the rule will be evaluated. But, if the requested URI is '/apiculture', the rule will not be evaluated.
The third rule will be evaluated if the requested URI matches the regex. For example, if the user requests '/media/image.jpeg', the rule will not be evaluated. But, if the requested URI is '/media/345.jpeg' the rule will be evaluated.

Please be aware that the uri type has a rule scope, so all uris in the rule will have the same type.

##### type [string] [mandatory]
Every rule has a type, which defines how the rule will be evaluated. These are the different types supported:

  - unrestricted
  - valid
  - claim
  - and
  - or

###### *unrestricted*
When rule type is set to unrestricted there will be no evaluation, the result of evaluating an 'unrestricted' rule will always be positive ('true'), no matter the token, no matter the request context.

###### *valid*
The 'valid' rule type will evaluate to true if a token (JWT or any other type) is present (in the Authorization header) and the token is valid (it is not expired nor has been manipulated).

###### *claim*
A 'claim' rule will evaluate to 'true' if a specific claim in the token satisfies the policy specified in the rule (see 'policy' downwards).

###### *and* and *or*
In order to have a way of creating rules with complex conditions that the tokens (JWT or whatever) must satisfy, you can create subsets of rules using 'and' and 'or'. As you have guessed, an 'and' rule type will evaluate to 'true' if all the sub-rules evaluate to true.

Conversely, an 'or' rule type will evaluate to 'true' if at least just one of the subrules evaluates to 'true'. When a subrule evaluation returns 'true', the authorizator will stop checking the rest of the subrules... there is no need.

The sub-rules of 'and' and 'or' rules can be specified usign the parameter 'subset' (see below).

##### policy [string] [optional]
Every 'claim' rule type must have a policy. A policy states how a rule type must be evaluated. These are the currently supported policies:

  - present
  - notpresent
  - is
  - containsany
  - containsall
  - matchesany
  - matchesall

You can find the expanation and applicability of each of the policies in following table.

>| policy        |  meaning |
 |--|--|
 |present        | The claim must exist in the token, that is, it must be present. |
 | notpresent    | The claim must not exist in the token |
 | is            | The claim must match any of the values specified in the 'values' parameter (see below) |
 | containsall   | The claim must contain ALL the values specified in the 'values' parameter (see below) |
 | containsany   | The claim must contain at least ONE of the values specified in the 'values' parameter (see below) |
 | matchesall    | The claim must match (using regex) ALL the regular expressions specified in the 'values' parameter (see below) |
 | matchesany    | The claim must match (using regex) at least ONE of the regular expressions specified in the 'values' parameter (see below) |

##### values [string] [optional]
'values' conatins an array (a list) of values used by the policy of the rule. For example, if you have a claim named 'user_attributes' which can contain any values like USER, ADMIN or MANAGER, and you want to write a rule to validate tokens emitted for managerial people (that is ADMIN and MANAGER), you should write a rule like this:

```yaml
  - uri: '/api/manage'
    uritype: exact
    type: claim
    policy: containsany
    values:
      - ADMIN
      - MANAGER
```

##### options [string] [optional]
When evaluating values you may need some configuration on how to perform that evaluation. You can manage it by adding a list of options. Current supported options are:

  - lowercase
  - uppercase

When evaluating policies like 'is' or 'containsall', for example, you can force the evalutation to be performed using the right case. Please note that the options do affect the values contained in the tokens to be evaluated, **not** the values in the rule.

If you do not specify lowercase or uppercase, the comparisons will be performed using a case-sensitive comparison.


##### subset [array of rule] [optional]
A subset is a set of rules contained inside a rule (like a ruleset, but specific to a rule). Subsets are only used when the **type** of a rule is 'and' or 'or'. As explainied before...

  - If the rule type is 'and' all the rules in the subset must evaluate to true in order to return a positive response to the ingress. When a rule in the subset evaluates to false, the evaluation of the subset is inmediatly stopped and rule evaluation is set to 'false'.
  - If the rule type is 'or' at least one rule in the subset must evaluate to true in order to return a positive response to the ingress. When a rule in the subset evaluates to true, the evaluation of the subset is inmediatly stopped and rule evaluation is set to 'true'. If none of the subrules evaluates to true, a negative response is sent back to the ingress.

For instance, the following rule ('and' type) will be evaluated to true if the token presented by the requestor contains a claim named 'USERTYPE' that is a MANAGER and **do** **not** contain a claim named 'BAN'.

```yaml
    - uri: "/a-rule"
      uritype: "exact"    
      type: "and"
      subset:
        - type: "claim"
          name: "USERTYPE"
          policy: "is"
          values : [ 'MANAGER' ]
        - type: "claim"
          name: "BAN"
          policy: "notpresent"
```

#### Behaviours: ontrue & onfalse
A behaviour is a specification on how the rule engine must manage the result of the evaluation of a rule. A behaviour can be specified positively or negatively, that is, you can indicate what to do when a positive evaluation (true) is received, and what to do when receiving a negative evaluation (false).

The behaviours can be of any these two types:
  - **ontrue**, the behaviour to apply when the evaluation of the rule is true.
  - **onfalse**, the behaviour to apply when the evaluation of the rule is false.

And the actions to be performed by the rule engine when applying a behaviour can be any of these three types:

  - **accept**, accept the request of the user and stop processing rules.
  - **reject**, reject the request of the user and stop processing rules.
  - **continue**, continue processing with the next rule in the ruleset.

If a behaviour is not specified, these are the default behaviours assumed by the rule engine:

  - **ontrue: accept**
  - **onfalse: continue**

That means, if a rule evaluates to true the access is granted. If a rule evaluates to false, the evaluation process continues with next rule in the ruleset.

##### ontrue [string] [optional] [default: accept]
This parameter indicates what to do with a positive (true) response after evaluating a rule. Possible values are: **accept**, **reject** or **continue**.

##### onfalse [string] [optional] [default: continue]
This parameter indicates what to do with a negative (false) response after evaluating a rule. Possible values are: **accept**, **reject** or **continue**.



## Compatibility matrix
Oberkorn (including controller and authorizator modules) has been succesfully tested under the following conditions:

**Kubernetes**

| Distribution | Version | Notes|
|-|-|-|
| K3s | v1.27.4-k3s1 | |
| Azure Kubernetes Service (AKS) | 1.27.7 <br/> 1.26.6 | |

**Ingress**

| Ingress | Version | Notes |
|-|-|-|
| Ingress Nginx | 1.9.4 <br/> 1.8.9 ||
| NGINX Ingress | 3.3.2 <br/> 3.3.1 ||
| HA Proxy Ingress | TBT ||
| Traefik Ingress | 2.10.7 ||

**IdM**

| Identity Manager | Version | Notes |
|-|-|-|
| KeyCloak | 24.0.2 ||
