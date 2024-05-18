# Change log
Although not too exhaustive, this page contains some detail on what we have been done on each version.

## 0.3
Added
  - **Basic Auth** validator with static and dynamic user list (based on Kubernetes secrets).
  - **Web console**. Right now you can see your config and performance, but no changes can be done to Oberkorn configuration via console. It's important to note that console is now aware of authorizators replicas, thta is, web console interacts with authorizators by communicating woth all the replicas available (more than one pod in the deployment). And, for instance, when you perform token invlaidation, user tracing or just gather some usage data, the controller is responsible of gathering information from all the replicas and merging it to show you precise information.
  - Authorizators now have its own **service accounts**, and the controllers give them permissions (role/rolebinsing) as needed.
  - Basic Auth of 'secret' store type now support password changes.
  - We have refactored the way validators are instantiated. Now the controller creates an instance of a validator and after that it calls the 'init' function of the newly created validator, which must return a boolean indicating if the initialization is correct (so the validator is usable).

## 0.2
Added:
  - **Keycloak** validator.
  - **Basic Auth** validator.
  - **Custom** validator.
  - **Google** validator.
  - Automatic version creation.
  - Finally, we implemented behaviours (onfalse & ontrue).

Changes:
  - No need to use config maps for configuring the authorizators (we now use annotations)

Correction:
  - Some minor optimizations on log management


## 0.1
The very first version includes:
  - the controller
  - the autorizator
  - Azure AD, Azure B2C, Cognito