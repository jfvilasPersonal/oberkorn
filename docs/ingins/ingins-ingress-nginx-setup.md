# Ingress Nginx setup
This document will guide you thru the process of installing and configuring an Ingress Nginx controller in order to test Oberkorn.

Please, don't let the real life confusing you. There exist two similar projects around the words 'Nginx' and 'ingress controller':

  - Ingress Nginx, is somehow a fork of the Nginx project mantained by the community.
  - NGINX Ingress, is an oficial ingress controller created by the team in charge of NGINX.

This guide is for working with the **first one**.

## Install Ingress Nginx
Let's create a lot of resources:

```yaml
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.9.4/deploy/static/provider/cloud/deploy.yaml
```

We install version 1.9.4 from source repositories.

## Install demo application
Our demo application is in [one of our repositories](https://github.com/jfvilasPersonal/jwta-demo), and it is just a tuned nginx with an index.html page that we will use for testing Oberkorn using different IdM (Azure B2C, AWS Cognito and Keycloak)

Deploying our demo application is simple, you just only need to apply a YAML to your kubernetes cluster:

```yaml
kubectl apply -f https://raw.githubusercontent.com/jfvilasPersonal/jwta-demo/main/demo-nginx.yaml
```

This creates a deployment and a service for accessing the application inside the pod. Once installed you should be able to reach it on port 81 (the service in the YAML publishes port 80 and redirects users to port 80 served by nginx demo application)

## Configure ingress for demo application
Once the demo application and the ingress controller are installed and running, next thing to do is publish the access to the demo application via our fresh Ingress Nginx.

Just apply this YAML to create an ingress that  points to our demo application:

```yaml
kubectl apply -f https://raw.githubusercontent.com/jfvilasPersonal/jwta-controller/main/test/ingress-jfvilas.yaml
```

## Test
Demo application contains a public HTML page (index.html) and some other protected paths so you can test the application. Please refer to IdP configuration section in order to create an IdP, create a user and obtain tokens to test the application.

