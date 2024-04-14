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
  - AWS Cognito
  - Azure AD (Entra ID)
  - KeyCloak
  - Custom

All validators include, aside from its specific configuration parameters, this 3 properties:

##### iss [optional] [string]
You can define a validator just defining its name, its tenant and its userflow (for instance). Later, when creating the rules, you can add conditions to the tokens emitted by a specific validator. Anyway, if you want to enforce a specific 'issuer' for the JWT tokens you plan to use, you can specify here a issuer value in such a way that every token to be validated against this validator must have an 'iss' value that matches the one specified here.

##### aud [optional] [string]
In a similar way, when creating rules you can add conditions to the tokens emitted by a specific validator. If you want to enforce a specific 'audience', you can specify here an audience value in such a way that every token to be validated against this validator must have an 'aud' value that matches the one specified here.

##### verify [optional] [boolean]
When a request needs to be validated, the authorizator will:
  1. Decode the token.
  2. Verify the signature of the token.

This is the default behaviour, but, when working with corporate applications that are not exposed to internet, that is, runnning in a trusty environment, the verification of the token may not be needed, so you can disable the verification by setting this parameter to 'false'. The JWT tokens will be decoded and its content will be checked if a rule requires doing it, but the signatures will not be verified.


### Azure B2C
These are the properties that define an Azure B2C validator.

##### name [mandatory] [string]
The name of the validator, that you will use later to link a specific rule to a validator. Please note that the name must be unique, but you can create different validators of the same type.

##### tenant [mandatory] [string]
The tentant name of your Azure B2C service.

##### userflow [mandatory] [string]
The name of the user flow to use.

### AWS Cognito
These are the properties that define an AWS Cognito validator.

##### name [mandatory] [string]
The name of the validator, that you will use later to link a specific rule to a validator. Please note that the name must be unique, but you can create different validators of the same type.

##### region [mandatory] [string]
The region where your AWS Cognito service has been deployed, something like "eu-west-1" or "us-east-2".

##### userpool [mandatory] [string]
The name of the userpool of your AWS Cognito service, something like "eu-west-1_up4ur74up" or " us-east-2_47h3hkgmu"


### Azure AD
These are the properties that define an Azure AD validator.

##### name [mandatory] [string]
The name of the validator, that you will use later to link a specific rule to a validator. Please note that the name must be unique, but you can create different validators of the same type.

##### tenant [mandatory] [string]
The name of the tentant of your Entra ID service.


### Keycloak
These are the properties that define a KeyCloak validator.

##### name [mandatory] [string]
The name of the validator, that you will use later to link a specific rule to a validator. Please note that the name must be unique, but you can create different validators of the same type.

##### url [mandatory] [string]
The base URL where the KeyCloak cab be accessed. Typucally it would conntain a vlaue like "https://my.keycloak.deployment.com", but, if the KeyCloak you want to use has been deloyed the tha same Kubernetes cluster where Oberkorn runs, then you should use internal DNS name, like "http://keycloak.dev.svc.cluster.local:8080", for example, using standard DNS names inside Kubernetes (service name, namespace and the "cluster.local" TLD).

##### realm [mandatory] [string]
The name of the realm you want to use.


### Examples
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
