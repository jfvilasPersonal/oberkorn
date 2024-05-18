# Roadmap

## Features
The new features added to the project will probably evolve following two different lines. We will add new features to the authorizator module in order to have a very rich and capable authorizator component. On the other side, we will add new features to the controller module in order to provide a flexible and operable controller that can ease your operation processes when managing your kubernetes clusters. 

### Authorizator features
We plan to add this features in the next months.

  - Rule processing improvements
      1. ~~Complete behaviour capabilities: 'ontrue' and 'onfalse' with actions: accept, reject, continue. Behaviour will include these default values: 'onfalse: continue' and 'ontrue: accept'~~ **DONE**
      1. ~~Add annotation to ingresses to send headers back to browsers for Basic Auth validators.~~ **DONE**
      2. Decide whether to add rule priority to rules in a ruleset (and implement it!!).
      3. Add capture groups to URI regexes and conditions on the value of the groups, so rules can contain references to those capture groups of the URIs.

  - Validators:
      1. ~~Google validator.~~ **DONE**
      2. Github validator.
      3. Facebook validator.
      4. ~~Custom validator. Ability to include external JS or TS code that can be injected in the authorizator as a plugin.~~ **DONE**
      5. External validator. Ability to invoke an external validator (via HTTP/HTTPS).
      6. ~~Basic Auth Secret. It's similar to Basic Auth List but credentials are stored in a Secret, so users can change it's passwords.~~ **DONE**
      7. ~~It is desirable to have the ability to change password in Basic Auth validator (specially when using 'secret' type). The idea comes from ancient Lotus Domino Go Webserver on IBM mainframe (https://www.ibm.com/docs/en/domino/8.5.3).~~ **DONE**
      8. Create an INLINE validator, or an 'Obklidator', that is:
         - Enable a customizable login page (a default one, and a zipped one stored in a configMap).
         - Enable endpoint and show login page.
         - Use a database (redis, nosql, sql...) to store users.
         - Create endpoints for login/logout/signup.
         - Having the ability to cerate and manage users in a validator, Oberkorn can become a simple IdM and you can protect your applications from zero to hero, just forget authetication and authorization, Oberkorn can do it all for you.

  - UX:
      1. Add a customizable login page, so applications can be deployed without worrying about authentication nor authorization, I mean, all this tasks would be performed in the Oberkorn authorizator, just by configuring rulesets.
      2. ~~Create a console website for accessing a dashboard where admins can view the status of Oberkorn.~~ **DONE**
      3. Create a fashionable demo application where Oberkorn authorizators can be defined, tested and deployed.
      4. Add 'fcgiwrap' to NGINX-based demo container so Google tokens can be moved from POST body to GET query string.
      5. Add managing capabilities to the console, I mean, to be able to modify *things* in the authorizators.
      6. ~~Add graphics/dashboards to the console.~~ **DONE**

  - Token invalidation:
      1. Invalidate token according to claim values (requires having an admin console).
      2. Invalidate token for a specific user (subject property of tokens) (requires having an admin console).

  - JWKS keys management
      1. Refresh cached signing keys in a schedulable way.
      2. Use different available keys (from JWKS config) when validating tokens.

  - Tracing & Audit:
      1. ~~Monitor and trace a specific user (i.e., token).~~ **DONE**
      
  - Monitoring
      1. Performance monitoring, we need to add some additional metrics.
      2. Integrate with Prometheus.

### Controller features
We plan to improve controller by:

  - Optimize authorizator modification (right now is just an 'apply' over existing deployed resources).
  - Optimize container escalation process.
  - Perform semantic validation on authorizator yaml's, for example, check that rule-referenced validators do exist in 'validators' section.

## Need more?
Do you miss something? Contat us by email or just add issues to our repo.
