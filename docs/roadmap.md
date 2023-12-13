# Roadmap

## Features
The new features added to the project will probably evolve following two different lines. We will add new features to the authorizator module in order to have a very rich and capable JWT Authorizator component. On the other side, we will add new features to the controller module in order to provide a flexible and operable controller that can ease your operation processes when managing your kubernetes clusters. 

### Authorizator features
We plan to add this features in the next months.

  - Rule processing improvements
      1. Add behaviour capabilities: 'ontrue' and 'onfalse' with actions: accept, reject, continue. Behaviour will include these default values: 'onfalse: continue' and 'ontrue: accept'
      2. Decide whether to add rule priority to rules in a ruleset.
      3. Add capture groups to URI regexes and conditions on the value of the groups.

  - New validators:
      1. Google validator.
      2. Github validator.
      3. Facebook validator.
      4. Custom validator.

  - Token invalidation:
      1. Invalidate token regading claim values.
      2. Invalidate token for a especifica user (subject).

  - Refresh cached signing keys schedulable.

  - Tracing:
      1. Monitor and trace a specific user (i.e., token).
      
  - Monitoring
      1. Performance monitoring, we need to add some additional metrics.
      2. Integrte with Grafana.

### Controller features
We plan to imprive controller with:

  - License validation.
  - Optimize authorizator modification.
  - Optimiza container escalation.


## Need more?
Do you miss something? Contat us by email or just add issues to our repo.
