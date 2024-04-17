# Roadmap

## Features
The new features added to the project will probably evolve following two different lines. We will add new features to the authorizator module in order to have a very rich and capable authorizator component. On the other side, we will add new features to the controller module in order to provide a flexible and operable controller that can ease your operation processes when managing your kubernetes clusters. 

### Authorizator features
We plan to add this features in the next months.

  - Rule processing improvements
      1. ~~Complete behaviour capabilities: 'ontrue' and 'onfalse' with actions: accept, reject, continue. Behaviour will include these default values: 'onfalse: continue' and 'ontrue: accept'~~ **DONE**
      2. Decide whether to add rule priority to rules in a ruleset (and implement it!!).
      3. Add capture groups to URI regexes and conditions on the value of the groups, so rules can contan references to those capture groups of the URIs.

  - Add more validators:
      1. Google validator.
      2. Github validator.
      3. Facebook validator.
      4. Custom validator. Ability to include external JS or TS code that can be injected in the authorizator as a plugin.
      5. External validator. Ability to invoke an external validator (via HTTP/HTTPS).

  - Token invalidation:
      1. Invalidate token according to claim values.
      2. Invalidate token for a specific user (subject).

  - JWKS keys management
      1. Refresh cached signing keys schedulable.
      2. Use different available keys when validating tokens.

  - Tracing:
      1. Monitor and trace a specific user (i.e., token).
      
  - Monitoring
      1. Performance monitoring, we need to add some additional metrics.
      2. Integrate with Grafana.

### Controller features
We plan to improve controller with:

  - Optimize authorizator modification.
  - Optimize container escalation.

## Need more?
Do you miss something? Contat us by email or just add issues to our repo.
