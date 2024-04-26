# Reference
Here you can find reference information for creating and operating an Oberkorn authorizator. This reference page documents all the parameters you can/must code in a YAML file in order to create an authorizator via 'kubectl apply -f'.

## YAML
To create an Oberkorn authorizator you must create a YAML file containing all the configuration that defines your needs to protect your web applications running in the Kubernetes cluster. This is a basic (minimal, with no rule restrictions) Oberkorn authorizator:

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
  ruleset:
    # unrestricted
    - uri: "/"
      uritype: "prefix"
      type: "unrestricted"
```

Let's explain the content. As any usual kubernetes resource, you must specify the 'apiVersion' and the 'kind' of the object you are defining. In this case, we are creating an Oberkorn Authorizator, so the kind is 'ObkAuthorizator'.

The 'metadata' section works as usual, you must define the name of the authorizator you are creating and the 'namespace' it belongs to.

Next section of the YAML is were the magic is built: **the *spec***. The 'spec' section is divided into 4 sections:
  - *config*. The 'config' section defines general behaviour of the authorizator, like the number of replicas you want to have.
  - *ingress*. The 'ingress' section is where you can correlate an authorizator with an ingress.
  - *validators*. In 'validators' section you can define all the validator services you will use all along your ruleset to protect your applications.
  - *ruleset*. Here you define all the rules that protect your applications.


## **config**
'config' is a general section where you define how the authorizator will behave regarding the performance or monitoring, that is, execution related aspects. These are the 'config' parameters.

##### replicas [optional] [number]
You can specify how many containers will be running in parallel in order to fulfill request validations from users. The default value is 1, but this value can grow according to your applications needs.

##### prometheus [optional] [boolean]
If you plan to monitor your authorizator performance by using Prometheus, the only thing you need to do is set this parameter to 'true' and configure a target in your Prometheus cluster. Please take into account that if your Promethus is running outside the kubernetes cluster, you must add an ingress to allow access to '/metrics' endpoint of your authorizators.

Following you can find a sample config section:

```yaml
config:
  replicas: 3
  prometheus: true
```

## **ingress**
The ingess section is used to establish a permanent relation between an ingress controller and an Oberkorn authorizator. You only need to set the name of the ingress (namespace must be the same as the one specified in the authorizator).

##### name [mandatory] [string]
The name of the ingress that will send authorization requests to this authorizator.

##### provider [mandatory] [string]
The provider of the ingress controller. Currently these are the supported providers:

| Provider | Value | Notes |
|-|-|-|
| Ingress Nginx| ingress-nginx | This is the community maintained open source ingress based on Nginx. |
| NGINX Ingress| nginx-ingress | |
| HAProxy | haproxy | It's still under testing. |
| Traefik | traefik | |

##### class [mandatory] [string]
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

##### name [mandatory] [string]
The name of the validator, that you will use later to link a specific rule to a validator. Please note that the name must be unique, but you can create different validators of the same type.

##### iss [optional] [string]
You can define a validator just defining its name, its tenant and its userflow (for instance). Later, when creating the rules, you can add conditions to the tokens emitted by a specific validator. Anyway, if you want to enforce a specific 'issuer' for the JWT tokens you plan to use, you can specify here a issuer value in such a way that every token to be validated against this validator must have an 'iss' value that matches the one specified here.

##### aud [optional] [string]
In a similar way, when creating rules you can add conditions to the tokens emitted by a specific validator. If you want to enforce a specific 'audience', you can specify here an audience value in such a way that every token to be validated against this validator must have an 'aud' value that matches the one specified here.

##### verify [optional] [boolean]
When a request needs to be validated, the authorizator will:
  1. Decode the token.
  2. Verify the signature of the token.

This is the default behaviour, but, when working with corporate applications that are not exposed to internet, that is, runnning in a trusty environment, the verification of the token may not be needed, so you can disable the verification by setting this parameter to 'false'. The JWT tokens will be decoded and its content will be checked if a rule requires doing it, but the signatures will not be verified.

**NOTE:** Validators that do not user cryptographic systems to cypher and sign the token do ignore the 'verify' property, they do just decoding and validation.

### Azure B2C
These are the properties that define an Azure B2C validator.

##### tenant [mandatory] [string]
The tentant name of your Azure B2C service.

##### userflow [mandatory] [string]
The name of the user flow to use.

### AWS Cognito
These are the properties that define an AWS Cognito validator.

##### region [mandatory] [string]
The region where your AWS Cognito service has been deployed, something like "eu-west-1" or "us-east-2".

##### userpool [mandatory] [string]
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

##### tenant [mandatory] [string]
The name of the tentant of your Entra ID service.


### Keycloak
These are the properties that define a KeyCloak validator.

##### url [mandatory] [string]
The base URL where the KeyCloak cab be accessed. Typucally it would conntain a vlaue like "https://my.keycloak.deployment.com", but, if the KeyCloak you want to use has been deloyed the tha same Kubernetes cluster where Oberkorn runs, then you should use internal DNS name, like "http://keycloak.dev.svc.cluster.local:8080", for example, using standard DNS names inside Kubernetes (service name, namespace and the "cluster.local" TLD).

##### realm [mandatory] [string]
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
These are the properties that define a Basic Auth validator. The Basic Authentication works like classical web authentication:

  1. The user tries to access a resource, an URI.
  2. The browser doesn't send any kind of authentication info.
  3. The server answers the browser by sending a 401 and adding a 'WWW-Authenticate' header.
  4. The user enters its credentilas (user and password).
  5. The browser re-sends the original request adding an Authorization header.

This validator can work in two different modes:

  - **Static user list**. The validator uses a static list of users and passwords that are read from the definition YAML.
  - **Secret user list**. The validator uses a Kubernetes secret to read/write users database.


##### realm [mandatory] [string]
The name of the realm you want to use.

##### storeType [mandatory] [string]
Type of storage used for storing users and passwords. Two values are possible:

  - '**inline**', users are read from the validator configuration YAML.
  - '**secret**', users are read from the validator YAML if the exist, and will be stored and retrieved in a Kubernetes secret.

##### storeSecret [string]
the name of a secret where users and passwords will be stored.

##### storeKey [string]
The key inside the secrete that will hold the users and passwords.

##### users [array]
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

When the authorizator starts, it reads all users existing in the YAML file and loads them into the secret **IF THEY DO NOT ALREADY EXIST**.

### Custom
The Custom validator allow you to define your aown validation mechanism vi a JavaScript Function. This is how ti works:

  1. The end user sends a request with authorizartion  info (via Authorization header).
  2. The Custom validator extracts data from the header and send it to your function.
  3. You evaluate the request according to your own rules, and yoy decide what to do:
      - If you want to authorize the reequest your function must send back a string with access information (or whatever you want, but not null nor undefined).
      - If you want to reject the request, the *function must return undefined*.

How do you configure a validator like this?

  1. You must create an authorization function like this:
     ```javascript
        function (context) {
          if (context.token!==null && contexto.token.toLowercase()==='enjoy')
            return "Ok";
          else
            return null;
        }
     ```
  2. You must store the function in a config map, and give access to the authorizator so it can read the config map (using a kubernetes role and a kubernetes role binding). You can find an example in the samples folder (named 'sample-custom.yaml').
  3. You configure the validator adding the needed properties to use the validator.

##### configMap [mandatory] [string]
The name of the config map that holds the JavaScript function.

##### key [mandatory] [string]
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



## **ruleset**
This is where you can define what rules will drive the security of your application. As the name states, the content of this section is a rule set (an array of rules, in fact).
When a user request needs to be validated, all the rules in the ruleset are evaluated unless one of them evaulates to 'true'. In this case, no more rule evaluation is performed, and a positive response is sent back to the ingress controller. This default way of working can be modified by changing the behaviours (as explained downwards).

##### uri [mandatory] [string] 
This is a uri specification that must match user request in order to be evaluated. If the URI the user wants to access does not match this parameter, this rule will not be evaluated. URI's inside rules are relative, that is, do not include the hostname, they are referred to **just the path**.

##### uritype [mandatory] [string] 
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


##### type [mandatory] [string] 
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

##### policy [optional] [string] 
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

##### values [optional] [string] 
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

##### options [optional] [string] 
When evaluating values you may need some configuration on how to perform that evaluation. You can manage it by adding a list of options. Current supported options are:

  - lowercase
  - uppercase

When evaluating policies like 'is' or 'containsall', for example, you can force the evalutation to be performed using the right case. Please note that the options do affect the values contained in the tokens to be evaluated, **not** the values in the rule.

If you do not specify lowercase or uppercase, the comparisons will be performed using a case-sensitive comparison.


##### subset [optional] [list] 
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
