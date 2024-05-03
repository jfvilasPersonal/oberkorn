# Oberkorn
Oberkorn is a fresh project for building an easy way to protect web applications deployed to **kubernetes clusters**. Initial version has been specifically designed to work with Kubernetes and Nginx Ingresss Controller, but other ingresses are also supported and we have plans to add even more.

The project is fully open sourced, you can access the repositories [here](https://github.com/jfvilasPersonal/obk-controller) and [here](https://github.com/jfvilasPersonal/obk-authorizator) (two repositories for storing code of the main two project components).

## Operation
The project is made up of 2 components:

  1. **Controller**. This is the control plane of the Oberkorn platform. The controller is in charge of receiving your commands (*kubectl* commands, for example) and create, update or remove Oberkorn authorizators that you configure for protecting your applications. It's just a typical Kubernetes controller.
  2. **Authorizator**. In order to protect one or more applications you must deploy an **Oberkorn authorizator** (kind: 'ObkAuthorizator'), which will receive authorization requests from the ingress and will respond accordingly following a set of preconfigured rules (inside a **ruleset**).

Typical use case for authorizators is implementing JWT validation, i.e., validating JWT tokens and check authorization behaviour of your applications. But you can use some other types of authentication and authorization mechanisms.

### Control plane
This is how the control plane works:

![Control Plane](/_media/architecture/controlplane.png)


The flow is as follows (once you have installed the Oberkorn Controller):
  1. You create a YAML containing the specs of an Oberkorn authorizator (See the rest of the documentation on how to build a YAML for this purpose).
  2. You apply the YAML to create the authorizator: 'kubectl apply -f your-authorizator-code.yaml'.
  3. The controller, which is listening for 'ObkAuthorizator' events receives an 'ADDED' event, so the controller creates all the resources needed to deploy an authorizator (a pod, a service, and, optionally, it configures your ingress to point its authorization needs to the newly created ObkAuthorizator).
  4. You can make changes to your auhtorizator (like changing scale process, modifying the ruleset...), so when you apply a new YAML the controller receives a 'MODIFIED' event and it performs requested changes.
  5. When you no longer need your authorizator, you can 'kubectl delete' it, so the controller will receive a 'DELETED' event and it will deprovision all previously provisioned resources (and optional configuration).

### Data plane
And this is how an Oberkorn authorizator works:

![Data Plane](/_media/architecture/dataplane.png)

The flow is as follows:
  1. A requestor tries to access a resource (typically an API call or a static web resource like 'index.html', an image, a CSS,...).
  2. When the ingress receives the request it routes the request to the Oberkorn authorizator.
  3. The authorizator examines the requested URI in order to find a **ruleset** that matches that URI.
  4. If a matching ruleset is found, the authorizator evaluates all the rules in the ruleset to discover if URI requested by user requieres any kind of authorization.
  5. If a rule matching user requested URI is found, then the rule is evaluated.
  6. If the evaluation is 'true' the access is granted. If the evaluation is false, normally, the authorizator continues searching for more rules that match the requested URI (this prcoess can be customized by using 'behaviour' configuration).
  7. If the access is granted (at least a rule evaluates to 'true'), the authorizator sends a 200 back to the ingress, then the ingress sends the request to the backend (typically a service inside the kubernetes cluster). If the response from the authorizator is 'false', a 4xx HTTP status code is sent back to the ingress, who will send back a 4xx to the requestor.

## Architecture
Oberkorn is build around two separate kubernetes components: **the controller** (in charge of the control plane) and **the authorizator** (responsible of the data plane). The architecture of the whole project is depicted below.

![Oberkorn architecture](/_media/architecture/oberkorn-architecture.png)
