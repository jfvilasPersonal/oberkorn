# Oberkorn authorizator use cases
In this section you will find different typical use cases. These are not complete scenarios, what we include here are specific solutions to typical authorization needs.

If you have an use case that is not covered here, please contact us in order to review it and help you on how to configure Oberkorn to cover your use case.

In next sections we will discuss different use cases, but showing only the ruleset needed to implement the case. 

## Basic protection: public access
This is the most simple use case: just granting acess to everyone:

```yaml
rulesets:
  - name: general
    uriPrefix: [ '' ]
    rules:
      - uri: "/"
        uritype: "prefix"
```
In this case every request will be granted.

## Basic protection: ensure token is present
On the other side, you may want to protect everything you publish, so you want to ensure every user that accesses you applications must present a valid token.

```yaml
rulesets:
  - name: general
    uriPrefix: [ '' ]
    rules:
      - uri: "/"
        uritype: "prefix"
        type: valid
        onfalse: reject
```
In this case, if a valid token is not presented by the user (via 'Authorization' HTTP header) the request is rejected and no further rule processing is done. If the user presents a valid token the request is granted (although not added to the rule, there is a defualt value for positive evaluation: 'ontrue: accept').

## Basic protection: token present and other rules
If you want to protect everything you publish but at the same time you need a mechanism to fine tune the accesses to your applications, you should ensure a valid token is present, but after that continue evaluating the rest of the ruleset.

```yaml
rulesets:
  - name: general
    uriPrefix: [ '' ]
    rules:
      - uri: "/"
        uritype: "prefix"
        type: valid
        onfalse: reject
        ontrue: continue
        .
        .
        .
```
In this case, if a valid token is not presented by the user (via 'Authorization' HTTP header) the request is rejected. But, if a valid token is present, the rule engine will continue processing the rest of the rules in the ruleset.

## API protection
If you want to protect access to an API resource the most simple rule is this one: for a user to access and enpoint that starts with "/api/" the user must present a valid JWT token. The Oberkorn authorizator expects an 'Authorization' header to be present in the request containing a valid token.

```yaml
rulesets:
  - name: general
    uriPrefix: [ '' ]
    rules:
      - uri: "/api/"
        uritype: prefix
        type: valid
```

Let's do it a bit more complex. We want now to protect access to two different API paths. A user with a valid token can access several API's, but if the user wants to access '/api/admin' or '/api/config' then the token must include a specific claim named 'PROFILE' and the claim must have the value 'ADMIN'. This would be the ruleset for this situation:

```yaml
rulesets:
  - name: general
    uriPrefix: [ '' ]
    rules:
      - uri: "/api/"
        uritype: prefix
        type: valid
        ontrue: continue
        onfalse: reject
      - uri: "^\/api\/admin\/|^\/api\/config\/"
        uritype: regex
        type: claim
        name: PROFILE
        policy: is
        values:
          - ADMIN
```
- If the first rule is matched (a request starting with '/api/' is received) and the evaluation of the rule is true (to have a 'valid' token) then the process continues to next rule in the ruleset, as the 'ontrue' behaviour is set to 'continue'.
- If the first rule is matched (a request starting with '/api/' is received) and the evaluation of the rule is false (the request does not include a 'valid' token) then the process ends and a negative response is sent back to the ingress controller.
- In the case of a positive response obtained from the first rule, the second rule must be evaluated (according to the behaviour set on the 'ontrue' parameter). If the API invocation path starts with '/api/admin/' or '/api/config/' then the rule must be evaluated. The second rule type is 'claim', and the policy is 'is', so, for the token to be valid, it must contain a claim named 'PROFILE' whose value must be 'ADMIN'.

## Different types of users in one application
This is a very typical use when exposing API's. Context: we have a an aplication (no mattter the front is SPA, mobile, desktop or whatever), and the front application consumes backend API exposed as REST API. There are three kind of endpoints:

  - /api/data, all users with a valid token can access this endpoint-
  - /api/operator, operator users (and admin users) can access this endpoint.
  - /api/admin, only admin users can access this endpoint.

To implement this scenario we have two options (at least):
  1. We create a USER_PROFILE claim containing the specific profile of the user (in fact, the type of user). For example: USER_PROFILE="OPERATOR" or USER_PROFILE="ADMIN"
  2. We create a USER_ROLES claim containing a list of roles. For example, USER_ROLES="OPERATOR", or USER_ROLES="ADMIN" or even USER_ROLES="OPERATOR ADMIN"


**Solution for use case 1**

We just need 3 rules: one for general users (they must present a valid token), another one for operators (the must have a profile of OPERATOR), and a third one for administrators (with value ADMIN). This could be the answer:

```yaml
rulesets:
  - name: general
    uriPrefix: [ '' ]
    rules:
      - uri: "/api/data/"
        uritype: prefix
        type: valid
      - uri: "/api/operator/"
        uritype: prefix
        type: claim
        name: USER_PROFILE
        policy: is
        values:
          - OPERATOR
      - uri: "/api/admin/"
        uritype: prefix
        type: claim
        name: USER_PROFILE
        policy: is
        values:
          - ADMIN
```

In this case, as you guess, an administrator cannot access the opertors endpoint, so it would be better to write a ruleset like this other:

```yaml
rulesets:
  - name: general
    uriPrefix: [ '' ]
    rules:
      - uri: "/api/data/"
        uritype: prefix
        type: valid
      - uri: "/api/operator/"
        uritype: prefix
        type: claim
        name: USER_PROFILE
        policy: is
        values:
          - OPERATOR
          - ADMIN
      - uri: "/api/admin/"
        uritype: prefix
        type: claim
        name: USER_PROFILE
        policy: is
        values:
          - ADMIN
```
That is, assigning two possible values to the claim in the case of '/api/operator'.

**Solution for use case 2**

In case 2 the idea is to assign roles to users, so one possible solution would be:

```yaml
rulesets:
  - name: general
    uriPrefix: [ '' ]
    rules:
      - uri: "/api/data/"
        uritype: prefix
        type: valid
      - uri: "/api/operator/"
        uritype: prefix
        type: claim
        name: USER_ROLES
        policy: containsany
        values:
          - OPERATOR
          - ADMIN
      - uri: "/api/admin/"
        uritype: prefix
        type: claim
        name: USER_ROLES
        policy: is
        values:
          - ADMIN
```

The second rule will evaluate to true if the claim USER_ROLES contains the text "OPERATOR" or the text "ADMIN". The third rule will evaluate to true if and only if the claim USER_ROLES value is ADMIN (you could alos use here 'conatinsall' or 'containsany' policies, there will be no difference since your are searching for just one concrete value).

Please note the difference between 'is' with 2 values (first use case) and 'containsany' with the same two values. In the first case ('is') the rule will be evaluated to true if the claim value is exactly ADMIN or is exactly OPERATOR. In the second case ('containsany') the rule will evaluate to true if the claim value (a string in fact) contains ADMIN or contains OPERATOR. In this second case, a value of 'ADMINISTRATOR' will evaluate to true, and a value of 'USER ADMIN OPERATOR' will also evaluate to true.

## Multiple applications on the same base endpoint
Another typical situation you can find is a portal to access different applications served from the same web site (this is a typical 90-like web architecture).

Imagine you have an intranet application, a corporate application, where your users, belonging to differente departments access differente sub-applications. You can use roles or permissions, like previous example, or you can just use audiences. 'audience' is a claim present in all JWT tokens that is used to say who is the audience ***(bunch of users) that can access a specific resource***.

The audience claim is added to tokens when they are created, and has a name of 'aud' and a string value.

So there are two possible solutions to this need:
  1. Create specific validators pre-configured for each audience and attach them to the applications.
  2. Validate the 'aud' claim on each application.

**Solution for use case 1**

We need to add to the solution the rulesets and the validators. It should be something like this:

```yaml
validators:
  - azure-ad:
      name: validator-one
      tenant: corporate
      aud: b8e73549-51bf-4bc2-ade0-6e4983bcd26d
  - azure-ad:
      name: validator-two
      tenant: corporate
      aud: cde21249-287f-befd-2318-9833bc3cd365
rulesets:
  - name: general
    uriPrefix: [ '' ]
    rules:
      - uri: "/app-one"
        uritype: prefix
        type: valid
        validators:
          - validator-one
      - uri: "/app-two"
        uritype: prefix
        type: valid
        validators:
          - validator-two
```

Very easy, yes, and it is very reconfigurable. Please read this other use case: application 'app-two' can be used by users of audience 'one' and audience 'two'. In this case, just write the second rule like this:
```yaml
rulesets:
  - name: general
    uriPrefix: [ '' ]
    rules:
      - uri: "/app-two"
        uritype: prefix
        type: valid
        validators:
          - validator-one
          - validator-two
```

**Solution for use case 2**

It's pretty like other examples we've shown before: just checking a claim value.

```yaml
rulesets:
  - name: general
    uriPrefix: [ '' ]
    rules:
      - uri: "/app-one"
        uritype: prefix
        type: claim
        name: aud
        policy: is
        values: ['b8e73549-51bf-4bc2-ade0-6e4983bcd26d']
      - uri: "/app-two"
        uritype: prefix
        type: claim
        name: aud
        policy: is
        values: ['cde21249-287f-befd-2318-9833bc3cd365']
```
