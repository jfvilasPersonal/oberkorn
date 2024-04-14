# Keycloak setup
Oberkorn can integrate with KeyCloak in such a way that you can:

  - Protect resource access using Oberkorn by validating JWT tokens creted by a KeyCloak instance.

  As any other IdP that Oberkorn can work with, you can configure rules for just validate tokens, verify token signatures, isser, audience, etc.

# Keycloak installation
This manual is intended to help you installing KeyCloak on the same Kuebernetes where Oberkorn is installed, but you can configure Oberkorn to integrate with an external KeyCloak installation.

## Step 1. Install Keycloak
You can install a fresh version of Keycloak by just entering:

```bash
kubectl create -f https://raw.githubusercontent.com/keycloak/keycloak-quickstarts/latest/kubernetes/keycloak.yaml
```

This command will deploy KeyCloak on your cluster and create a service for accessing it on port 8080 (please pay attention to namespaces). You may need an Ingress to acccess it externaly (or just doing port-forwarding).

To create an Ingress (assuming you already have any Ingress controller installed), just create an Ingress YAML like this:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: keycloak
  namespace: dev
spec:
  rules:
    - host: localhost
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: keycloak
                port:
                  number: 8080
```

And apply it:
```bash
kubectl apply -f keycloak-ingress-v24.0.2.yaml
```

## Step 2. Create realm
Once the KeyCloak is up and running, just access the console by pointing to "http://localhost:8080" and access by entering **strong** administrator credentials:

  - USER: admin
  - PASSWORD: admin

First thing to do is to create a realm, so create realm with these parameters:

  - Name: testrealm

## Step 3. Create client application
Select the newly created realm and click on "Clients" and then on "Create Client", and create a client with these parameters:

  - Client type: OpenID Connect
  - Client name: testclient

Click on Next and enter these parameters:

  - Client Authentication: On

Click on Next and enter this parameters:

  - Valid redirect URIs: "http://localhost"

Once created, you can switch to tab "Credentials" to write down the client secret (the client id is the client name you entered before).

## Step 4. Create user
On the left menu select "Users" and create a user ("Add User" of "Create new user"):

  - Email verified: On
  - Username: testuser

Once created switch to Credentials tab and enter a password and set it to not to expire.

# Next steps
You have now a running KeyCloak with a realm (testrealm) and a user to test (testuser). You can now go to [scenarios](../scenarios.md) or [use cases](../usecases.md) to learn how to configure Oberkorn.
