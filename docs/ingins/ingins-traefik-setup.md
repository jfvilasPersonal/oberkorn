# Traefik setup
This document will guide you thru the process of installing and configuring a Traefik ingress controller in order to test JWTA.

## Install Traefik
Installing traefik is real simple. The only issue you can find is to have a clear documentation for a simple kubernetes deploymment. This guide includes all the steps for deploying Traefik v2.10 for testing JWTA.

### Step 1. the basics
Create Namespace, ClusterRole and Service Account

```yaml

```

### Step 2, the controller
Create the controller applying this YAML:

### Step 3, the CRD's
Add the CRD's.

### Step 4, the ingress
Once everything is installed, create an Ingress for our demo purposes.


## Install demo application
Our demo application is in [one of our repositories](./demo), and it is just a tuned nginx with an index.html page that we will use for testing JWTA using different IdM (Azure B2C, AWS Cognito and Keycloak)

Deploying our demo application is simple and straightforward, you just only need to apply a YAML to your kubernetes cluster:

```yaml
kubectl apply -f enlace-al-yaml
```

Once installed you should be able to reach it on port 81 (the service in the YAML publishes port 80 and redirects users to por 80 served by nginx demo application)

## Configure ingress for demo application
Once the demo application and the ingress controller are installed and running, next thing to do is publish the access to the demo application via our fresh Traefik ingress.

## Test
Demo application contains a public HTML page (index.html) and some other protected paths so you can test the application. Please refer to IdP configuration section in order to create an IdP, create a user and obtain tokens to test the application.

