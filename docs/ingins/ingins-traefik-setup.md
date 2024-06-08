# Traefik setup
This document will guide you thru the process of installing and configuring a Traefik ingress controller in order to test Oberkorn.

## Install Traefik
Installing traefik is real simple. The only issue you can find is to have a clear documentation for a simple kubernetes deploymment. This guide includes all the steps for deploying Traefik v2.10 for testing Oberkorn.

### Step 0, the pre-requisites
Prior to install Traefik you must have Helm installed. For installing Helm you can perform these simple steps:

```sh
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3
chmod 700 get_helm.sh
sudo ./get_helm.sh
```

Then you must add the Traefik repo to helm:
```sh
helm repo add traefik https://helm.traefik.io/traefik
```

### Step 1, the basics
Create a namespace.

```sh
kubectl create ns traefik
```

### Step 2, the controller
Installing the traefik controller is just a command away:

```sh
helm install traefik traefik/traefik --namespace traefik
```

Some installation options you may find interesting are:

```sh
--set dashboard.enabled=true \
--set rbac.enabled=true \
--set="additionalArguments={--api.dashboard=true,--log.level=INFO,--providers.kubernetesingress.ingressclass=traefik-internal,--serversTransport.insecureSkipVerify=true}" \
--version <version>
```

From top to bottom:
  - Enable the Traefik dashboard
  - Enable the use of RBAC control inside Traefik
  - Some additional options (please refer to Traefik setup docs)
  - Version

If you have enabled the dashboard, you must expose it like this:

```sh
kubectl port-forward $(kubectl get pods --selector "app.kubernetes.io/name=traefik" --output=name) 9000:9000
```

And access the dashboard on your local browser via: http://localhost:9000/dashboard

(You can also expose the dashboard via a service. Please refer to Traefik installation docs).

### Step 4, the ingress
Once everything is installed, create an Ingress for our demo purposes.


## Install demo application
Our demo application is in [one of our repositories](./demo), and it is just a tuned nginx with an index.html page that we will use for testing Oberkorn using different IdM (Azure B2C, AWS Cognito and Keycloak)

Deploying our demo application is simple and straightforward, you just only need to apply a YAML to your kubernetes cluster:

```sh
kubectl apply -f https://raw.githubusercontent.com/jfvilasPersonal/obk-demo/main/deployment-demo.yaml
```

Once installed you should be able to reach it on port 81 (the service in the YAML publishes port 80 and redirects users to por 80 served by nginx demo application)

## Configure ingress for demo application
Once the demo application and the ingress controller are installed and running, next thing to do is publish the access to the demo application via our fresh Traefik ingress.

Just apply this YAML to create an ingress that  points to our demo application:

```sh
kubectl apply -f https://raw.githubusercontent.com/jfvilasPersonal/obk-demo/main/ingress-jfvilas-traefik.yaml
```

## Test
Demo application contains a public HTML page (index.html) and some other protected paths so you can test the application. Please refer to IdP configuration section in order to create an IdP, create a user and obtain tokens to test the application.


