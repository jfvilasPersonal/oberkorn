# JWT Authorizator
JWTA is a fresh project for creating an easy way to protect web applications deployed to kubernetes clusters. Initial version has been spscifically designed to work with Kubernetes and Nginx Ingresss Controller, but we have plans to add more support regarding other ingresses integrations.

The project is fully open sourced, you can access the repositories [here](https://github.com/jfvilasPersonal/jwta-controller) and [here](https://github.com/jfvilasPersonal/jwta-authorizator) (two repositories for storing code of the two main project components).

## Operation
The project is made up of 2 components:

  1. **Controller**. This is the control plane of the project. The controller is in charge of receiving your commands and creating, updating or removing JWT Authorizators that you configure for proteciton your applications.
  2. **Authorizator**. In order to protect one or more applications you must deploy a JWT Authorizator (kind: 'JwtAuthorizator'), which will receive authorization requests from the ingress and will respond accordingly following a set of preconfigured rules (a ruleset).

### Control plane
This is how the control plane works:

![Control Plane](/_media/architecture/controlplane.png)


The flow is as follows:
  1. You create a YAML containing the specs of a JWT Authorizator. See the rest of the documentation on how to uild a YAML like this.
  2. You apply the YAML to create the authorizator: 'kubectl apply -f your-authorizator-code.yaml'.
  3. The controller, which is listening for 'JwtAuthorizator' events receives an 'ADDED' event, so the controller creates all the resources needed to deploy an authorizator (a pod, a service, and, optionally, it configures your ingress to point its authorization needs to the new JwtAuthorizator).
  4. You can make changes to your auhtorizator (like changing scale process, modifying the ruleset...), so when you apply a new YAML the controller receives a 'MODIFIED' event and it performs requested changes.
  5. When you no longer need an Authorizator, you can 'kubectl delete' it and the controller will receive a 'DLETED' event and it will deprovision all previously provisioned resources (and optional configuration).

### Data plane
And this is how a JWT Authorizator works:

![Data Plane](/_media/architecture/dataplane.png)

The flow is as follows:
  1. A user request a resource (typically an API call or a static web resource like 'index.html', an image, a CSS,...).
  2. When the ingress receives the requests it routes the request to the JWT Authorizator.
  3. The Authorizator examines the requested URI in order to find a rule that matches that URI.
  4. If a matching rule is found, then the rule is evaluated.
  5. If the evaluation is 'true' the access is granted. If the evaluation is false, normally, the authorizator continues searching for more rules that match the requested URI (this behaviour can be customized).
  6. If the access is granted (at least a rule evaluates to 'true'), the ingress sends the request to the backend (typically a service inside the kubernetes cluster). If the response form the authorizator is 'false', a 4xx HTTP status code is sent back to the customer.

## Architecture
JWTA is build around two separate resources: **the controller** (in charge of the control plane) and **the authorizator** (repsonsible of the data plane). The architecture of the whole project is depicted below.

