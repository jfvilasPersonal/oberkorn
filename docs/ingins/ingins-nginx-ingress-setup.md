# NGINX Ingress setup
This document will guide you thru the process of installing and configuring an NGINX Ingress controller in order to test Oberkorn.

Please, don't let the real life confuse you. There exist two similar projects around the words 'Nginx' and 'ingress controller':

  - Ingress Nginx, is somehow a fork of the Nginx project, mantained by the community.
  - NGINX Ingress, is an oficial ingress controller created by the team in charge of NGINX.

This guide is for working with the **second one**.

## Install NGINX Ingress
These are the step tou should follow to install NGINX Ingress 3.3.2.

### Step 1, the basics
The first things that must be done are:
  1. Create a namespace and a service account.
  2. Create the ingress class.
  3. Create a ConfigMap where to putoptional ingress configuration

```sh
	kubectl apply -f https://raw.githubusercontent.com/nginxinc/kubernetes-ingress/v3.3.2/deployments/common/ns-and-sa.yaml
	kubectl apply -f https://raw.githubusercontent.com/nginxinc/kubernetes-ingress/v3.3.2/deployments/common/ingress-class.yaml
	kubectl apply -f https://raw.githubusercontent.com/nginxinc/kubernetes-ingress/v3.3.2/deployments/common/nginx-config.yaml
```

### Step 2, the permissions
Next you must create a ClusterRole and a ClusterRoleBinding for the ServiceAccount you created before.

```sh
kubectl apply -f https://raw.githubusercontent.com/nginxinc/kubernetes-ingress/main/deployments/rbac/rbac.yaml
```

### Step 3, the resources
Next task is to create the CRD's for additional resources like virtual servers, policies, etc. Maybe you don't need these, it depends on the way you configure the ingress.

```sh
kubectl apply -f https://raw.githubusercontent.com/nginxinc/kubernetes-ingress/v3.3.2/deployments/common/crds/k8s.nginx.org_virtualservers.yaml
kubectl apply -f https://raw.githubusercontent.com/nginxinc/kubernetes-ingress/v3.3.2/deployments/common/crds/k8s.nginx.org_virtualserverroutes.yaml
kubectl apply -f https://raw.githubusercontent.com/nginxinc/kubernetes-ingress/v3.3.2/deployments/common/crds/k8s.nginx.org_transportservers.yaml
kubectl apply -f https://raw.githubusercontent.com/nginxinc/kubernetes-ingress/v3.3.2/deployments/common/crds/k8s.nginx.org_policies.yaml
```

### Step 4, the controller
Finally you must create the controller.

```sh
kubectl apply -f https://raw.githubusercontent.com/nginxinc/kubernetes-ingress/v3.3.2/deployments/deployment/nginx-ingress.yaml
```

### Step 5, creating ingresses
NGINX Ingress offers two ways of routing user requests to the backends:

1. Using an Ingress resources. This is the simplest case, and it is very similar to the other ingresses we have tested.
2. Using VirtualServer resoruces. This is the *enterpise* version of the ingress option. It is more complex to configure, more granular but mor powerful.

For our testing purposes we will use an **Ingress**-like resource.


## Install demo application
Our demo application is in [one of our repositories](./demo), and it is just a tuned nginx with an index.html page that we will use for testing Oberkorn using different IdM (Azure B2C, AWS Cognito and Keycloak)

Deploying our demo application is simple and straightforward, you just only need to apply a YAML to your kubernetes cluster:

```sh
kubectl apply -f https://raw.githubusercontent.com/jfvilasPersonal/obk-demo/main/ingress-jfvilas.yaml
```

Once installed you shuld be able to reach it on port 81 (the service in the YAML publishes port 80 and redirects users to por 80 served by nginx demo application)

## Configure ingress for demo application
Once the demo application and the ingress controller are installed and running, next thing to do is publish the access to the demo application via your fresh NGINX Ingress.

## Test
Demo application contains a public HTML page (index.html) and some other protected paths so you can test the application. Please refer to IdP configuration section in order to create an IdP, create a user and obtain tokens to test the application.

