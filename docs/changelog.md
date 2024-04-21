# Change log
Although not too exhaustive, this page contains some detail on what we have been done on each version.

## 0.2
Added:
  - **Keycloak** validator.
  - **Basic Aauth List** validator.
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