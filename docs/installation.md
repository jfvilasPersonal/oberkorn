# Oberkorn installation
Installing Oberkorn is very simple and straightforward.

The process includes:
  - Creating Custom Resource Definitions (CRD) on your kubernetes cluster.
  - Deploying the Oberkorn controller.
  - Optionally, configure web console access (console yas web access to the controller functions).

Once everything is ok, you can start deploying Oberkorn authorizators to protect your applications.

## Setup

#### **Step 1**. Create the CRD.
You can download and review the CRD or you can just apply it to your cluster. **Applying a CRD is not harmful**.

```sh
kubectl apply -f https://raw.githubusercontent.com/jfvilasPersonal/obk-controller/main/installation/crd.yaml
```

This will create a CRD which you will use to create your authorizators.

#### **Step 2**. Deploy the controller.
The controller is the software component in charge of managing your authorizators. It listens for kubernetes events like ADD, DELETE o MODIFY anytime you create, remove or modify one authorizator. To deploy the controller you just need to apply a YAML that contains several definitions.

```sh
kubectl apply -f https://raw.githubusercontent.com/jfvilasPersonal/obk-controller/main/installation/controller-deployment.yaml
```

The controller YAML contains several kubernetes resources:

 - A *Deployment*, which launchs a pod based on the Oberkorn controller image ('obk-controller' available on docker hub).
 - A *Service Account*. The controller needs some permissions to manage resources inside the cluster, like creating services or deployments, so we need a service account that we assign to the deployment.
 - A *Cluster Role*. The permissions needed by the controller are defined inside a cluster role named 'obk-controller-sa'. Cluster role permission is needed beacuse the controller must be able to create resources in any namespace.
 - Finally, a *Cluster Role Binding* named 'obk-controller-crb' assigns the cluster role to the created service account.

You can check whether the controller is ready by examining controller stdout (the logs of the pod) and searching for the message "Oberkorn Controller is watching events...".

#### **Step 3**. Enable the web console (**optional**).
Since version 0.3, Oberkorn includes some intersting capabilities in the controller and also in the authorizators:

  - The Oberkorn authorizators can expose an API for interactiong with them in a programatic way (enabling this API interface is optional).
  - The Oberkorn controller provides a web console (which intearacts with authorizators API), which you can use to interact with them from your favourite browser.

To enable the web console you must follow these steps:

  - Regarding the **controller**:
    - Enable the console capability.
    - Expose the web console externaly through an ingress (your favourite one).
    - Add a protection to the console (for example, require 'admin' credentials to enter the console).
  - Regarding the **authorizators**:
    - Enable the API.
    - Expose the API interface externaly through an ingress (your favourite one). This is optional, since web console access authorizators API through the cluster network. You should create ingress access if you plan to invoke authorizators API for other purposes than using the web console.
    - If you publish the APIs externally throughan ingress, you should add protection to the APIs.

To enable the the web console on a basic install you can just simply apply this YAML:

```sh
kubectl apply -f https://raw.githubusercontent.com/jfvilasPersonal/obk-controller/main/installation/controller-webconsole.yaml
```

This YAML:
  - Creates a Kubernetes service for accessing the controller console endpoint (typically exposed at http://your.dns.name:3882/obk-console).
  - Creates an ingress for accessing the web console (path '/obk-console') and its API (on the same endpoint).
  - Creates an Oberkorn Authorizator to protect the access to the console.

You have some more detaled documentation on the section [Web Console](/webconsole).

## Next step
Everyting is ready! Next you should check if there is any special configuration that you must perform depending on the ingress controller you are using. Continue to [Ingress configuration](/ingress-configuration).
